# Long-form: essays, docs, and READMEs

Long-form prose published under his name — strategy essays, technical tutorials, structured explainers, substantial documentation. The voice from `SKILL.md` applies; this file adds the structure and rhetorical moves that long form allows but short contexts dial down. These illustrate the principle — consider what fits the piece.

## Contents

- Voice and stance
- Structure
- Headers
- Closings
- Sentence rhythm and rhetorical moves
- Tone and posture
- Diction
- Formatting conventions
- Dialing down for short contexts

## Voice and stance

First-person and anchored in lived experience, not asserted as abstract truth: "over the years, I've noticed", "I've come to believe", "in my experience". Ideas are framed as beliefs arrived at gradually rather than fixed positions. Intellectually honest to the point of pre-empting objections — flag up front that a claim may sound striking, then earn it. Name uncomfortable conclusions as such; that reads as candor, not hedging.

For a team audience the voice shifts to "we" (a piece framed around a shared retrospective). For a general audience it leans on "I" and direct "you".

## Structure

Tightly scaffolded — the map comes before the territory. A short intro frames the problem and states what the piece will cover. Technical posts follow a consistent arc:

1. State the problem.
2. Walk through the naive approach.
3. Explain precisely why it is flawed.
4. Present the correct approach.
5. Show the implementation.
6. Show the result.

Code is almost always followed by a bulleted "here's why" rather than left to speak for itself.

## Headers

Headers do real work and come in two flavors:

- Punchy / evocative for conceptual pieces: "Ship fast, learn slow", "Guesswork dressed up as strategy", "Impactful or tractable?".
- Plainly functional for how-tos: "Steps to Gather Deployment Stats", "Custom matchers".

Pick the flavor by piece type. Do not mix a jokey header into a procedural how-to.

## Closings

Deliberate, never trailing off: a "Summing up", "Conclusion", "Final Result", or "Further Reading" section, often with links to further reading and prior art.

## Sentence rhythm and rhetorical moves

Default sentence is medium-to-long and clause-rich, punctuated by short declaratives for emphasis: "Constraints are hard." "The PRs look good." Signature moves:

- **State-the-belief, then counter it.** Name a common assumption, then push against it. The most frequent device in his argumentative writing.
- **Rhetorical questions** as engagement and transition: "It's cleaner, right?", "What's your constraint?".
- **Rule of three / parallel lists** woven into prose: tests "easier to read, maintain, and evolve".
- **Concrete "imagine" scenarios** to ground abstraction rather than explaining in the abstract.
- **Pull-quote takeaways.** The single sharpest sentence of a section lifted into a bolded blockquote: "Output isn't outcome."

## Tone and posture

Measured, pragmatic, and warm. Hedging is deliberate and accurate — "often", "frequently", "can", "may", "at best… at worst" — which makes the rare unhedged line land harder. Confident without being dogmatic, with explicit room for nuance ("TOC isn't a silver bullet").

In technical posts the warmth becomes mild enthusiasm: "The joy of…", "made onboarding a breeze", "a pleasure to expand". The instinct is pedagogical — explain *why*, not just *what* — leaning on cited authorities (Goldratt, Fowler, Beck, Feathers, Kim) and generous linking. Comfortably pragmatic about good-enough ("for now, it's good enough"). A light playful streak appears occasionally — a throwaway example named "chuchu", a well-placed exclamation.

## Diction

Professional but accessible; jargon is defined rather than assumed. Domain terms are used precisely (throughput, subordination, matchers, deployment statuses) but unpacked. Conceptual keywords are bolded liberally on first introduction — **Theory of Constraints**, **deferred investment**, **Ubiquitous Language**, **cognitive load** — almost as inline glossary markers. Metaphors are vivid but restrained: tech debt as a financial loan, planning that "feels like theater".

## Formatting conventions

Markdown throughout: hierarchical `##`/`###` headers, generous inline links to docs and books, fenced code blocks with language hints, numbered lists for procedures and bulleted lists for categories. Blockquotes serve two roles — bolded pull-quotes for key takeaways, and callout asides ("New to TOC?" boxes) for optional context.

## Dialing down for short contexts

The moves above suit an essay and feel heavy in a PR comment or doc snippet. When the target is shorter, carry over the problem → naive → why-flawed → correct arc, the grounding of claims in reasoning, the "here's why" after code, the precise hedging, and the deliberate close. Dial down the bolded-keyword density, the pull-quote blockquotes, the further-reading sections, and the longer rhetorical build-ups. The voice stays recognizably the same; only the scaffolding compresses.
