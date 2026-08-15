# How the entity layer is actually built

Read this before diagnosing any merge, split, or missing edge. Most apparent errors are a documented rule behaving exactly as specified, and a finding that does not know which rule produced the record is not a finding.

Everything below was read from `main` at version `0.411.0` on 2026-08-03, at `/Users/guillermo/src/kanikosen/kanikosen-pipeline` (use absolute paths; shell cwd drifts between calls). Thresholds and vocabularies move. Verify anything you are about to route on:

- Controlled vocabularies live: `corpus_schema` with no arguments returns `entity_relationship_types`, the link roles, and `attribute_names.<section>` for `owner`, `operator`, and `master`. That is the authority; this file is a convenience copy that can drift.
- Constraint domains live: read the `CHECK` constraints on `entity_name_pair_reviews`, `entity_relationships`, and the override tables rather than trusting the lists here.
- Rules and thresholds: `src/main/java/com/kanikosen/pipeline/synthesis/engine/EntityReconciler.java` and `EntitySynthesiser.java`.

## Contents

- The rebuild invariant
- The cluster key, and why role is part of it
- The reconcile rules and their confidences
- The key-group cap, and the one asymmetry that matters
- Name normalisation
- Confidence and review-reason ladders
- Triage status
- The near-name pair queue
- The override tables
- Relationship edges and what each one means
- Where to read further

## The rebuild invariant

Entity reconcile is TRUNCATE and rebuild on every synthesis run. Entity ids change every run. Cluster ids change every run.

Human decisions are persisted outside that rebuild, keyed on evidence that survives it: normalised name pairs, and source row references. That is the design constraint behind every review mechanism below, and it is why nothing durable is ever keyed on an entity id.

Consequence for your work: an entity id you cite in a finding is valid for the synthesis run it came from and no longer. Cite the run, or cite something stable - a name, an IMO company number, a source row reference. `corpus_stats` returns the synthesis run id the current figures belong to.

## The cluster key, and why role is part of it

`pipeline.entity_clusters` is keyed `(source_id, sheet_no, source_row_no, role)` with role in `owner`, `operator`, `master`.

One physical source row contributes several parties. The owner column and the operator column on the same row are usually two different companies. A mechanism that keys on the row triple alone cannot say which party it means, and applying it role-blind pairs a row's owner observation with its own operator observation - two unrelated firms fused.

This is the origin of a whole class of real defects, and it is why "check the role before concluding the data is wrong" is a standing instruction rather than a nicety. The corpus carries 42.6M+ owner and operator observations across those three roles.

## The reconcile rules and their confidences

`EntityReconciler` clusters rows sharing entity-identity evidence. Union-find runs over DISTINCT identity signatures, not per row: the raw corpus multiplies each entity's identity across millions of rows and a per-row graph exhausts the run heap. Signatures carry a `row_count` weight; `role` and the row triple are member-key metadata fanned out afterwards, not part of the signature.

Ten rules, with their declared confidences:

- `imo_company_no` - 1.00
- `lei_code` - 1.00
- `duns_code` - 1.00
- `trusted_source_record` - 1.00 (source tier from `sources.registry_type`)
- `registration_number_country` - 1.00
- `tax_number_country` - 1.00
- `ogrn_country` - 1.00
- `name_address` - 0.85
- `name_vessel` - 0.75
- `name_country` - 0.50

The seven at 1.00 are globally unique legal identifiers. The three below it contain a normalised name.

`pipeline.vessel_clusters` and its entity sibling carry a `rule_claims` JSONB explaining why a row clustered where it did. That column is where a suspicious merge is diagnosed, and it is the first thing to read before asserting one.

## The key-group cap, and the one asymmetry that matters

`good_keys` enforces a lower bound of 2 raw rows for every rule: a key seen once cannot merge anything. The upper bound is derived from rule confidence alone.

Definitive rules (confidence >= 1.00) have no upper bound. Their key cannot collide across firms by construction, so a large group is evidence of a large fleet, not of a homonym.

Corroboration rules (`name_address`, `name_vessel`, `name_country`) keep a 2 to 50 raw-row cap. Their key contains a normalised name, which collides across unrelated firms, and group size is the only cheap signal that a name is generic. A normalised name is never uncapped at any group size.

Two readings follow directly, and both are findings you will produce repeatedly:

A firm whose entities failed to cluster may have hit the cap rather than failed a match. Check the group size before calling it a normalisation failure - `SCOPE_HEURISTIC_ARTEFACT`, not `NORMALISATION_FAILURE`.

A definitive-rule over-merge is the expensive failure. Uncapping turns a sentinel value in an identifier column into a fleet-wide fusion, which is why the rules carry explicit guards against reserved and placeholder values. A definitive rule producing an implausibly large or jurisdictionally incoherent cluster is a code finding, routed to Code, not a data finding.

## Name normalisation

`pipeline.norm_entity_name` is a SQL function, revised repeatedly (migrations V128 through V130, V192, V196). It carries guards for numeric IMO values, enumerator and acronym forms, and country-token stripping against generic names.

Three helper functions sit alongside it and are worth calling directly when triaging a pair:

- `pipeline.entity_name_is_firm(name)` - true when the name carries a legal-form or industry token. Whole-token matched, so `SA` and `CO` fire as standalone tokens and not inside `SANTOS` or `COASTAL`.
- `pipeline.entity_name_is_placeholder(name)` - the named-junk gate: `NO INFO`, `NO TIME`, `UNKNOWN`, `NA`, `SAME AS ABOVE`, `VARIOUS`, `OWNER`, `VESSEL OWNER`, and their relatives.
- `pipeline.entity_name_content_tokens(name)`, `entity_name_token_jaccard(a, b)`, `entity_name_token_onediff(a, b)` - the discriminating-token view, with the firm vocabulary dropped, which is what the near-name tiers are computed over.

A placeholder entity is not deleted. It legitimately anchors authorisations. Only its derived relationship edges are suppressed. An "owner" called `UNKNOWN` is a real record of a source declining to name one, and reporting it as a missing owner without saying that is a `MISSING_LINK` verdict resting on the wrong interpretation.

## Confidence and review-reason ladders

`entities.entity_confidence`, assigned per cluster:

- `definitive` - the entity has an `imo_company_no`.
- `high` - no IMO company number, but two or more distinct sources corroborate.
- `low` - single-source entity with no IMO company number.

`entities.entity_review_reason` currently has exactly one non-null value: `imo_company_conflict`, set when a cluster's member observations carry more than one distinct IMO company number. That is the machine's own over-merge alarm and it is the cheapest cohort to audit first.

A `low`-confidence entity is a single-source record. It is not wrong; it is uncorroborated, and a finding resting on one needs that stated.

## Triage status

`triage_status` on every candidate table: `candidate`, `accepted`, `rejected`. `entity_relationships` alone adds `pending`.

`pending` is not a weaker `candidate`. It marks the low-confidence relationship tier deliberately withheld from consumption. An edge sitting in `pending` was never shipped as a candidate, and quoting it as an established relationship misreads the tiering.

Triage writes go through `SynthesisFacade`, which is the sole writer for `triage_decisions` and `triage_status`. You do not write either.

## The near-name pair queue

`pipeline.entity_name_pair_reviews` is the human triage queue for same-vessel near-name pairs that survived the exact-after-normalisation auto-merge.

Keyed on `(norm_name_lo, norm_name_hi)` - the two normalised names, lexically ordered. That key is stable evidence that survives the rebuild, so an accept or reject is never re-offered even though entity ids change every run.

Tiers, in the `tier` column: `edit_distance` (Levenshtein <= 2), `one_token` (content-token sets differ by exactly one, with at least one shared token), `jaccard` (token Jaccard >= 0.8).

Status: `pending`, `accepted`, `rejected`. Accepted pairs feed `EntityReconciler` as role-aware forced unions in a final pass.

Both sides carry a representative observation with its own role (`a_role`, `b_role`, defaulting to `owner`) precisely because the role-blind override mechanism would pair a row's owner with the same row's operator. Telemetry on the row: `shared_vessel_count`, `sample_vessel_id`, `edit_distance`, `jaccard`, `classifier`.

The gate that generates these pairs requires both sides to be firm-shaped, which excludes person names and named placeholders in one test. A genuinely distinct pair of firms that both look firm-shaped is exactly what a human is being asked to catch, so a wrong `accepted` decision here produces an over-merge that no later rule will undo.

## The override tables

`pipeline.cluster_split_overrides` - must never cluster. `pipeline.cluster_merge_overrides` - must always cluster. Both carry a `domain` column and serve the vessel and entity paths.

Both gained `a_role` and `b_role` (migration V191). The default is the empty string and it is load-bearing: empty means role-blind, which preserves the vessel path and the pre-existing rows bit for bit. The entity reconciler keeps a role fan-out for a blank pair and becomes role-exact only when the column is set.

So a split override written without roles over-applies. Removing an owner observation from a cluster also forbids the operator pair on the same row, splitting an unrelated operator cluster. If you are auditing a split that looks wrong, check whether a role-blind override caused it before concluding the reconciler misbehaved.

Known and recorded: `cluster_merge_overrides` and `entity_name_pair_reviews` are reached through raw SQL rather than generated jOOQ, so they do not benefit from compile-time schema checks.

## Relationship edges and what each one means

`pipeline.entity_relationships` is TRUNCATEd and repopulated every run. Its columns are provenance for the current run's edges, not durable history. Four rules produce it, all machine-derived from vessel co-occurrence:

**`OPERATOR_FOR`**, confidence 0.80, rule `operator_owner_shared_vessel`, basis `shared_vessel`. Directional: operator to owner on the same hull. `evidence_count` is the number of distinct shared hulls.

**`MASTER_OF`**, confidence 0.80, rule `master_company_shared_vessel`, basis `shared_vessel`. Directional: master person to the owner or operator company sharing the hull. Any finding built on this names a natural person and is a hard escalation.

**`SAME_GROUP`**, confidence 0.50, rule `shared_address_shared_vessel`, basis `shared_address+shared_vessel`. Requires a shared address AND a shared vessel AND differing normalised names. The address side is fanout-guarded at 25 distinct entities: an address held by more than 25 entities emits no edge at all. That guard is why a formation-agent address produces silence here rather than a thousand spurious group edges, and it means absence of a `SAME_GROUP` edge at a high-fanout address is the expected behaviour, never a missing link.

**`CO_OWNER`**, confidence 0.30, rule `co_owner_shared_vessel`, basis `shared_vessel`, landing in `triage_status = 'pending'`. Two distinct-named owners on a hull carrying 2 to 25 distinct owner entities. Norm-equal pairs are excluded, because those are merge candidates rather than edges.

Edges anchored to a named-placeholder entity are generated and then deleted, per type, so the run telemetry records what was suppressed. `OPERATOR_FOR` carries essentially all of that false-positive volume, because it matches on vessel co-occurrence alone while the two name-distinct rules already exclude placeholders.

**The reading that governs all four.** `evidence_count` counts co-anchoring hulls or addresses. Not sources, not documents, not filings. An edge establishes that two names were recorded against the same hull. It establishes nothing about shareholding, control, or group membership. "Linked to 130 companies" is a claim the data does not support; "recorded against the same hulls as 130 other parties, machine-derived" is what you have.

The corollary for the audit: this table is not a beneficial ownership graph. It is co-occurrence with a type label attached. A genuine control chain needs registry evidence, and the absence of that evidence is the finding rather than a reason to promote an edge.

## Where to read further

- `specs/components-map.md` - what owns what. `synthesis.engine` (`SynthesisFacade`), `synthesis.review` (`ClusterReviewFacade`, `EntityNamePairReviewFacade`), `synthesis.triage` (six workbenches, all read-only projections; every triage write goes through `SynthesisFacade`).
- `src/main/java/com/kanikosen/pipeline/synthesis/engine/EntityReconciler.java` - clustering. The class javadoc carries the signature-dedup and cap reasoning in full.
- `src/main/java/com/kanikosen/pipeline/synthesis/engine/EntitySynthesiser.java` - attributes, confidence ladder, entity type classification, relationship edge construction.
- `src/main/java/com/kanikosen/pipeline/synthesis/review/` - `ClusterReviewFacade`, `EntityNamePairReviewFacade`, and the package README.
- `src/main/java/com/kanikosen/pipeline/synthesis/triage/entity/EntityWorkbenchFacade.java` - the entity workbench projection: filters on `entity_review_reason` and `triage_status`, and the detail view that assembles attributes, timelines, `rule_claims`, vessel links, relationships, source refs, and provenance for one entity. The closest thing to a ready-made audit view.
- `docs/mcp-server.md` - the tool catalogue.
- `src/main/resources/db/migration/` - the migrations named in this file: V074, V122, V128 to V132, V191, V192, V195, V196.
