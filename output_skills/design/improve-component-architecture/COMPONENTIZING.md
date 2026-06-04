# Componentizing

How to extract, seal, and decouple components safely, and how to lift orchestration into a BTX. Assumes the vocabulary in [LANGUAGE.md](LANGUAGE.md) - **component**, **facade**, **BTX**, **library**, **ownership**, **autonomy**.

## The four moves

Every candidate this skill surfaces resolves to one (or a sequence) of these.

### 1. Seal a leaky facade

A caller depends on a component's models, persistence, or internal helpers. Pull that dependency back to the facade.

- Decide what the caller actually needs and express it as a facade operation returning **types** (not internal models) and raising named **errors**.
- Replace direct model/table access with facade calls.
- The internal model stops being exported. If nothing outside the component references it, it's now genuinely owned.

The facade test: after sealing, the caller works knowing only the facade, types, and errors.

### 2. Break an inter-component dependency

Component A calls or imports component B. Autonomy is absolute - this must go.

First classify what A reuses from B:

- **It's domain behaviour** → A and B are two components that a **BTX** should compose. Lift the coordination: the BTX calls B's facade, then A's facade (or vice versa), passing data between them. Neither component names the other.
- **It's a gap-filling reusable** (logging, events, ids, dns) → it was never a component. Extract it as a **library**. Both A and B may then depend on the library freely; autonomy holds.

The dividing test: does the shared thing carry domain, or is it a language feature you wish existed? Domain → BTX composition. Gap-filler → library.

### 3. Lift logic into a BTX

A BTX holds a domain rule (validation, branching on business state, a calculation). BTX must be pure orchestration.

- Find the facade that *should* own the rule - usually the component whose models the rule reasons about.
- Move the rule behind that facade, exposing it as an operation (or folding it into an existing one) with the right errors.
- The BTX now only sequences facade calls and routes their results/errors.

The orchestration test: after lifting, deleting the BTX loses only sequencing - no domain rule disappears with it.

### 4. Reclaim escaped persistence (writes and complex reads)

Two components write the same tables/models, or a complex read over a component's models lives outside the owner. Ownership must be exclusive over both.

- Choose the owning component - the one whose domain the data belongs to (cross-check the components map).
- The other side stops touching the tables. A stray write becomes a facade operation; a stray complex query becomes a **semantically-named facade read method** (`overdue_invoices_for(customer)`, not a raw AR chain in a controller). The caller gets results through the facade, composed by a BTX.
- In Rails specifically: hunt for heavy ActiveRecord chains, scopes, and joins over a component's models sitting in controllers, jobs, serializers, or other components. Each is a candidate facade read method.
- If both sides genuinely need to *own* parts of the data, the table is doing two jobs - split it along the component line.

The ownership test: after reclaiming, exactly one component writes each table and owns its complex reads.

### 5. Extract a BTX from scattered coordination

A controller, job, callback, or component sequences several bounded contexts inline. That coordination is a BTX that was never named.

- Name the business operation and create a BTX for it: constructor-injected components, a `call(input)` method, `input` an `Input` subtype, optional `Output` return.
- Move the sequence of facade calls (and only the sequencing - no domain rules) into `call`. Domain rules encountered on the way get lifted behind the relevant facade first (move 3).
- The original site (controller action, job perform, callback) shrinks to: build the `Input`, instantiate the BTX with its components, call it, map the `Output`.
- Cross-check the components map: the BTX coordinates components that already exist there; if it needs one that doesn't, propose adding it.

The coordination test: after extraction, no controller/job/callback sequences multiple components directly - they each invoke one BTX.

## Sequencing the moves

Order matters when a candidate needs several:

1. **Seal facades first.** You can't safely break a dependency or reclaim persistence while callers still reach past the facade.
2. **Reclaim persistence next.** Exclusive ownership is the precondition for autonomy - shared tables *are* a hidden dependency.
3. **Break inter-component dependencies / lift to BTX last.** Once facades are sealed and data is owned, the remaining coupling is pure coordination, which moves cleanly into a BTX.

## Testing strategy: grey-box at the facade and the BTX

Both components and BTX are tested **grey-box**: tests may touch real infrastructure (database, filesystem, queues) in setup and assertions, but they must not leak internal implementation details. You drive and observe through the facade / the `call(input)` interface; you may use the DB to arrange state and to verify outcomes, but you never assert on private structure.

- Test each component **through its facade**, against real persistence. Arrange state in the DB, call a facade method, assert on the returned **types**, the raised **errors** (sorted into the Unexpected / Permanent / Temporary family), and the observable persistence outcome. Never assert on internal models or private methods.
- Test each BTX **through `call(input)`**, with its components wired to real (or realistically substituted) facades. Verify the business operation's observable outcome and its error routing - bubble Unexpected, surface Permanent, retry Temporary. The BTX holds no domain rules, so there are no domain-rule assertions at this level.
- **Collaboration testing - only when scenarios explode.** Prefer testing each BTX in isolation. *Only* when conditional combinations across collaborating BTX/components blow up the scenario count, test the BTX in collaboration (exercising the real collaborators end to end), and keep a few isolated grey-box tests for the happy path and any mission-critical path. This is the exception, not the default - note explicitly when a candidate's tests will need it.
- Old tests that reached past a facade (asserting on internal models, hitting private methods) become waste once grey-box facade/BTX tests exist - delete them.
- Tests should survive internal refactors - they describe facade/operation behaviour observed through the interface, not implementation. If a test has to change when the implementation changes (not the behaviour), it was leaking internals.
- A library is tested in isolation like any reusable; it carries no domain, so its tests assert mechanics, not business rules.
