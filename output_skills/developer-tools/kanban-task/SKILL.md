---
name: kanban-task
description: "Turns one or more requests into well-formed kanban cards in tasks/todo/ by dispatching background analyzer subagents that decompose each request, locate the work in the codebase, and write a navigation-brief card. The main agent stays a thin router: it owns the user conversation, relays any clarifying questions an analyzer needs, and never blocks on analysis. Use when adding one or several tasks to the kanban board."
argument-hint: "[task description]"
disable-model-invocation: true
---

STARTER_CHARACTER = 📝

# Kanban task

Turn requests into well-formed kanban cards in `tasks/todo/`, ready for the `kanban` skill to dispatch. The reasoning-heavy part — decomposing a request and locating the work in the codebase — runs on **Opus in a background analyzer subagent**, not inline on the main agent. Analysis is where the strong model earns its keep: a sharp card lets a cheap model execute it. This is what lets a queue of prompts be analyzed in parallel instead of one at a time.

## Your role: router, not analyst

You (the main agent) do almost no analysis here. You:

1. **Dispatch** one analyzer subagent per pending request (see [references/analyzer-subagent.md](references/analyzer-subagent.md) for its prompt and return contract). When several requests are pending, dispatch them **concurrently in a single message** so they analyze in parallel — this is the whole point.
2. **Route returns.** Each analyzer returns one of two verdicts:
   - `card_written` — the analyzer already wrote the card to `tasks/todo/`. Acknowledge it to the user with the path, one-line summary, and any assumptions it recorded. Nothing else to do.
   - `needs_clarification` — the analyzer hit a *blocking* gap (an essential it could not infer) and wrote **nothing** to the board. It returns questions pre-shaped for `AskUserQuestion`. Relay them to the user, collect the answers, then **resume that same analyzer** with `SendMessage` (by its agent id/name) carrying the answers. The analyzer finishes the card with its context intact and returns `card_written`.
3. **Ensure the board is running** once cards exist (see Steps below).

You never explore the repo or write a card yourself. The only user-facing conversation — clarifying questions — stays with you because a background subagent cannot prompt the user.

## Why this split

A background analyzer cannot ask the user a question; its final message returns to you as a tool result, invisible to the user. So the analyzer **returns** its questions instead of asking them, and you do the asking. That keeps the human loop on the main agent while N analyzers churn in the background, and it keeps incomplete cards off the board entirely — a card only ever lands in `todo/` when it is complete and dispatchable.

## Clarification round-trip

For a request with blocking gaps:

1. You dispatch the analyzer. It explores, classifies each gap as **blocking** (an essential — Title, Type, Problem/goal, Done-when — that cannot be inferred) or **non-blocking** (Priority, Base, Scope, Model, Pointers — which it infers and records under `## Assumptions`).
2. On a blocking gap, the analyzer returns `needs_clarification` with a `questions[]` list already shaped as `AskUserQuestion` options, plus `draft_notes` on what it already figured out.
3. You batch those questions into a single `AskUserQuestion` call (≤4 questions). Other analyzers keep running while you wait.
4. You `SendMessage` the analyzer its answers. It resumes — no re-derivation — and writes the card, returning `card_written`.

If the user does not answer, that request simply produces no card until they do. No board pollution, nothing stuck. Surface a brief "awaiting your answer on: <request>" line so the pending question is visible.

## Steps

1. For each pending request, dispatch an analyzer subagent on **Opus** (`model: "opus"`) — see Analyzer model below — passing the request text, a short `## Context` block of any directly relevant recent conversation, and the analyzer prompt from [references/analyzer-subagent.md](references/analyzer-subagent.md). Dispatch concurrent requests in one message.
2. Route each return as above (`card_written` → acknowledge; `needs_clarification` → ask → `SendMessage` resume).
3. Once at least one card exists, ensure the board is running. The `kanban` orchestrator never self-terminates — it idle-polls `tasks/todo/` — so a card dropped there is picked up automatically by a running board. If a kanban loop is already active in this session, just note the card will be absorbed on its next poll. If none is running, start one by following `kanban/SKILL.md` directly (it sets `disable-model-invocation`, so read and run its control loop inline rather than via the Skill tool). Never start a second loop when one is already active — duplicate orchestrators race on the base branches.

## Analyzer model

Analyzers run on **Opus** by default. That is the right model for nearly every request, and it is the one you pick unless something specific says otherwise.

Dispatch on **Fable** (`model: "fable"`) only for a genuinely demanding analysis, and say in your acknowledgement that you did:

- The request spans several subsystems, or its shape has to be designed before it can be described.
- The work is a hard diagnosis — an intermittent or cross-cutting bug with no obvious entry point.
- Getting the decomposition wrong would be expensive: a migration, a schema or protocol change, anything with wide blast radius.
- The user asked for Fable, or asked for a deep or careful analysis.

Never dispatch below Opus. A weak analysis produces a vague card, and a vague card forces an expensive executor — the cost lands downstream either way.

## Model economy

Two model choices sit in this flow, and they pull in opposite directions on purpose.

- **The analyzer** is strong (Opus, occasionally Fable), because it runs once per card and its output determines everything after it.
- **The executor** is cheap by default (`haiku`, stepping up to `sonnet`, and to `opus` only for genuinely hard work). The analyzer sets this in the card's **Model** field.

The trade is deliberate: spend on analysis, save on execution. When you see cards routinely coming back with `model: opus`, that usually means the analysis is under-specifying scope and pointers, not that the work is hard — raise it with the user rather than accepting it as normal.

## Principles

- One card, one unit of work. Split unrelated asks into separate cards (separate analyzers) rather than one sprawling card.
- The card is a *navigation brief*, not a restatement of the request — that quality bar lives in the analyzer prompt; your job is to dispatch and route, not to second-guess its output. A brief good enough to execute is also a brief a cheap executor can execute.
- Keep the user conversation tight: relay only genuine blocking questions, batched, never drip-asked.

## Anti-patterns

- Analyzing the request or writing a card yourself on the main agent — that re-serializes the queue this skill exists to parallelize.
- Letting an analyzer ask the user directly (it cannot) or writing a half-finished card to the board to "hold" a question — questions stay in-context via the return/resume loop.
- Re-dispatching a fresh analyzer to answer a clarification instead of resuming the suspended one with `SendMessage` — that throws away its exploration and repeats the work.
- Starting a second kanban loop when one is already running.
- Dispatching every analyzer on Fable, or dispatching one below Opus — Fable is the demanding-analysis exception, and Opus is the floor.
