# Facade Design

When the user wants to explore alternative facades for a chosen component, use this parallel sub-agent pattern. Based on "Design It Twice" (Ousterhout) - your first facade is unlikely to be the best.

Uses the vocabulary in [LANGUAGE.md](LANGUAGE.md) - **component**, **facade**, **types**, **errors**, **BTX**, **encapsulation**.

## Process

### 1. Frame the problem space

Before spawning sub-agents, write a user-facing explanation of the problem space for the chosen component:

- What domain behaviour the component owns (and where its models/persistence sit).
- The BTX (existing or future) that will compose it, and what those BTX need from the facade.
- The autonomy constraint: the facade must let BTX coordinate without any other component depending on this one.
- A rough illustrative code sketch to ground the constraints - not a proposal, just a way to make the operations, types, and errors concrete.

Show this to the user, then immediately proceed to Step 2. The user reads and thinks while the sub-agents work in parallel.

### 2. Spawn sub-agents

Spawn 3+ sub-agents in parallel using the Agent tool. Each must produce a **radically different** facade for the component.

Prompt each sub-agent with a separate technical brief (file paths, the behaviour the component owns, its models/persistence, the BTX that compose it, which inter-component coupling is being removed). Give each agent a different design constraint:

- Agent 1: "Minimize the facade - 1–3 operations max. Maximise behaviour owned per operation."
- Agent 2: "Optimise for the composing BTX - make the common orchestration trivial to express."
- Agent 3: "Maximise encapsulation - expose the narrowest types and errors that still let BTX coordinate; hide every model."
- Agent 4 (if applicable): "Design for a horizontal/cross-cutting concern - make the facade reusable across many BTX without leaking domain."

Include [LANGUAGE.md](LANGUAGE.md) vocabulary and the codebase's own domain nouns in the brief so each sub-agent names things consistently with the architecture language and the project's domain.

Each sub-agent outputs:

1. Facade (operations named in the **bounded domain's** vocabulary - not the internals' - plus the **types** and **errors** exposed, invariants, ordering). Dependencies are constructor-injected; method arguments carry operation input only.
2. The **error taxonomy** for the facade: each error sorted into Unexpected (bubble), Permanent (handle upstream), or Temporary (retry).
3. Usage example showing how a BTX composes it via `call(input)`.
4. What stays hidden behind the facade (models, persistence, internal helpers). Note where a framework model is deliberately exposed as a **type** while its querying/mutating stays inside.
5. How autonomy is preserved - what would otherwise have been a cross-component dependency, now expressed as a BTX composition or a library.
6. Trade-offs - where the facade is sharp, where it's thin or awkward for some caller.

### 3. Present and compare

Present designs sequentially so the user can absorb each one, then compare them in prose. Contrast by **encapsulation** (how much stays hidden), **composability** (how cleanly BTX use the facade), and the shape of the **types** and **errors**.

After comparing, give your own recommendation: which facade you think is strongest and why. If operations from different designs would combine well, propose a hybrid. Be opinionated - the user wants a strong read, not a menu.
