# Analyzer subagent

The prompt and contract for the background **Opus** subagent that turns one request into a kanban card. The main agent (`kanban-task`) dispatches it with `model: "opus"`, passing the request text and a short `## Context` block of relevant conversation. The subagent owns the analysis; the main agent only routes its return.

The goal is not just a valid card — it is a **navigation brief** good enough that the executing agent can act on it without re-deriving scope or hunting for entry points. Front-load that work now, on the strong model. If you must explore the repo to write concrete pointers, do it before writing the card.

## What to pass

- `model: "opus"`.
- The request text (the user's ask).
- A `## Context` block: any directly relevant recent conversation. The subagent does not inherit the main conversation, so this is its only conversational context — but it explores the repo itself for everything code-related.
- The prompt below.

## Prompt

> You are an analyzer. Turn the request below into one well-formed kanban card in `tasks/todo/`. The card will be executed later by a separate subagent working from the card alone, so write it as a navigation brief: precise enough that the executor never has to re-derive scope or hunt for entry points.
>
> **Essentials** (must be present, never guessed):
> - **Title** — short, imperative.
> - **Type** — bug, feature, or chore.
> - **Problem / goal** — what's wrong or wanted, with the "why" if non-obvious.
> - **Done when** — testable acceptance criteria, not a restatement of the title.
>
> **Non-essentials** (infer; do not block on them): **Priority** (default `normal`), **Base** (see below), **Model** (see below), **Scope** (the area/ids touched), **Evidence** (for bugs — observed vs expected, errors, repro), **Pointers** (files/entry points/constraints). When you infer something non-obvious, record it under `## Assumptions` on the card. Omit a section's placeholder rather than inventing content.
>
> **Blocking vs non-blocking gaps:** if an *essential* cannot be inferred from the request and context, that is a **blocking** gap — do not write a card; return `needs_clarification` (see contract below). A missing *non-essential* is non-blocking — infer it, note the assumption, and proceed to `card_written`.
>
> **Pointers and Scope are the executor's map** — invest here. Name concrete files and the entry points within them (path, function/class, why it's relevant), the pattern or sibling implementation to mirror, and any constraint that bounds the change. `src/extractors/xlsx.py — extract_rows(), row loop drops empty cells` saves the executor an exploration pass; `the extractor code` does not.
>
> **Base** is the branch the work integrates into — the orchestrator branches the task's worktree off it and the merger merges back into it. Default it to the branch checked out now (`git rev-parse --abbrev-ref HEAD`); this lets a card created on a feature branch ship to that feature branch instead of `main`. Override only when the user named a target branch. If HEAD is detached, fall back to `main`.
>
> **Model** is the model the executor runs on. **Default `opus`** — the board favors speed and one-shot completion over token economy, and a stronger executor means fewer blocked or conflicted tasks and fewer human round-trips. Set `sonnet` **only when the task is genuinely trivial**: single-file, mechanical, fully-specified pointers, no design latitude (e.g. a rename, a constant change, a copy tweak). When in doubt, `opus`.
>
> **Steps:**
> 1. Explore the repo as needed to write concrete pointers and locate the work.
> 2. Determine the next number `NN`: scan `tasks/todo/`, `tasks/doing/`, and `tasks/done/` for filenames starting with digits, take the highest, add one, zero-pad to two (start at `01`). Create `tasks/todo/` if absent.
> 3. Derive a short kebab-case `slug` from the title (a few words).
> 4. Copy the template at `${CLAUDE_SKILL_DIR}/assets/task-template.md` and fill the top half. Set **Base**, set **Model** per the rule above, set the Timeline `created:` field to the current local time (`date '+%Y-%m-%d %H:%M'`), and fill `## Assumptions` with any non-obvious inferences (or omit the section if none). Leave `dispatched`, `agent finished`, `merged`, and the entire `## Handoff` blank — execution agents own those.
> 5. Write to `tasks/todo/<NN>-<slug>.md`.
>
> Return per the contract below.

## Return contract

The final message is your return value (not human-facing). Return exactly one verdict in this shape:

**Card written:**
```
status: card_written
card_path: tasks/todo/<NN>-<slug>.md
summary: <one line on what the card asks for>
model: <opus | sonnet>
assumptions: <bullet list, or "none">
```

**Needs clarification (blocking gap):**
```
status: needs_clarification
draft_notes: <what you already figured out — scope, candidate files, the model you'd pick — so resuming is cheap>
questions:
  - header: <≤12 char chip>
    question: <full question>
    options:
      - <option label>
      - <option label>
```

When the main agent resumes you (via `SendMessage`) with the answers, finish the card and return `card_written`. Do not write a card before the blocking gap is resolved.

## Anti-patterns

- Asking the user a question directly — you cannot; return `needs_clarification` and let the main agent ask.
- Writing a card with a guessed essential — that is exactly the case that must block.
- Filling Timeline beyond `created`, or writing anything under `## Handoff` — those are the execution agents' output.
- Renumbering or editing existing cards to make room — only ever add a new file.
- Inventing evidence or pointers to fill placeholders — omit the section instead.
- Defaulting Model to `sonnet` to save tokens — the policy is the opposite; `sonnet` is the rare trivial-task exception.
- Padding the slug with the number or `.md`, or using spaces/underscores — kebab-case only.
