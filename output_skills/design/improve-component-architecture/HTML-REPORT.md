# HTML Report Format

The component-architecture review is rendered as a single self-contained HTML file in the OS temp directory. Tailwind and Mermaid both come from CDNs. Mermaid handles graph-shaped diagrams reliably (component dependency graphs, BTX call sequences); hand-built divs and inline SVG handle the more editorial visuals (facade cross-sections, ownership maps). Mix the two - don't lean on Mermaid for everything, it'll start to look generic.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Component architecture review - {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly */
      .facade { stroke-width: 3px; }            /* the sealed surface, drawn thick */
      .leak { stroke: #dc2626; }                /* facade leak / cross-component dep */
      .btx { background: linear-gradient(135deg, #1e3a8a, #1e40af); } /* orchestration band */
      .component { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, and a compact legend: thick-bordered box = component (its border = the facade), blue band = BTX, red arrow = facade leak or cross-component dependency, cylinder = owned persistence. No introduction paragraph - straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the glossary terms ([LANGUAGE.md](LANGUAGE.md)) without ceremony.

Each candidate is one `<article>`:

- **Title** - short, names the move (e.g. "Seal the Order facade", "Lift Checkout orchestration into a BTX", "Break Pricing → Inventory dependency").
- **Badge row** - recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the signal (`inter-component dep`, `logic in BTX`, `leaky facade`, `escaped persistence`, `escaped read`, `un-extracted BTX`).
- **Files** - monospaced list, `font-mono text-sm`.
- **Before / After diagram** - the centrepiece. Two columns, side by side. See patterns below.
- **Problem** - one sentence. Which signal, in glossary terms.
- **Solution** - one sentence. Which of the four moves (see [COMPONENTIZING.md](COMPONENTIZING.md)).
- **Wins** - bullets, ≤6 words each. e.g. "Order autonomy restored", "BTX back to pure orchestration", "Pricing owns its tables".

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same - variety is part of the point.

### Mermaid graph (the workhorse for dependencies / orchestration)

Use a Mermaid `flowchart` when the point is "this component reaches into that one." Wrap it in a Tailwind-styled card. Style with classDef to colour the cross-component dependency red and the BTX blue.

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart TB
      BTX[Checkout BTX]:::btx
      BTX --> O[Order]
      BTX --> P[Pricing]
      O -.depends on.-> P
      classDef btx fill:#1e40af,color:#fff;
      classDef leak stroke:#dc2626,stroke-width:2px;
      class O,P leak
  </pre>
</div>
```

Sequence diagrams work well for BTX orchestration: "BTX → Pricing.quote → Order.place → Notification.send", showing the facade calls in order.

### Hand-built component boxes (when Mermaid's layout fights you)

Components as `<div>`s with a thick border (the border *is* the facade - draw it heavy). Internal models shown faded inside. A cross-component dependency or a facade leak is a red arrow piercing the border. The "after" shows the arrow rerouted up to a BTX band that sits *above* the components and calls into them.

### Facade cross-section (good for leaky facades)

A component as a sealed capsule. Before: callers' arrows pierce the wall, touching internal models directly. After: every arrow terminates at the facade rim; the wall is unbroken; the models sit safely inside.

### Ownership map (good for escaped persistence)

Components on top, persistence cylinders below. Before: two components both arrow into one cylinder (red). After: the cylinder belongs to one component, and the other reaches it through that component's facade (composed by a BTX).

### Orchestration lift (good for logic in BTX)

Before: the BTX band is fat, stuffed with domain-rule glyphs. After: the band is thin (just sequencing arrows) and the rule glyphs have dropped down into the owning component.

### BTX extraction (good for un-extracted BTX)

Before: a controller (or job) box with arrows fanning straight out to several components - coordination smeared across the entrypoint. After: the controller calls a single BTX band, which fans out to the same components. The entrypoint shrinks to "build Input → call → map Output".

### Escaped-read reclaim (good for complex queries outside the owner)

Before: a controller/job box holding a heavy query glyph (a tangled chain) reaching into a component's persistence cylinder. After: the query glyph moves inside the component as a named facade read method; the caller's arrow now terminates at the facade rim.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: one accent (emerald or indigo) plus red for leaks/cross-component deps, blue for BTX, amber for warnings.
- Draw a component's facade as its border, heavier than internal lines - the seal should read at a glance.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for labels inside diagrams - they should read as schematic, not as UI.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. Otherwise static.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise - but the architectural nouns and verbs come straight from [LANGUAGE.md](LANGUAGE.md). Concision is not an excuse to drift.

**Use exactly:** component, vertical, horizontal, facade, types, errors, business transaction (BTX), library, ownership, autonomy, encapsulation, composability.

**Never substitute:** service, module, layer (for component) · API, interface, public methods (for facade) · orchestrator-that-decides (for BTX) · "infra layer," "shared kernel" (for horizontal component).

**Phrasings that fit the style:**

- "Order facade leaks - callers touch its models directly."
- "Pricing depends on Inventory - autonomy broken."
- "Checkout BTX holds a domain rule; lift it behind a facade."
- "Two components write `orders`; ownership escaped."
- "Complex AR query on `invoices` in the controller - escaped read; make it a facade method."
- "Controller sequences Pricing + Order + Notification - un-extracted BTX."
- "Facade error is unmodelled - sort into Permanent and handle upstream."

**Wins bullets** name the gain in glossary terms: *"autonomy: Pricing names no other component"*, *"encapsulation: callers see only the facade"*, *"BTX back to pure orchestration"*, *"ownership: one writer per table"*. Don't write *"cleaner code"* or *"better separation of concerns"* - those terms aren't in the glossary and don't earn their place.

No hedging, no throat-clearing. If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in [LANGUAGE.md](LANGUAGE.md), reach for one that is before inventing a new one.
