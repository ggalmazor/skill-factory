# GitHub: issues, PRs, review comments, commit messages

Short-form, public engineering contexts. The voice from `SKILL.md` applies directly; this file adds the scaffolding specific to each artifact. These illustrate the principle — consider what fits the actual repo and situation.

## Contents

- Issue bodies
- Pull request descriptions
- Review comments
- Issue and status comments
- Commit messages
- Formatting habits
- Evidence dumps

## Issue bodies

Lead with the why. Move symptom → mechanism → consequence → required outcome. State "Expected Behavior" or the requested fix only after the chain is laid out, so the reader understands the need before the ask.

Back the report with evidence inline — Bugsnag stacktraces, `dig` output, Datadog statistics — in fenced blocks, each followed by a one-line interpretation. Quantify exactly: counts, dates, durations, validity windows. Where both a quick patch and a proper fix exist, name the immediate one and argue for the structural one. Acknowledge the alternative you did not take, with its tradeoff.

If the repo provides an issue template, fill it verbatim. Write voice only into the prose slots; never reorder or drop sections.

## Pull request descriptions

Same why-first arc as issues. Open with the problem the PR addresses and its mechanism, then give the change list — often an "In this PR" set of bullets describing what changed and why each change is needed.

Fill the repo PR template verbatim, keeping its emoji-shortcode headers (`:mag:`, `:clipboard:`, `:shipit:`) intact even on small changes. Titles use his verbs: Tweak, Enhance, Revamp, Consolidate, Extract, Decouple, Gracefully (update/adapt), Build, Schedule, Support, Prevent, Fix, Revisit.

## Review comments

Collaborative and lightly deferential. Propose a direction, give the reasoning, and leave room to be corrected rather than asserting. Invite discussion: "Let's discuss this further if I'm mistaken, please :bow:", "@person let's do a live review of this one". `:bow:` softens a request for agreement; it is his own tic, not a template artifact.

No jokes, no sarcasm, no slang. Personality shows as politeness, not humor.

## Issue and status comments

Terse, plain, and functional — sharply different from long-form bodies. One idea per sentence. Many comments exist only to record state for later readers: what was decided, what was split out, what is now resolved.

- "Yes, I agree."
- "I've extracted individual tasks for the remaining 2 tasks."
- "I've backburnered this PR for now."
- "This ticket is no longer an issue after having released: #X, #Y, #Z."

Close loops explicitly and link the PRs/issues that resolved the work. Singular "I" is acceptable here for personal status notes; shared work is still "we".

## Commit messages

Subject in his verb set, imperative or descriptive, naming the domain concept it touches. Body, when present, follows the why-first arc compressed: what changed and why, with domain nouns capitalized and identifiers backticked. Keep it calm and factual.

Note: Guillermo's global rules forbid AI attribution or co-author trailers in commit messages. Never add them.

## Formatting habits

- Structured markdown: section headers, nested bullet lists, task checkboxes (`- [x]` / `- [ ]`) for multi-step tracking. Sub-bullets nest under parents to show dependent detail.
- Inline backticks for every identifier: class names, methods, table and column names, feature flags, literal values.
- Dense linking: cross-reference issues/PRs by `#number`; deep-link Bugsnag events, Datadog dashboards, Confluence pages, Slack threads, and exact source lines permalinked to a commit SHA. Evidence is always reachable, not just described.

## Evidence dumps

When sharing test results, post the raw data in fenced blocks under small headers (`main`, `This branch`, `Conclusions`) and end with a single sentence stating what it proves. The same evidence discipline applies to quick comments as to full bodies: cite exact figures and link the dashboard or PR that backs the claim.
