<!--
  Kanban task card. Created by the `new-task` skill, or copy this file to
  tasks/todo/<NN>-<short-slug>.md and fill the top half by hand.
  The `kanban` skill dispatches each todo card to a background worktree subagent
  and advances it todo → doing → done. The bottom half (Timeline / Handoff) is
  filled by the agents during execution — leave it blank at creation.
-->

# <Short imperative title, e.g. "Source 11 extracts only one row">

**Type:** bug | feature | chore
**Priority:** high | normal | low
**Scope:** <source id(s) and/or area — e.g. "source 11", "source-list UI", "all XLSX extractors">

## Problem / goal
<What's wrong, or what you want built. Include the "why" if it isn't obvious.>

## Evidence  (bugs — optional for features)
<Observed vs expected. Paste any error text / stack trace. Repro steps. Screenshot path.>

## Pointers  (optional)
<The executing (Sonnet) agent's map. Name concrete files AND entry points within them — path, function/class, why it's relevant — the pattern or sibling code to mirror, related prior work, and constraints that bound the change. Prefer "src/extractors/xlsx.py — extract_rows(), drops empty cells" over "the extractor code".>

## Done when
<Acceptance criteria — how we confirm it's fixed/built.>

---
<!-- Filled during execution — do not edit by hand. -->

## Timeline  (local timestamps `YYYY-MM-DD HH:MM`)
- created:
- dispatched:
- agent finished:
- merged:

## Handoff
<!--
  The task subagent appends a structured handoff here BEFORE finishing. The
  merger reads it to integrate the work and update CHANGELOG — the `Changelog
  entry` line is copied verbatim, so always include it.
-->

**Summary:** <one or two sentences on what changed>

**Changes:** <key files/areas touched and why; branch is kanban/<slug>>

**Decisions:** <non-obvious choices and their rationale; root cause for bugs>

**Notes / follow-ups:** <verification done, admin/manual steps, leftover TODOs>

**Changelog entry:** <single line, as it should appear in CHANGELOG>
