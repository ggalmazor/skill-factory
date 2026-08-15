# Tradecraft

Three parts: the structures that break an ownership chain on purpose, the resolution hazards that break one by accident, and the sweeps that find both. The queries here illustrate the shape of an approach. They are not a checklist to run in order; the question comes first, and the query follows it.

## Contents

- Structures built to break the chain
- Entity resolution hazards
- Completeness checks worth running
- Query shapes

## Structures built to break the chain

Each of these is a starting point. None is a conclusion, and several are the ordinary structure of the shipping industry rather than a signal of anything.

**Shell layers.** A registered owner that is a company with no employees, no other assets, and one vessel. Single-ship companies are the normal structure in shipping for liability reasons. The finding is never the shell; it is where the chain stops above it.

**Formation agent addresses.** An address hosting hundreds of registered entities is a company formation service, not a fleet headquarters. That is a finding for the attribution layer, not a defect in the record. `entity_shared_address` returns the count per address, and calling it before treating a shared address as a corporate link is not optional: a formation-agent address carries hundreds of unrelated companies and a concealment address carries three, and they are indistinguishable without the count.

Matching is on the exact stored string, so a differently spelled rendering forms its own cohort. The count is a floor, never an overstatement. Twenty-one address renderings of one company's premises are twenty-one spellings, not twenty-one locations, and the neighbour count seen three times through three spellings is one neighbour.

**Nominee directors and shareholders.** Repeated across unrelated companies, often resident in the jurisdiction of incorporation. Their presence tells you the real owner is deliberately one layer further back. It tells you nothing about who that owner is, and any finding naming one is a hard escalation.

**Layered holdings across jurisdictions.** Each hop crossing into a jurisdiction with weaker disclosure is a design choice. Record the hops you can evidence and name the one where the trail goes cold. "The chain stops here and I cannot see past it" is the complete finding; what sits behind the wall is not yours to guess.

**Flag of convenience beneficial owners.** The registered owner's nationality frequently differs from the flag. In this fleet that is the normal case, not an anomaly. It is also exactly the signal the attribution layer exists to capture, so it is never normalised away either.

**Joint ventures versus parent and subsidiary.** Structurally different relationships that sources describe with the same vague language. Where the evidence cannot distinguish them, say so and leave the generic relationship rather than inventing a hierarchy. The corpus has no relationship type that means "parent", and asserting one from a `SAME_GROUP` edge invents the finding.

## Entity resolution hazards

**Generic names.** "Ocean Fishing Company Limited" and its thousand cousins. Name similarity alone is never sufficient evidence for a merge across jurisdictions. The reconciler already knows this, which is why every name-bearing rule carries a group-size cap and no name-bearing rule is ever uncapped.

**Transliteration.** Chinese, Korean, Japanese, Russian, and Spanish source names vary systematically across registries. Variation is expected and is not evidence of distinct firms. The inverse error is equally live: two genuinely different firms whose romanisations converge.

**Name recycling and sequential naming.** Numbered corporate series from one operator are real. So is the reuse of a dissolved company's name by an unrelated party. Neither is distinguishable from name alone, and the resolver for both is a registry extract with incorporation dates.

**Role confusion.** Owner, operator, and master are different roles and frequently different parties. A merge that ignores role can fuse a management company with the family that owns the hull. Before concluding a merge is wrong, check whether the mechanism that produced it was role-aware.

**Country and flag heuristics.** Where a reconciler scopes entities by flag or country, foreign owners in a domestic register get mislabelled and a real firm splits across the boundary. Check whether a suspicious split follows a scope boundary before concluding the data is wrong. The `name_country` rule is the obvious candidate; it is also the lowest-confidence rule in the set, at 0.50.

**Over-merge is worse than under-merge.** An under-merged network under-reports a relationship, and the missing edge is recoverable. An over-merged one asserts a connection between parties that has no basis, it is invisible once made, and it is the error class that ends up in someone's compliance decision. When the evidence is balanced, the answer is under-merge.

## Completeness checks worth running

Each of these is a cohort audit. Run it at cohort level and report the cause, not the instances.

- Vessels with no owner observation at all, grouped by source and by flag. A concentration almost always means a source's ownership column was never mapped, not that the vessels are unowned. That is `UNMAPPED_SOURCE_FIELD` and it is a single finding routed to Code, not four hundred missing-owner findings.
- Coverage by source: which registries actually carry ownership data and which have never contributed an edge. A source with vessels and zero owner links is the cheapest high-value finding in the whole audit.
- Entities appearing exactly once in the corpus, and whether that is genuine or a normalisation failure. Cross-check against `entity_confidence = 'low'`, which is the machine's own name for the same cohort.
- Clusters whose member observations disagree about country of incorporation. The machine already flags the identifier version of this as `imo_company_conflict`; the country version has no flag and has to be asked for.
- Ownership chains terminating at a jurisdiction rather than a natural or legal person, and whether a further hop exists in any held source. If it does, the gap is a data problem. If it does not, the gap may be the structure.
- Vessels whose owner changed at the same moment as flag and name. A coordinated identity change is worth attributing, and it is the single pattern that most justifies the corpus existing.
- Entities carrying a relationship edge but no vessel link, or vessel links but no edge in either direction. Both are shapes the rebuild should not produce.

## Query shapes

The named MCP tools resolve source labels, capture dates, and their own caveats. Prefer them for anything that reaches a finding. `sql_query` is for the cohort questions the named tools cannot express, and it carries none of that metadata, so you supply it.

Confirm column names with `corpus_schema` before writing any of these. They are shapes, not statements known to run today.

Owner-edge coverage per source is the highest-yield sweep: count vessels contributed by a source against owner links attributable to it, and read the sources contributing many of the first and none of the second.

Single-observation entities are a join from `pipeline.entities` through `pipeline.entity_clusters` to `pipeline.stage_obs`, counting distinct `source_id` per cluster. That is the same aggregate the confidence ladder uses, so a disagreement between your count and `entity_confidence` is itself a finding.

Country disagreement within a cluster is a `COUNT(DISTINCT value)` over `pipeline.entity_attributes` where `attribute = 'country'`, grouped by `entity_id`, keeping the rows above one. Read the renderings before calling it a disagreement: two spellings of one country are one value.

Chain termination is a walk over `pipeline.entity_relationships` from a vessel's `owner` link outward, one hop at a time. `entity_network` never expands recursively, by design, and neither should you without saying how many hops you took and what each one rested on.

Coordinated identity change is a comparison of `valid_from` on `pipeline.vessel_attributes` rows for `name` and `flag` against `valid_from` on the `owner` row in `pipeline.vessel_entity_links` for the same vessel. Clustering within a window is the signal; the window has to be stated in the finding because the answer depends on it.
