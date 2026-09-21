# Task subagent

The prompt template the orchestrator hands to each task subagent. It runs on the **model named in the card's Model field** (default `haiku`, stepping up to `sonnet` when the work needs judgement and to `opus` only for genuinely hard cards; an absent field defaults to `haiku` for back-compat) in the background in its own git worktree, cut from the card's **Base** branch — often `main`, but a feature branch when the card targets one. The harness names the worktree and its branch (`worktree-<name>`); the slug never reaches the branch name, so read the branch off the harness's report rather than constructing it. Substitute the bracketed values.

The card is the subagent's whole world — it was written by an Opus analysis pass precisely so the executor can act without re-deriving scope or hunting for entry points, which is what lets most cards run on a cheap model. The prompt below steers the agent to that brief, bounds its exploration, and holds it to the acceptance criteria.

## What to pass

- `model:` the card's **Model** field (default `haiku` if absent).
- Absolute path to the task file (in `tasks/doing/`).
- The card's **Base** branch (the branch the worktree was created from; the subagent never needs to merge into it, but state it so the agent knows its integration target).
- The worktree path and branch **as the harness reported them** when `isolation: "worktree"` — not a name derived from the slug. In this repo the `WorktreeCreate` hook puts the worktree at `<repo-parent>/worktrees/<name>` on branch `worktree-<name>`; elsewhere it defaults to `<repoRoot>/.claude/worktrees/<name>` on the same branch name. Either way the branch is the harness's to name. Keep both values in the task table: the merger is handed them verbatim, and a merger sent a branch that does not exist has to go hunting for the real one.

## Prompt

> You are working task **[slug]**, in an isolated git worktree branched from `[base]` (the branch this work will be merged back into). Everything you need is in the card at `[task-file-path]` — it was written for you to execute directly.
>
> 1. Read the whole card first. **Scope** and **Pointers** are your map: open the files/areas they name before anything else, and treat **Done when** as the literal definition of finished. The card's top half is your spec — do not widen it.
> 2. Stay inside the card's scope. If the listed pointers are enough, do not explore the rest of the repo; reach wider only when a pointer is wrong or insufficient, and note that gap in your handoff. Do the work entirely within this worktree.
> 3. Commit your changes in the worktree with clear messages. Do not push, do not touch other branches, do not run merges. If a commit fails because 1Password blocks the signing prompt — it cannot be approved in this non-interactive background run — always retry the same commit with signing disabled (`--no-gpg-sign`, or `-c commit.gpgsign=false`). Defer to an unsigned commit rather than leaving the work uncommitted; only fall back after a block, never disable signing pre-emptively.
> 4. **Commit rather than stash, and never end your turn with your work set aside.** Your worktree is removed when your turn ends and it holds no work, so a turn that ends while your changes sit in a stash destroys the worktree out from under you. If you need the base state to compare against - to reproduce a failure before your fix, say - commit your work first and check out the base in a scratch clone or a second worktree, or make the comparison and restore the stash before you stop. For the same reason, never return control while waiting on something you started yourself: nothing will wake you, and the wait is what puts you in that state. Wait for it inline, or abandon it and say so.
> 5. Check your work against every **Done when** item before finishing.
> 6. Write a handoff into the task file (see format below) so the merger can integrate your work and update the changelog.
> 7. Stamp the task file's Timeline `agent finished:` field with the current local time (`date '+%Y-%m-%d %H:%M'`).
> 8. If you cannot complete the task, still write a handoff describing how far you got, which acceptance criteria are unmet, and what blocks you, then report failure.
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
