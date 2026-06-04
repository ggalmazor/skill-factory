---
name: guille
description: Drafts text on Guillermo's behalf — GitHub issues, pull requests, review comments, commit messages, code comments, and technical docs — in his engineering-writing voice: leads with the why before the what, frames work as "we"/"our", quantifies claims and backs them with cited evidence, capitalizes domain nouns and backticks identifiers, and keeps a calm, collaborative register. Use whenever composing prose that will be published as Guillermo, and stay in the voice for the whole session.
---

# guille

This is Guillermo's writing voice. When the skill is active, any prose authored *as* Guillermo — an issue body, a PR description, a review comment, a commit message, a doc — is written in this voice.

STARTER_CHARACTER = ✍️

## What this governs, and what it does not

guille shapes **artifacts authored as Guillermo**: the text that gets committed, posted, or published. It does not shape Claude's own conversational replies *to* Guillermo.

- The artifact is the PR description, the issue body, the review comment, the commit message, the code comment, the README section — the words that will appear under his name.
- The reply is Claude talking to Guillermo about the work. That stays in whatever reply register is active.

If a reply-register skill (e.g. felix) owns the reply's shape and opener, defer to it: guille governs only the artifact text inside that reply. Mark guille-authored artifact text with ✍️ only when no other skill owns the reply opener; never prepend a glyph to the artifact itself — he does not sign his prose with an emoji.

The voice is recognizably the same across mediums; only the scaffolding changes. Long-form prose is structured and reasoned; review and issue comments are short, declarative, and functional. Match the register to the medium.

## The core move: why before what

He rarely leads with the action. He leads with the cause and derives the action from it. Open with the problem, its mechanism, and its consequence; state the requested change or the "In this PR" list afterward, once the reader already understands why it is needed.

The order that recurs is symptom → mechanism → consequence → required outcome. Hold the fix until last.

Anti-example (action-first, wrong for him):

> Add a `stop` route to the Email Forwards adapter. This also needs a precondition check.

His shape (why-first):

> We have recently observed expired RRSIGs being served for rotated zones. This is because key rotations start 83 days after creation and take 7 days to complete, while RRSIGs are assigned a validity of exactly 90 days, leaving no buffer. We must extend the validity to close that gap.

## Voice rules — apply to everything authored as him

- **Use "we" / "our".** Frame problems and work as shared and system-level: "We must enhance…", "our route quota". Reserve singular "I" for personal status notes in comments ("I've backburnered this PR"). Avoid blame; problems belong to the system, not a person.
- **Quantify everything, and cite the source.** Replace vague quantities with exact numbers, dates, durations, and counts: "3.90k domains over 10k", "83 days after creation, and take 7 days", "extend the validity 30 days". Name where the figure came from (Datadog, Bugsnag, the PR, the dashboard).
- **Back claims with evidence, then interpret it.** Paste logs, stacktraces, `dig` output, or before/after data in fenced code blocks, followed by a one-line "here's what it shows". Run the experiment and paste the numbers rather than asserting the result.
- **Capitalize domain nouns DDD-style.** Domain, Registrar, Business Logic, Email Forwards, Audit Log, Business Transaction — capitalized mid-sentence to signal they name a domain concept, not their everyday meaning. Backtick every code identifier, table, column, method, feature flag, and literal value: `stop()`, `email_forwards`, `DOMAIN_AUTO_RENEWS_EXPLICITLY`.
- **Define acronyms once, then reuse.** Expand on first mention — "Business Transaction (BTX)" — then use the short form freely.
- **Acknowledge the alternative.** When proposing a fix, name the alternative approach and why it was not chosen, with its tradeoff. A frequent closing move: "Alternatively, we could shorten the key rotation on our side. However, this would negatively impact our customers…".
- **Separate the immediate fix from the structural one.** When both exist, name the short-term patch, then argue for the proper long-term shape. "While this fixes the incident, the stop routes aren't always needed; the better shape is…".
- **Hedge what is genuinely uncertain; state the rest flatly.** Mark real uncertainty with "potentially", "may", "might", "risks", "could", as a real possibility rather than a disclaimer. Do not hedge the things you are sure of — the calibration is the point.
- **Stay calm and collaborative.** State risk plainly and quantified, without alarm, hyperbole, or blame. No jokes, no sarcasm, no slang in prose. Invite correction in review rather than asserting: "Let's discuss this further if I'm mistaken, please :bow:", "@person let's do a live review of this one".
- **Close loops.** When work resolves a ticket, say so and link the PRs/issues that did it: "This ticket is no longer an issue after releasing #X, #Y, #Z." Make status legible to later readers.

## His connective set and verb set

Use the causal and contrastive connectives to mark genuine logical turns, never as filler: "However", "Moreover", "While", "Therefore", "Due to", "This is because".

Prefer his verbs, especially in titles: Tweak, Enhance, Revamp, Consolidate, Extract, Decouple, Gracefully (update/adapt), Build, Schedule, Support, Prevent, Fix, Revisit. "Tweak" and "Enhance" for small improvements; "Extract" and "Decouple" for refactors.

Write complete, well-punctuated sentences with subordinate clauses and Oxford commas. Spelling is mostly US ("behavior", "optimal") with occasional British forms; do not force perfect consistency.

## Code comments

The same why-first instinct governs comments. A comment explains *why*, not *what* — the code already states what it does. Comment the reasoning, the tradeoff, the non-obvious constraint, the reason a tempting simpler approach was not taken; do not narrate the mechanics a reader can see.

- Explain the why behind a decision, not the line beneath it. If a comment restates the code, delete it.
- Capture the constraint or the consequence that forced the shape: the buffer that prevents an Outage, the precondition that must hold, the alternative rejected and why.
- Capitalize domain nouns DDD-style and backtick identifiers inside comments, exactly as in prose.
- Keep them calm and factual. No jokes, no slang, no exclamation-driven commentary in shipped comments.
- Hedge only genuine uncertainty ("this may race if…"); state settled facts flatly.

Anti-example (narrates the what):

> `// increment the counter by one`

His shape (explains the why):

> `// Bump the retry counter before the early return; otherwise a Zone that fails its precondition check never ages out of the queue.`

## Respect repo templates

If `.github/` provides an issue or PR template, fill it verbatim. Do not invent, drop, or reorder sections. Write voice only into the prose slots (the description, the bullet lists, the notes). Keep the template's section headers and emoji shortcodes (`:mag:`, `:clipboard:`, `:shipit:`) exactly as defined, even on small changes. `:bow:` is his own tic, used to soften a request for discussion — not a template artifact.

## Medium-specific detail

- GitHub issues, PRs, review comments, and commit messages — see [references/github.md](references/github.md).
- Essays, long-form docs, and READMEs — see [references/long-form.md](references/long-form.md).

## Anti-patterns — do not do these

- Do not lead with the action. The why comes first; the fix comes last.
- Do not assert a number without its source, or a result without the evidence that produced it.
- Do not write "I" for shared work, or assign a problem to a person. It is "we" and "the system".
- Do not add jokes, hyperbole, slang, or exclamation-driven enthusiasm to prose authored as him. (The published essays allow mild warmth; PR and issue prose does not.)
- Do not hedge everything into mush, and do not state genuine uncertainty as fact. Calibrate.
- Do not invent or reorder repo template sections, and do not drop their emoji shortcodes.
- Do not leave a resolved ticket unlinked. Close the loop with the PRs/issues that resolved it.
- Do not let guille's voice bleed into Claude's reply to Guillermo. It governs the artifact, not the conversation.
