# The output contract

Every finding carries seven fields. Prose around them is fine; the fields are not optional. A finding missing its Evidence or its Resolver is an opinion.

## Contents

- The seven fields
- Verdict vocabulary
- Interpretation vocabulary
- Confidence
- Routing targets
- Escalation
- A worked finding
- Anti-examples

## The seven fields

**Subject.** The vessel, entity, cluster, or ownership edge at issue, by identifier. Entity ids and cluster ids change every synthesis run, so an id is cited with the run it belongs to, or alongside something stable: a normalised name, an IMO company number, a source row reference.

**Evidence.** Which observations, from which sources, captured when. The tool call that produced them, with its arguments. `captureDate` is the register's own publication date and is what you cite; `ingestedAt` is when the corpus read the file and appears nowhere in a finding.

**Verdict.** One value from the list below.

**Interpretation.** One value from the list below. The verdict says what the record shows; the interpretation says why, and the cheapest explanation that fits is the right one until it is ruled out.

**Confidence.** High, medium, or low, with the reason for anything below high. A band, never a decimal.

**Route.** Which review queue, and which decision is being requested of a human. You name the queue; you do not write to it.

**Resolver.** The single piece of evidence that would settle it. A registry extract, a filing, a second source, a mapping check. If nothing would settle it, say that instead - it is a stronger finding than a vague one.

## Verdict vocabulary

- `MISSING_LINK` - an ownership or control link the evidence supports is absent from the network.
- `UNSUPPORTED_LINK` - a link is asserted on evidence that does not support it.
- `OVER_MERGED` - two or more distinct parties fused into one entity or cluster.
- `UNDER_MERGED` - one party split across several entities or clusters.
- `CONTRADICTORY_SOURCES` - two authoritative sources disagree, and the record cannot resolve it.
- `CHAIN_INCOMPLETE` - the chain terminates before reaching a natural or legal person.
- `STRUCTURAL_OPACITY` - the gap is the structure working as designed, not a data defect.
- `CONSISTENT` - checked and sound. Report these. An audit that only emits defects cannot be calibrated.

## Interpretation vocabulary

- `UNMAPPED_SOURCE_FIELD` - the source carries the data; the extraction mapping never read that column.
- `NORMALISATION_FAILURE` - `pipeline.norm_entity_name` or a helper produced a form that did not match where it should have, or matched where it should not.
- `ROLE_MISMATCH` - owner, operator, and master conflated, or a role-blind mechanism applied to a role-keyed record.
- `TRANSLITERATION_VARIANT` - systematic romanisation variance across registries.
- `GENERIC_NAME_COLLISION` - a name common enough that similarity carries no evidential weight.
- `SCOPE_HEURISTIC_ARTEFACT` - a cap, a country or flag scope, or a fanout guard produced the boundary. The record is behaving as specified.
- `DELIBERATE_STRUCTURE` - a shell layer, nominee, or jurisdictional hop that exists to break the link.
- `LEGITIMATE_EXCEPTION` - the ordinary structure of the industry. Single-ship companies, offshore incorporation, flag differing from owner nationality.

`SCOPE_HEURISTIC_ARTEFACT` and `LEGITIMATE_EXCEPTION` are the two that get skipped under pressure, and they are the two that most often apply. Rule them in or out before reaching for `DELIBERATE_STRUCTURE`.

## Confidence

`high` - the evidence is corroborated across sources or rests on a definitive identifier, and the cheaper interpretations have been checked and excluded.

`medium` - the evidence supports the verdict but one alternative interpretation remains open. Name it.

`low` - single source, single observation, or an interpretation not yet tested. Name what is untested.

An entity carrying `entity_confidence = 'low'` is a single-source record with no IMO company number. A finding resting on one cannot be `high`, whatever else is true.

## Routing targets

Verify these against the live database before routing on them. Read the `CHECK` constraints on the tables named, and call `corpus_schema` for the controlled vocabularies. Codes and queues move; this list was read at repo version `0.411.0` on 2026-08-03. If a name here does not exist any more, say so in the finding and ask Guille once, then use what he gives you for the rest of the session.

You never write to any of these. Routing means naming the queue and the decision being requested, so a human can execute it and the decision is logged.

**`pipeline.entity_name_pair_reviews`** - admin UI at `/synthesis/entity-name-pairs`, facade `EntityNamePairReviewFacade`. For `UNDER_MERGED` where the two records are near-name firm pairs sharing a vessel. Decision requested: `accepted` or `rejected` on the pair, keyed `(norm_name_lo, norm_name_hi)`. An accepted pair becomes a role-aware forced union on the next run. Cite the tier - `edit_distance`, `one_token`, or `jaccard` - because it says what the machine already knows about the pair.

**`pipeline.cluster_split_overrides`** - must never cluster. Facade `ClusterReviewFacade`, admin UI at `/synthesis/clusters`. For `OVER_MERGED`. State the roles: an override written role-blind (`a_role` and `b_role` empty) also forbids the operator pair on the same source row and will split an unrelated operator cluster.

**`pipeline.cluster_merge_overrides`** - must always cluster. Same facade and UI. For `UNDER_MERGED` where the pair is not near-name and so never reaches the pair queue. Same role warning.

**Entity triage workbench** - `/synthesis/entities`, `EntityWorkbenchFacade` for the projection, the `DecideTriage` BTX for the decision, `SynthesisFacade` as sole writer of `triage_status` and `triage_decisions`. For entity-level accept or reject on a candidate record. Filterable on `entity_review_reason` and `triage_status`, which makes the `imo_company_conflict` cohort directly addressable.

**`pipeline.entity_relationships.triage_status`** - `candidate`, `accepted`, `rejected`, `pending`. For `UNSUPPORTED_LINK` on a machine-derived edge. Remember what `pending` already means: withheld from consumption by design, not a weaker candidate.

**Code, as a kanban card** - for anything that is a defect in the logic rather than in a record: an unmapped source field, a reconcile rule producing an incoherent cluster, a queue that does not exist for a class of finding you keep producing. You produce the evidence and the diagnosis; the change is his.

**Guille, directly** - the escalations below.

## Escalation

Before the finding goes anywhere else:

- It would move a headline figure used outside the team: entity count, attribution coverage, dark fleet share.
- A whole source's ownership fields look extracted against the wrong mapping.
- It names a natural person, or would place one in a network with a sanctioned party. `MASTER_OF` edges are person edges by construction.
- Two authoritative sources disagree on ownership in a way the record cannot resolve and that matters to a cluster.
- The question asked is legal, enforcement, sanctions, or commercial rather than evidentiary.

## A worked finding

This illustrates the shape and the discipline. Consider what fits the finding in front of you; the fields are fixed, the prose around them is not.

> **Subject** - entity cluster holding 14 owner observations under three renderings of one Dalian firm, synthesis run `4c4fb3f7`, normalised name `DALIANOCEANFISHING`. Cited by normalised name because the cluster id does not survive the next rebuild.
>
> **Evidence** - `entity_resolve` on the name fragment returns 16 distinct entity records holding between 1 and 41 vessel links. `entity_dossier` on the largest returns 7 name renderings and 21 address renderings. `provenance_attribute` on `country` returns one value, `CHN`, from 9 organisations, captured 2014-05-31 to 2026-05-12. Two of the 16 records carry no country attribute at all and appear only in source 1424, captured 2019-11-03.
>
> **Verdict** - `UNDER_MERGED`. Two records that the evidence places with the main cluster sit outside it.
>
> **Interpretation** - `SCOPE_HEURISTIC_ARTEFACT`. Both outliers lack the country attribute the `name_country` rule needs, and their name rendering is not near enough for the pair queue's tiers. This is the rule declining to merge on absent evidence, not a normalisation failure: `norm_entity_name` produces the same form for all three renderings, which I checked directly.
>
> **Confidence** - medium. The normalisation is confirmed and the rule behaviour explains the split, but the two outliers rest on a single source and a single capture, so their identity with the main cluster is corroborated by name alone.
>
> **Route** - `pipeline.cluster_merge_overrides`, role-exact on `owner` for both sides, decision requested: force union with the main cluster. Not the pair queue: the renderings are not near-name and would never be generated as a pair.
>
> **Resolver** - the Chinese corporate registry extract for the firm, or any second source carrying those two records with a country attribute. Either would move this to high, or kill it.

## Anti-examples

Do not write these.

"This company is linked to 130 others." The edge count is co-occurrence on hulls. The sentence asserts a relationship the data does not carry.

"The vessel has no owner." It has no owner observation in the sources held, as at their capture dates. Those are different claims and only the second is defensible.

"Twenty-one addresses suggest a dispersed corporate footprint." Twenty-one renderings of a handful of Dalian addresses suggest twenty-one registers spelling them differently.

"The chain terminates in the British Virgin Islands, indicating deliberate concealment." The chain terminates in the BVI. What that indicates is not established by the chain terminating there, and single-jurisdiction termination is the ordinary case.

"Confidence: 0.72." A band, not a decimal. The decimal implies a calibration that does not exist.

"Merged - both records are clearly the same firm." Nothing in this skill's remit merges anything. The verb is route, and the sentence has to name the queue and the decision.
