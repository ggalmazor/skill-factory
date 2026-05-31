---
name: kanban-task
description: "Creates a kanban task card in tasks/todo/ from a short description: assigns the next sequence number and fills type, priority, scope, problem, and acceptance criteria, leaving the agent-filled sections blank. Use when adding a task to the kanban board."
argument-hint: "[task description]"
---

STARTER_CHARACTER = 📝

# Kanban task

Turn a request into a well-formed kanban card in `tasks/todo/`, ready for the `kanban` skill to dispatch. The card template is `${CLAUDE_SKILL_DIR}/assets/task-template.md`.

This analysis runs on **Opus**: decomposing a request and locating the work in the codebase is the reasoning-heavy step. The card it produces is then executed by a **Sonnet** subagent that the `kanban` orchestrator spawns. So the goal here is not just a valid card — it is a *navigation brief* good enough that a Sonnet agent can act on it without re-deriving scope or hunting for entry points. Front-load that work now, while on the stronger model. If you must explore the repo to write concrete pointers, do it before writing the card.

## Inputs

Take the task from `$ARGUMENTS` and the surrounding conversation. Fill what you can infer; these are essential and must not be guessed:

- **Title** — short, imperative.
- **Type** — bug, feature, or chore.
- **Problem / goal** — what's wrong or wanted, with the "why" if non-obvious.
- **Done when** — acceptance criteria.

If any essential is missing and cannot be inferred from context, ask the user — batch all gaps into a single round of questions rather than drip-asking, and proceed once answered. Don't block on the non-essentials: infer **Priority** (default `normal`), **Scope** (the area/ids touched), **Evidence** (for bugs — observed vs expected, errors, repro), and **Pointers** (known files/constraints), and omit a section's placeholder rather than inventing content.

**Pointers and Scope are the Sonnet executor's map** — invest here. Name concrete files and the entry points within them (path, function/class, and why it's relevant), the pattern or sibling implementation to mirror, and any constraint that bounds the change. A pointer that says `src/extractors/xlsx.py — extract_rows(), row loop drops empty cells` saves the subagent an exploration pass; `the extractor code` does not. The more precisely the change is located now, the less a Sonnet agent has to rediscover at execution time.

## Steps

1. Determine the next number `NN`: scan `tasks/todo/`, `tasks/doing/`, and `tasks/done/` for filenames starting with digits, take the highest, add one, zero-pad to two (start at `01`). Create `tasks/todo/` if absent.
2. Derive a short kebab-case `slug` from the title (a few words).
3. Copy the template, fill the top half, and set the Timeline `created:` field to the current local time (`date '+%Y-%m-%d %H:%M'`). Leave `dispatched`, `agent finished`, `merged`, and the entire `## Handoff` blank — agents own those.
4. Write to `tasks/todo/<NN>-<slug>.md`.
5. Report the path and a one-line summary of the card.
6. Ensure the board is running. The `kanban` orchestrator never self-terminates — it idle-polls `tasks/todo/` — so a card dropped here is picked up automatically by a running board. If a kanban loop is already active in this session, just note the card will be absorbed on its next poll. If none is running, start one by following `kanban/SKILL.md` directly (it sets `disable-model-invocation`, so read and run its control loop inline rather than via the Skill tool). Never start a second loop when one is already active — duplicate orchestrators race on `main`.

## Principles

- One card, one unit of work. Split unrelated asks into separate cards rather than one sprawling card.
- Acceptance criteria are testable statements, not a restatement of the title.
- Write `Scope` concretely (ids, files, area) — it is the agent's first orientation.

## Anti-patterns

- Filling Timeline beyond `created`, or writing anything under `## Handoff` — those are the execution agents' output, not creation input.
- Renumbering or editing existing cards to make room — only ever add a new file.
- Inventing evidence or pointers to fill placeholders — omit the section instead.
- Padding the slug with the number or `.md`, or using spaces/underscores — kebab-case only.
