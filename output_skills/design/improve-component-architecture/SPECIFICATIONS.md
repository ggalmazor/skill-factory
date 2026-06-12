# Project Specifications

Some projects ship **specification files** that define their architecture in their own exact terms - the primitives that exist, the rules each must obey, the naming and file conventions, the error families, the testing approach. When a project has them, they are **authoritative for the run**: they replace this skill's generic vocabulary and rules with the project's own, and they may introduce primitives this skill has never heard of.

Example layout (from `dnsimple-app/specifications/`):

```
specifications/
  components.md            ← what a Component is, its rules, its tests
  business-transactions.md ← what a BTX is, its interface, its rules
  workflows.md             ← a primitive this skill doesn't define on its own
  testing.md               ← how components / BTX / workflows are tested
```

The filenames, the set of documents, and the format are the project's choice. Read whatever is there; don't impose a schema.

## Precedence

This skill ships a built-in vocabulary and ruleset in [LANGUAGE.md](LANGUAGE.md). That built-in is the **default** - it governs only when the project ships no specifications.

When specifications **are** present:

1. **The specs win on every conflict.** If a spec says facade methods are named `get_` / `list_` and YARD-documented, that is the rule - not this skill's softer "domain-semantic names" phrasing. If a spec defines error families as `Dnsimple::Error::{Temporary,Permanent,Unexpected}Error`, that exact hierarchy is what findings reference, not the generic Unexpected / Permanent / Temporary triple.
2. **The specs extend.** A spec may define a primitive outside this skill's Component / BTX / Library / Workflow model (the dnsimple specs define a **Workflow** as a first-class long-running primitive). Absorb it: treat it as a real architectural element, look for friction against *its* rules, and use *its* name in the report. Never flatten a spec-defined primitive into the nearest built-in just because the built-in is familiar.
3. **The built-in fills gaps.** Where the specs are silent on something this skill knows about (say, the autonomy test, and the spec never spells it out), the built-in `LANGUAGE.md` still applies as the default - but say so, and treat a conspicuous gap as a finding (see below).

The one-line mental model: **`LANGUAGE.md` is the default dialect; project specifications are the dialect actually spoken on this codebase.** Speak the codebase's.

## What to extract from the specs at load time

Before exploring, read every spec file and pull out, in the project's own words:

- **Primitives** - the full set of architectural elements the project names (Component, BTX, Workflow, Library, …), including any this skill doesn't define.
- **Rules per primitive** - the hard constraints ("a component must not query another component's AR models", "a BTX owns transaction wrapping", "no DB transactions inside a facade").
- **Boundary tests** - most specs include an explicit "tests for the boundaries" / heuristics section. These *are* the friction signals for Step 1 - prefer them over this skill's hardcoded four tests.
- **Conventions** - naming (`get_`/`list_`/`new_`), file/namespace layout (`lib/dnsimple/<name>/`), error base classes, doc requirements (YARD), test location and style (grey-box, state-based assertions).
- **Human-judgment vs mechanical split** - specs often mark which decisions are mechanical (apply directly) and which need reasoning (raise in the grilling loop). Respect that split.

Announce what you loaded: "Loaded 4 specifications (components, business-transactions, workflows, testing). Using their vocabulary and rules for this run."

## How the specs drive each step

- **Step 1 (Explore).** The friction signals you hunt for are the spec rules turned negative: each "must / must not" is a violation to look for, each boundary test is a probe to apply. A spec-defined primitive (Workflow) gets its own signal set. The generic four tests apply only where the specs leave a concept unaddressed.
- **Step 2 (Report).** Every card speaks the project's dialect: its primitive names, its error classes, its file paths, its naming conventions. A "Solution" should read as something a developer could apply against *this* spec - "extract a `Checkout` BTX with a `call(input)` interface under `lib/dnsimple/`", not "make an orchestrator." Cite the spec rule a candidate violates.
- **Step 3 (Grill).** Check every proposed design for spec conformance, point by point. When a candidate's right shape is something the specs don't yet cover, or two specs contradict, that is itself a finding - propose the spec change alongside the code change.

## Specifications vs the components map

Two different optional documents, both read first, doing different jobs:

- **Specifications** (this file) - the **rules and language**: what a Component *is*, what a BTX *must* do. Read [COMPONENTS-MAP.md](COMPONENTS-MAP.md) for the other.
- **Components map** - the **inventory**: which components actually exist, how they group, vertical vs horizontal, who owns which models.

Specs tell you the laws; the map tells you the current population. A finding usually cites both: "this query violates the ownership rule (*spec*) and belongs to the Domains component (*map*)." A project may ship either, both, or neither.
