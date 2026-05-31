# Task subagent

The prompt template the orchestrator hands to each task subagent. It runs on **Sonnet** in the background in its own git worktree (branch `kanban/<slug>` off `main`). Substitute the bracketed values.

The card is the subagent's whole world — it was written by an Opus analysis pass precisely so a Sonnet agent can execute without re-deriving scope or hunting for entry points. The prompt below steers Sonnet to that brief, bounds its exploration, and holds it to the acceptance criteria.

## What to pass

- `model: "sonnet"`.
- Absolute path to the task file (in `tasks/doing/`).
- The worktree path and branch (the harness sets these when `isolation: "worktree"`).

## Prompt

> You are working task **[slug]** on Sonnet, in an isolated git worktree on branch `kanban/[slug]`. Everything you need is in the card at `[task-file-path]` — it was written for you to execute directly.
>
> 1. Read the whole card first. **Scope** and **Pointers** are your map: open the files/areas they name before anything else, and treat **Done when** as the literal definition of finished. The card's top half is your spec — do not widen it.
> 2. Stay inside the card's scope. If the listed pointers are enough, do not explore the rest of the repo; reach wider only when a pointer is wrong or insufficient, and note that gap in your handoff. Do the work entirely within this worktree.
> 3. Commit your changes in the worktree with clear messages. Do not push, do not touch other branches, do not run merges.
> 4. Check your work against every **Done when** item before finishing.
> 5. Write a handoff into the task file (see format below) so the merger can integrate your work and update the changelog.
> 6. Stamp the task file's Timeline `agent finished:` field with the current local time (`date '+%Y-%m-%d %H:%M'`).
> 7. If you cannot complete the task, still write a handoff describing how far you got, which acceptance criteria are unmet, and what blocks you, then report failure.
>
> Return a final message stating success or failure and a one-line summary.

## Handoff format

Append to the task file. This is the merger's only source of truth about the work.

```markdown
## Handoff

**Summary:** <one or two sentences on what changed>

**Changes:** <key files/areas touched and why>

**Decisions:** <non-obvious choices and their rationale>

**Notes / follow-ups:** <anything the merger or a human should know; leftover TODOs>

**Changelog entry:** <a single line, written as it should appear in CHANGELOG>
```

The `Changelog entry` line is what the merger copies into `CHANGELOG`. Write it in the changelog's existing style if the task implies one; otherwise a plain imperative line.

## Principle

The subagent owns the work and the handoff; it never integrates. Integration is the merger's job, serialized across all tasks. Keep the worktree self-contained so the merge is a clean branch merge.
