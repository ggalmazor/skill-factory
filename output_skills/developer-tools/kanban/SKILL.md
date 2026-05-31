---
name: kanban
description: "Orchestrates a file-based kanban board: advances task files through todo/doing/done, running each in a parallel git-worktree subagent with serialized merges to main. Use to autonomously work a queue of task files, or when the user mentions a kanban or task-board workflow."
argument-hint: "[max-parallel]"
disable-model-invocation: true
---

STARTER_CHARACTER = 📋

# Kanban

Drive a `tasks/` board to completion: each task file becomes a background subagent working in its own git worktree, and finished work is merged into `main` one task at a time.

## Board layout

- `tasks/todo/` — queued tasks, one Markdown file each
- `tasks/doing/` — in progress
- `tasks/done/` — merged

A task file is in exactly one directory at any moment. The board is local orchestration state; do not commit board moves onto task branches.

## On trigger

1. Resolve parallelism `N`: the number the user passed, else `4`. If the user passed none, proceed with 4 and tell them so, noting they can override (e.g. `/kanban 6`).
2. Confirm a git repo with a `main` branch and a `tasks/todo/` directory exist. Create `tasks/doing/` and `tasks/done/` if missing.
3. Initialize an in-context ledger (below) and enter the control loop.

## Invariants

These hold for the entire run. Violating them corrupts the board or `main`.

- At most `N` task subagents running concurrently.
- At most **one** merger subagent active at any time. Never spawn a second to clear backlog faster — queue completed tasks instead. Serialization is what protects `main`.
- Each task subagent runs in the background in its own worktree branched from `main`.
- Only merge a task whose agent reported success. A failed or conflicted task stays in `doing`, marked blocked, surfaced to the user.

## Ledger

Track in context, one row per task: slug, task-file path, worktree path, branch, and state — one of `pending | working | awaiting-merge | merging | done | blocked`. Also track the merge queue (ordered, oldest first) and whether a merger is active.

## Control loop

Run one iteration on each of: the initial trigger, every background completion notification, and every scheduled wake. Per iteration:

1. **Reconcile completions.** For each task subagent that finished, record its reported result and mark it `awaiting-merge` (success) or `blocked` (failure); append blocked tasks' reason to their task file and surface to the user.
2. **Drain the merge queue.** If no merger is active and ≥1 task is `awaiting-merge`, spawn a merger for the oldest one (see [references/merger-subagent.md](references/merger-subagent.md)) in the background; mark it `merging` and the merger active.
3. **Apply a finished merger.** On a merger's completion: success → stamp the card's Timeline `merged:` with the current local time, move its task file `doing/`→`done/`, mark `done`; conflict → leave the file in `doing/`, mark `blocked`, surface to the user. Clear the merger-active flag either way.
4. **Backfill.** While running task subagents < `N` and `tasks/todo/` is non-empty: select the next card by **priority** (`high` before `normal` before `low`, ties broken by ascending `NN` filename order), move it `todo/`→`doing/`, stamp its Timeline `dispatched:` with the current local time, and spawn a task subagent (see [references/task-subagent.md](references/task-subagent.md)) in the background. Count in-flight agents before each spawn so `N` is never exceeded.
5. **Status and cadence.** If any task is `pending`, `working`, `awaiting-merge`, or `merging`, emit a status line (below) and schedule a wake ~60s out. If nothing remains in those states and `todo/` is empty, the board is **idle, not finished**: emit an idle status line, report cumulative counts, and schedule a longer wake (~5min) to poll for newly created cards. The loop never self-terminates — it keeps polling so cards dropped into `todo/` later (e.g. by `kanban-task`) get picked up automatically. Stop only when the user explicitly says to stop.

Drain the merge queue (step 2) before backfilling (step 4) so merges are never starved by new work.

## Spawning subagents

Both kinds run in the background so their completion notifies the loop; treat each notification as input to the next iteration.

- **Task subagent:** spawn with `model: "sonnet"`, branch `kanban/<slug>` off `main`, worktree isolation. It does the work, commits in the worktree, and writes a handoff into its task file. Full prompt and handoff format: [references/task-subagent.md](references/task-subagent.md).
- **Merger subagent:** spawn with `model: "sonnet"` (mechanical integration). Merges one task's branch into `main`, deletes the worktree, updates `CHANGELOG` from the handoff, and reports success or conflict. One at a time. Full prompt: [references/merger-subagent.md](references/merger-subagent.md).

## Models

The orchestrator (this loop) and any up-front planning run on **Opus** — backlog reasoning, priority calls, and conflict triage need the stronger model. The execution subagents run on **Sonnet** for cost and speed; their cards carry the navigation context Opus front-loaded (see [kanban-task](../kanban-task/SKILL.md)), so a Sonnet agent can act without re-deriving it. If a task card is thin on pointers, that is an analysis gap to fix in card creation — not a reason to upgrade the subagent.

## Status line

One compact line with per-state counts, e.g.:

`📋 working 3/4 · awaiting-merge 1 · merging 1 · todo 2 · blocked 0`

Add a sub-line naming any `blocked` task and why.

## Anti-patterns

- Spawning a second merger to drain backlog — concurrent merges race on `main`. Queue instead.
- Spawning task agents in a backfill burst without recounting in-flight agents — overruns `N`.
- Polling in a foreground sleep loop — schedule a wake and yield instead.
- Merging a task whose agent failed, or auto-resolving a merge conflict — both bypass the block-and-surface rule.
- Committing board moves (`todo`→`doing`) onto a task branch — keep board state off the code branches.
