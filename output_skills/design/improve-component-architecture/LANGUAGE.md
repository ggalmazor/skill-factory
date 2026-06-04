# Language

Shared vocabulary for every suggestion this skill makes. Use these terms exactly - don't substitute "service," "module," "layer," or "API." Consistent language is the whole point.

## Terms

**Component**
A unit of behaviour that hides domain-bound logic. Exposes a **facade**, **types**, and **errors** - and nothing else. Owns its models and persistence. Implementation-agnostic: the inside can be hexagonal, CQRS, an event-sourced aggregate, or three plain functions. The architecture cares about the seal, not the contents.
_Avoid_: service, module, package (for component).

**Vertical component**
A component organised around a business domain (Order, Pricing, Inventory). Owns business behaviour and the models/persistence behind it.

**Horizontal component**
A component organised around an infrastructure or cross-cutting *engineering* concern (Notification, Audit, Scheduling). Same rules as a vertical component - facade, types, errors, ownership, autonomy. The split is about *what the behaviour serves*, not about a relaxed dependency rule.
_Avoid_: "infra layer," "shared kernel" (for horizontal component).

**Facade**
The single entrypoint to a component's behaviour. Everything a caller must know to use the component: the operations, their **types**, their **errors**, their invariants and ordering. The facade *is* the component, as far as the rest of the system is concerned.
- **Constructor-injected.** A facade is instantiated and receives all its dependencies through the constructor. Its methods take operation input, not wiring.
- **Domain semantics, not internals.** Methods are named and shaped after the bounded domain (`place_order`, `quote_price`), never after the implementation (`insert_order_row`, `run_pricing_sql`). The facade reads as the domain's vocabulary.
_Avoid_: API, interface keyword, public methods (too narrow, or carry other-architecture baggage).

**Types**
The data shapes a component accepts and returns at its facade. Part of the exposed surface. A type *may* be a framework model (e.g. an ActiveRecord object) when avoiding it means fighting an opinionated framework harder than it's worth - exposing the AR model is acceptable, but **querying and mutating that model stays inside the owning component**. The model travels out as data; the behaviour does not leave.

**Errors**
The named failure modes a component surfaces at its facade. Callers (and BTX) handle these; they never see the internal cause. Every exposed error belongs to one cohesive, three-family taxonomy:
- **Unexpected** - not a known failure mode. Bubbles up; nobody downstream is expected to handle it.
- **Permanent** - a known failure mode that will not succeed on retry. Handled upstream (by the BTX or its caller).
- **Temporary** - a known failure mode signalling the operation can be retried (transient I/O, contention, rate limit).
A facade error that fits none of these is a smell: either it's unmodelled (make it Unexpected and let it bubble) or it's miscategorised.

**Business Transaction (BTX)**
Composes components by orchestrating calls to their facades into one business operation. The entrypoint for anything triggered by a user action or an automated process. **Pure orchestration**: it sequences facade calls and routes their results and errors. It holds no domain rules of its own.
- **Constructor-injected.** Instantiated with all its component dependencies passed through the constructor.
- **`call(input)` interface.** Implements a uniform interface: one `call` method taking a single argument `input`, which is an **`Input`** subtype. It may optionally return an **`Output`** subtype. This uniformity is what lets BTX be triggered interchangeably by controllers, jobs, and schedulers.
_Avoid_: service, use-case-with-logic, orchestrator-that-decides.

**Library**
A reusable that fills a gap in the language or runtime - a logging wrapper, an event bus, a DNS client. May be non-trivial. It is *not* a component: it carries no domain, owns no business persistence, and anyone may import it. The test: "is this a language feature we wish existed?" If yes, it's a library, and depending on it does not break autonomy.

**Ownership**
Each component owns its models and persistence exclusively. Exactly one component writes a given table, *and* owns the complex reads over it. A heavy query (a long ActiveRecord chain, a hand-built join) living outside the owner is escaped ownership just as much as a stray write - it belongs as a semantically-named facade method. Other components get what they need through the owner's facade, composed by a BTX. (Trivial reads a framework makes unavoidable are a judgement call; *complex* reads are not.)

**Autonomy**
A component depends on no other component - vertical or horizontal, absolutely. Libraries are exempt. All cross-component coordination lives in a BTX.

**Encapsulation**
What the rest of the system gains: it learns one facade, with its types and errors, and is shielded from everything behind it. Change inside a component stays inside it.

**Composability**
What BTX gives: business operations are assembled by sequencing facades, so a new operation is a new composition, not a new tangle of cross-component calls.

## Principles

- **The facade is the test surface for a component; the BTX is the test surface for an operation.** Test a component through its facade. Test a BTX by stubbing the facades it composes. If you must test *past* a facade, the component is the wrong shape.
- **The autonomy test.** Editing component A must never force an edit in component B. If it does, A and B are coupled - the coordination belongs in a BTX, or one of them isn't a real component.
- **The facade test.** A caller does its job knowing only the facade, types, and errors. Reaching past them - into a model, a table, a helper - means the facade leaks.
- **The orchestration test.** Delete the BTX. If only *sequencing* is lost, it was a real BTX. If a *domain rule* vanished, logic had leaked up out of a component and into the orchestration.
- **The ownership test.** Exactly one component writes each table, and owns its complex reads. Two writers - or a heavy query living outside the owner - means persistence has escaped.
- **The coordination test.** Where two or more bounded contexts are sequenced together (in a controller, job, callback, or component), that coordination *is* an un-named BTX. Extract it rather than leave it scattered.
- **The error-family test.** Every facade error sorts into Unexpected (bubble), Permanent (handle upstream), or Temporary (retry). An error that fits none is unmodelled or miscategorised.
- **Library, not component.** Reusing a gap-filling library never breaks autonomy. Reusing another *component* always does. The line is domain: a library carries none.
- **Domain semantics at the facade.** Facade method names belong to the bounded domain, not the implementation. If a method name leaks how it's stored or computed, it's the wrong name.

## Relationships

- A **Component** exposes exactly one **Facade** (plus its **types** and **errors**).
- A **Component** owns its models and persistence (**Ownership**) and depends on no other **Component** (**Autonomy**).
- A **Component** is **vertical** (business) or **horizontal** (infra / cross-cutting). Both obey the same rules.
- A **Business Transaction** composes **Components** by calling their **Facades**. It holds no domain logic.
- A **Library** may be used by any **Component** or **BTX** and does not affect **Autonomy**.
- **Encapsulation** is what callers gain from the facade; **Composability** is what BTX gains from autonomy.

## Rejected framings

- **"Component" as a UI component / framework widget**: unrelated. Here a component is a sealed unit of behaviour with a facade.
- **Horizontal components as a privileged layer everything may depend on**: no. Autonomy is absolute; horizontal components are composed by BTX like any other. Only *libraries* are freely depended upon.
- **BTX as a service that also decides**: no. The moment a BTX holds a domain rule, that rule belongs behind a facade. BTX is pure orchestration.
- **"Module / service / API"**: carry baggage from other architectures. Say **component**, **facade**, **BTX**.
