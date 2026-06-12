---
name: improve-component-architecture
description: Find component-architecture friction in a codebase - domain logic that should sit behind a component facade, components that depend on each other, cross-context coordination that should be extracted into a Business Transaction, and persistence (writes or complex reads) that has escaped its owner. Driven by the project's own architecture specifications when they exist, and informed by a project components map when one exists. Use when the user wants to align a codebase with the Component / Business Transaction architecture, seal leaky facades, decouple tangled components, extract BTX, or reclaim escaped queries.
---

# Improve Component Architecture

STARTER_CHARACTER = 🧩

Surface architectural friction and propose **componentizing opportunities** - refactors that move domain-bound logic behind sealed component facades and lift orchestration into Business Transactions. The aim is testability and AI-navigability: each component understood through its facade alone, each business operation read top-to-bottom as a BTX.

## The project's specifications come first

Before anything else, check whether the project ships **architecture specification files** (commonly a `specifications/` directory - e.g. `components.md`, `business-transactions.md`, `workflows.md`, `testing.md`). When they exist, **they are authoritative for the run**: their vocabulary, rules, naming, error families, and conventions replace this skill's generic defaults, and they may define primitives this skill doesn't (a project's `workflows.md` defines a *Workflow* this glossary never mentions). The glossary below is the **default dialect, applied only when the project ships no specs.** See [SPECIFICATIONS.md](SPECIFICATIONS.md) for discovery, precedence, and how the specs drive every step.

## Glossary

This is the **default** vocabulary - it governs when the project has no specifications of its own (otherwise the specs win, per [SPECIFICATIONS.md](SPECIFICATIONS.md)). Use these terms exactly in every suggestion. Consistent language is the point - don't drift into "service," "module," "layer," or "API." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Component** - a unit of behaviour hiding domain-bound logic. Exposes a **facade**, **types**, and **errors**, and nothing else. Owns its models and persistence. Vertical (business) or horizontal (infra / cross-cutting). Implementation-agnostic (hexagonal, CQRS, plain functions…). Instantiated with all dependencies injected through the constructor.
- **Facade** - the only entrypoint to a component's behaviour. Callers know the facade, its types, and its errors - never the internals. Its methods speak the bounded domain's semantics, not the internals' (`place_order`, not `insert_order_row`).
- **Types** - the data a facade accepts and returns. May include framework models (e.g. ActiveRecord) when fighting the framework costs more than it's worth - but querying and mutating those models stays inside the component.
- **Errors** - every facade error belongs to one cohesive family: **Unexpected** (bubble up), **Permanent** (known failure mode, handled upstream), or **Temporary** (known failure mode, signals the operation can be retried).
- **Business Transaction (BTX)** - composes components by orchestrating facade calls into a business operation. The entrypoint for anything a user action or an automated process triggers. **Pure orchestration** - no domain logic. Constructor-injected dependencies; implements a `call(input)` interface where `input` is an `Input` subtype and the optional return is an `Output` subtype.
- **Library** - a gap-filling reusable (logging wrapper, event bus, DNS client). Not a component; freely importable by anyone. The "language feature we wish existed" test.
- **Ownership** - each component owns its models and persistence exclusively; no shared tables, and no complex reads (queries) of those models from outside the owner.
- **Autonomy** - a component depends on no other component (vertical *or* horizontal). All cross-component coordination lives in a BTX.

When the project documents a **components map** (which components exist, how they group around business concerns / domains, which are vertical and which horizontal), read it first and let it inform every placement decision - where new behaviour belongs, which component owns a model, which BTX a coordination becomes. See [COMPONENTS-MAP.md](COMPONENTS-MAP.md). The specifications and the map are different documents doing different jobs - the specs are the *rules*, the map is the *inventory*; both are optional and both are read before exploring.

The principles below are the **default** tests, derived from this skill's glossary. When the project ships specifications, prefer the boundary tests written *in the specs* (most spec sets include an explicit "tests for the boundaries" section) and apply these only where the specs are silent. See [LANGUAGE.md](LANGUAGE.md) for the full list.

- **Autonomy test**: editing component A must never force an edit in component B. If it does, A and B are coupled - lift the coordination into a BTX.
- **Facade test**: a caller does its job knowing only the facade, types, and errors. If it reaches past them, the facade leaks.
- **Orchestration test**: delete the BTX. If only *sequencing* is lost, it was a real BTX. If a *domain rule* is lost, logic had leaked up out of a component.
- **Ownership test**: exactly one component writes each table - *and* one component owns its complex reads. A complex query (e.g. a gnarly ActiveRecord chain) over a component's models living outside that component is escaped ownership; it belongs as a semantically-named facade method.
- **Coordination test**: where two or more bounded contexts are coordinated (a controller, a job, a callback calling several components in sequence), that coordination *is* a BTX that hasn't been named yet - extract it.

## Process

### 1. Explore

First, load the project's guidance, in this order:

1. **Specifications** (if present) - read every spec file and extract its primitives, rules, boundary tests, and conventions (see [SPECIFICATIONS.md](SPECIFICATIONS.md)). These define the vocabulary and the rules for the entire run. Absorb any primitive this skill doesn't define (e.g. a *Workflow*) and hunt for friction against its rules too. Announce what you loaded.
2. **Components map** (if present) - read it (see [COMPONENTS-MAP.md](COMPONENTS-MAP.md)); it names the components, their domain grouping, and the vertical/horizontal split. Every candidate's placement and ownership reasoning should reference it.

When specifications exist, the signals below become the *spec rules turned negative* - each "must / must not" is a violation to look for, each spec boundary test is a probe to apply, and they take precedence over this generic list. The list below is what you hunt for by default, when the project has no specs of its own:

Then use the Agent tool with `subagent_type=Explore` to walk the codebase. Don't follow rigid heuristics - explore organically and note where you experience friction:

- **Inter-component dependencies** - where one component imports, calls, or reaches into another instead of being composed by a BTX. (A shared *library* is fine; a shared *component* is the smell.)
- **Business logic in a BTX** - where domain rules (validation, pricing, state transitions) live in the orchestration layer instead of behind a facade.
- **Leaky facades** - where callers depend on a component's models, persistence, or internal helpers rather than its facade + types + errors.
- **Shared / escaped persistence** - where two components write the same tables/models, *or* where complex reads (heavy ActiveRecord chains, hand-built joins) over a component's models live outside the owning component instead of as a named facade method.
- **Un-extracted BTX** - where coordination across two or more bounded contexts lives in a controller, job, callback, or component instead of in a named BTX with a `call(input)` interface.

Also note the softer signals: vertical vs horizontal misclassification (cross-check against the map), behaviour that has no component home and is smeared across callers, facade methods named after internals rather than the domain, and facade errors that don't fall cleanly into the Unexpected / Permanent / Temporary family.

Apply the boundary tests to anything you suspect - the spec's own tests when specifications exist, otherwise the four default tests above. A "yes, this couples / leaks / escaped" is the signal you want.

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/component-architecture-review-<timestamp>.html` so each run gets a fresh file. Open it for the user - `xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows - and tell them the absolute path.

The report uses **Tailwind via CDN** for layout and styling, and **Mermaid via CDN** for diagrams where a graph/flow/sequence reliably communicates the structure. Mix Mermaid with hand-crafted CSS/SVG visuals - use Mermaid when relationships are graph-shaped (component dependency graphs, BTX call sequences), and hand-built divs/SVG when you want something more editorial (facade cross-sections, ownership maps, coupling collapse). Each candidate gets a **before/after visualisation**. Be visual.

For each candidate, rendered as a card:

- **Files** - which files/components/BTX are involved
- **Problem** - why the current architecture is causing friction (which signal, and the specific spec rule it violates when specifications exist, otherwise in glossary terms)
- **Solution** - plain English description of what would change
- **Benefits** - explained in terms of autonomy, encapsulation, ownership, and composability, and how tests would improve
- **Before / After diagram** - side-by-side, custom-drawn, illustrating the violation and the fix
- **Recommendation strength** - one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with a **Top recommendation** section: which candidate you'd tackle first and why.

**Use the codebase's own domain nouns for components, and the project's specifications (or [LANGUAGE.md](LANGUAGE.md) as the default) for the architecture vocabulary.** If the domain has "Order," talk about "the Order component" and "the Checkout BTX" - not "the OrderService," not "the order module." When specifications exist, speak their exact dialect: their primitive names, error classes (`Dnsimple::Error::PermanentError`, not a generic "Permanent"), naming conventions (`get_` / `list_`), and file/namespace layout - so each Solution reads as something a developer could apply directly against that spec.

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.

Do NOT propose facades yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them - what behaviour the component owns, where the facade sits, what types and errors it exposes, which models and persistence (writes *and* complex reads) it absorbs, and which BTX composes it.

When specifications exist, **check every proposed design against them, point by point** - naming, error families, file layout, transaction ownership, testing style, whatever the relevant spec mandates. If the right shape for a candidate is something the specs don't yet cover, or two specs contradict each other, that gap *is* a finding: propose the specification change alongside the code change.

- **Placement.** When a candidate creates a new component or moves legacy code into one, consult the **components map** to decide where it belongs and whether it's vertical or horizontal. If the map is missing the new component, propose adding it (see [COMPONENTS-MAP.md](COMPONENTS-MAP.md)).
- **Sealing a leaky facade or extracting a new component?** Decide what stays domain-bound behind the facade and what is genuinely a reusable *library*. Facade methods speak the domain; dependencies are constructor-injected. See [COMPONENTIZING.md](COMPONENTIZING.md) for how to extract and seal safely.
- **Lifting logic into a BTX, or extracting a new BTX?** Confirm the BTX stays pure orchestration - every domain rule lands behind a facade, the BTX only sequences facade calls and routes their errors/results, exposes a `call(input)` interface with an `Input` subtype, and constructor-injects its components.
- **Errors.** Sort every facade error the candidate touches into the Unexpected / Permanent / Temporary family - that decision drives whether a BTX bubbles, handles, or retries.
- **Want to explore alternative facades for the component?** See [FACADE-DESIGN.md](FACADE-DESIGN.md).

Side note on grey-box testing: as the design crystallizes, sketch the tests too - components and BTX are tested grey-box (DB/filesystem in setup and assertions, no internal leakage). See the testing section of [COMPONENTIZING.md](COMPONENTIZING.md).
