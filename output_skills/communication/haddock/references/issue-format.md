# Issue format

## Contents

- Shape of an issue
- The lede
- An item, part by part
- Worked item
- The roundup
- The sign-off
- Citations and links
- Length and cutting

## Shape of an issue

```markdown
# <Issue title: the take of the strongest item, not a summary of the week>

<Lede: two or three sentences.>

## <Item headline>
## <Item headline>
## <Item headline>

## Also on the water
<Roundup of linked stories with one line each.>

---
<Sign-off.>
```

Three to five items. Each has a take or it does not appear.

## The lede

Two or three sentences, Haddock's voice, no scene setting. It names the thread running through the issue, or it names the single thing that mattered most and says the rest is smaller. It does not preview each item in turn.

It never explains what the newsletter is, what Kanikosen is, or why the reader should care. A reader who is here already knows, and a reader who is not is convinced by the items.

## An item, part by part

**Headline.** The take compressed. Not the article's headline, and not a neutral summary.

**The incision.** One line, Haddock. What the story glossed over, blurred, or under-read. This is the first line of the body and it lands before any attribution.

**The story.** What was reported, by whom, when, in two or three sentences. Outlet and date cited, with a link to the specific article. Enough that the item stands alone without the reader clicking through.

**The name collision, where there is one.** If more than one hull currently carries a name used in the item, enumerate them and say which is the subject. Give each collision enough to be told apart: build year, builder country, flag, type. A collision is never resolved silently, and it is frequently the strongest thing in the item, because it is usually what another outlet got wrong.

**The mechanism.** Why-first, guille. What physically or procedurally produced the situation: the reflagging, the gear the hull can actually shoot, the authorisation that does not fit, the ownership step. This is where seamanship earns the item.

**The Kanikosen delta.** What the registers hold that the article did not report. Presented as finding: names, flags, dated register entries, owners, and which sources disagree. Identifiers backticked. This is the part that sells, and it sells by being specific.

Specific means public: `IMO`, `MMSI`, call sign, Flag, type, tonnage, build date, registered owner, and the register's own dated entries. It does not mean licence numbers, regional register IDs, capture timestamps, source counts, or corpus totals, which are working material and stay in the notes. Full rule: "What the newsletter may publish" in [voice.md](voice.md).

**The honest limit.** One line naming what is not known and what would settle it. Never omitted. It is the cheapest credibility available and the thing that distinguishes this from marketing.

## Worked item

This illustrates the shape and the voice. Consider what fits the story in front of you; do not transplant the structure sentence for sentence. The corpus figures below are real, drawn from `IMO 8908117` at the time of writing, and would be re-queried before publication.

> ## The trawler that has been six ships
>
> One hull, one `IMO`, and a name history long enough that one registry gave up and packed it into a single field.
>
> The Maritime Executive reported on 24 April 2023 that a Russian trawler operating as ESTER had been identified in connection with activity off the Norwegian coast. The coverage treats it as one vessel with one name.
>
> It is one vessel. It is not one name, and it has not been one flag. This is a 120 m Atlantik-type freezer supertrawler built in 1991, a class with the endurance and freezing capacity to work far from its flag state for months, which is what makes a name and flag history like this operationally worth having.
>
> `IMO 8908117` is carried under three current names by the registers that publish it. IMO GISIS and Lloyd's Register hold it as GERMCS. NEAFC's vessel register holds it as ESTER. A Russian national registry renders its alias history as one string: `RYBAK ODESSY RYBAK 1(C 92) ARMOR HELIOS FENIKS SOUZ(S 06)`. The Flag reads PER in IMO GISIS and in the SPRFMO authorised list, RUS in the NEAFC register, and BLZ in Lloyd's. Four call signs are on file, including `UBHP` and `V3OP8`. The registered Owner chain runs through QINGDAO DEEP-SEA FISHERIES CO and APEX LUCK BUSINESS LIMITED, the latter recorded in the British Virgin Islands, with APEX LUCK also listed as Operator.
>
> What I cannot tell you is which of those flags is current today. The registers disagree, and each is right as at the date of its own entry, which is not the same date for any two of them. The Flag State's own deletion record would settle it.

Note what the item does not do. It does not call the vessel illegal. It does not infer beneficial ownership from a British Virgin Islands address. It does not reconcile the three flags into one. It states the disagreement, says which question it cannot answer, and names the document that would answer it.

Note also what it prints and what it does not. Every value in the delta is one a register publishes. There is no source count, no capture timestamp, no internal identifier, and no sentence that assumes the reader can look anything up. This hull has no live name collision, so the item has no collision paragraph; where one exists it goes immediately after the story, before the mechanism.

## The roundup

Stories worth a reader's attention that produced no take. One line each: what happened, who reported it, and the link. No analysis. An item demoted here for lack of a delta is doing its job; padding it back up into a full item is how an issue rots.

## The sign-off

One or two lines. Haddock, dry, brief. It may point at what he is looking at next. It does not solicit, thank, or ask for shares.

## Citations and links

Every outlet citation carries a link to the specific article. Not the outlet's front page, not a section index, not a search result.

A URL is never constructed, guessed, or inferred from a pattern that other articles on that site happen to follow. Either the link was seen resolving to the cited content or there is no link.

A citation that cannot be linked runs as plain text and is listed in the notes as a gap. A link that does not land on the cited content, such as a bulletin that only exists behind a signup form, is published with the claim flagged in the notes as not independently checkable by the reader.

## Length and cutting

An issue runs as long as the takes justify and no longer. When cutting, the order is: pad first, then any item whose delta is thin, then the roundup, then the honest-limit lines, which are never cut.

**Decide the length in items before drafting, never in words.** A word target is converted to an item count first, because compressing afterwards eats the mechanism paragraph, and the mechanism and the honest limit are the two things that must not go. Budget:

| Part | Words |
| --- | --- |
| Lede | 45 |
| Item | 200 to 300, by how much record it carries |
| Roundup line | 30 |
| Sign-off | 30 |

An 800 word issue is two items and a short roundup. Three items with real deltas land between 1,100 and 1,250. Decide which of those is being written before the first sentence.

The floor for an item is about 200 words. Below that, an item carrying an incision, a story, a mechanism, a record, and an honest limit has to drop one of them, and the two it drops first are the two that matter. An item that will not fit above the floor is demoted to the roundup, not squeezed.

This is not licence to pad. The budget decides how many items an issue has; it never decides how long an item is once written. An item that says what it has in 210 words is finished at 210.

An item that cannot survive the verification step is cut entirely rather than hedged into safety. A hedged item nobody can act on costs more credibility than a missing one.
