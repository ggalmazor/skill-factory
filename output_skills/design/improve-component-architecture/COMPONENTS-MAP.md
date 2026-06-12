# Components Map

An optional project document that describes the system's components. When it exists, this skill reads it first and lets it inform every placement and ownership decision. When it's missing, the skill works without it - but proposes adding entries as new components and BTX are designed.

This document is the **inventory** (which components exist); the project's **specifications** are the **rules** (what a component must be). They are separate, both optional, both read before exploring. See [SPECIFICATIONS.md](SPECIFICATIONS.md).

The map is read-only guidance for the skill; the skill never silently rewrites it. It *proposes* additions during the grilling loop (Step 3), framed as "this map is missing the Pricing component - add it?".

## What the map records

- **Components** - the components that exist, each with its name and a one-line responsibility.
- **Grouping** - how components organise around business concerns (modules / domains / bounded contexts). Which components belong to which domain.
- **Vertical / horizontal** - which components are vertical (business) and which are horizontal (infra / cross-cutting), so placement and the autonomy rule are unambiguous.
- **Ownership** - optionally, which models / tables each component owns. Useful for the ownership test.

The exact filename and format are the project's choice (commonly `docs/components-map.md`, a diagram, or a structured file). Read whatever the project has; don't impose a schema.

## How the skill uses it

- **Placement** - when a candidate creates a new component or moves legacy code into one, the map decides where the code belongs and whether the home is vertical or horizontal. Don't guess a placement the map already answers.
- **Ownership** - when reclaiming escaped persistence (writes or complex reads), the map says which component is the rightful owner.
- **Coordination** - when extracting a BTX, the map confirms the components being coordinated already exist; if the BTX needs one that isn't on the map, that gap is itself a finding.
- **Classification checks** - vertical/horizontal misclassification in the code is checked against the map, not invented fresh.

## Sketch of a minimal map

```md
# Components Map

## Sales (domain)
- **Order** (vertical) - places and tracks orders. Owns `orders`, `order_lines`.
- **Pricing** (vertical) - quotes and applies prices. Owns `price_lists`, `discounts`.

## Cross-cutting
- **Notification** (horizontal) - delivers email/SMS. Owns `notifications`.
- **Audit** (horizontal) - records domain events. Owns `audit_entries`.

## Business Transactions
- **Checkout** - composes Pricing + Order + Notification.
```

If the project has nothing like this, treat building it as a worthwhile side effect of the architecture work - propose the first entries as the skill surfaces components.
