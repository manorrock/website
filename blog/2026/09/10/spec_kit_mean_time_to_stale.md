# Spec Kit's Mean Time to Stale — How Ken and the Data Talked Me Out of My Own Policy

*A note from the maintainer's chair about a graceful default, a contributor who told me it had gotten too slow, and the mean that proved him right.*

Every maintainer picks a default before they know what they are defaulting for. Mine, on [Spec Kit](https://github.com/github/spec-kit), was the stale workflow: **150 days of inactivity before an item is marked stale, then 30 more before it closes.** 150/180. I set it wide on purpose. When you are new to a project and you have no idea what the community will send you, grace is the right bias — a generous window is how you avoid closing someone's real work simply because you were too new to recognize it. For a long stretch, that was exactly the right call.

This post is about the moment it stopped being the right call. It stopped the way these things usually do — a person pushed on it, the data pushed harder, and the project itself had quietly changed underneath the number while I wasn't looking.

## First, someone told me it was too slow

The person was Ken. The message was simple: the stale process is too slow. Five and six months of silence before an item even gets a warning means the backlog carries dead weight for half a year, and by the time the workflow acts, everyone has long since stopped caring. Speed it up.

My first reaction was the one I suspect most maintainers have to "make the timeout more aggressive": *no.* 150/180 is safe. Shortening it risks closing real work. That instinct was not vanity — it was the same protective grace that set 150/180 in the first place. But grace you never re-examine quietly hardens into inertia, and "I like the number I already have" is not an argument.

So instead of defending it, I did the only thing that actually settles this kind of disagreement: I went and measured.

## Then I asked the data what "too slow" actually meant

The question I put to the analysis was deliberately narrow, because vague questions produce vague policy. Not "is 150/180 too slow" in the abstract, but: *if I tighten the window, what specifically gets swept up, and does the real work finish inside the tighter window anyway?*

I looked at two candidate policies against the live upstream backlog — 140 open issues and 164 open PRs on the day I ran it:

| Policy: stale / close | Newly eligible issues | Newly eligible PRs | Total |
|---|---:|---:|---:|
| 90 / 120 days | 41 | 39 | 80 |
| 60 / 90 days | 52 | 49 | **101** |

That was the *exposure* side — who gets touched. But the number that actually mattered was on the other side of the ledger: **how long does work that genuinely gets done take to get done?** If real fixes routinely take four months, then a 60-day window is a guillotine. If they finish in days, it is a broom.

So I measured the mean time to completion across Spec Kit's full history:

| Population | Full mean |
|---|---:|
| Merged PRs | **3.41 days** |
| Completed issues closed through a merged PR or commit | **15.16 days** |
| All issues marked "completed" | **25.56 days** |

And there it was. The **worst** mean in the whole table — every completed issue, long tail and all — was **25.56 days.** Comfortably less than half of a 60-day warning threshold. The typical piece of work that actually ships on Spec Kit is done, merged, and closed weeks before the *fastest* stale policy I was considering would so much as post a warning.

## The distinction I had been missing

Staring at 25.56 days sitting under a 60-day threshold forced me to name the thing I had been quietly getting wrong. I had been treating the stale window as if it were a deadline on *total age* — as though a 60-day policy said "you have 60 days to finish or we close you." It does not. The stale clock measures **consecutive inactivity**, not lifetime. An issue can take six months of back-and-forth to resolve and never once go stale, as long as it keeps getting activity along the way.

That reframes the whole risk. The question was never "does real work take longer than 60 days?" — sometimes it does. The question is "does real work sit *untouched* for 60 straight days and then still turn out to matter?" And that is a much rarer, much more specific event. The long tail exists — 24 of 513 code-linked completed issues took more than 90 days — but total lifetime is not inactivity, and a `pinned` label protects the genuine slow-burn work that deserves to survive a quiet stretch.

The data did not just answer Ken's question. It dissolved the objection I had walked in with.

## So I changed the number

I had come into this recommending 90/120 — the cautious compromise. The evidence did not support the caution. It supported him. **60/90, keep the 30-day warning, use `pinned` for accepted work that should survive extended silence.** I revised the recommendation on the record, from the conservative position I started with to the more aggressive one the numbers actually justified.

And here is the part I had missed about my own project. 150/180 was graceful for a repo I did not yet understand — but Spec Kit is not that repo anymore. It ships a release every other day or so now. It is a high-touch flow, with work moving fast enough that a genuinely live issue almost never sits untouched for two months, and a genuinely dead one has no business lingering for six. 60/90 is not me abandoning the grace I started with. It is that grace recalibrated to the cadence the project actually runs at today.

That recalibration was still uncomfortable in the specific way changing your mind in public always is. But the discomfort was not "I was wrong." It was "the thing that was right has an expiry date, and I nearly missed it."

## Three things this cost me, and why I'd pay again

I have ended nearly every Spec Kit post with what I appreciate about a design. This time the design is a decision-making one, and what I appreciate about it is that it made three uncomfortable things unavoidable:

**You have to actually listen.** Ken's message was easy to wave off — it contradicted a setting I was fond of, and I had a ready-made reason why he was wrong. Listening does not mean nodding; it means letting someone else's push be the reason you go and check. The maintainer who treats every "this is too slow" as an attack on their judgment will optimize for their own comfort and call it stewardship.

**You have to be data-driven, especially against yourself.** My instinct said "60/90 is reckless." The data said the worst completion mean was 25.56 days and the whole fear was mostly a category error about age versus inactivity. When your gut and your measurements disagree, the measurements are the ones that don't have an ego to protect. The numbers were not there to win the argument for Ken; they were there so that *neither* of us was just asserting.

**You have to be willing to change.** Listening and measuring are worthless if the conclusion was decided in advance. The entire exercise only meant something because I let it move me off 150/180 — past the 90/120 I would have preferred — to a policy I had spent the first ten minutes arguing against. And the deeper reason it moved me is worth saying plainly: a good default has a shelf life. 150/180 was the correct, generous call for a maintainer who did not yet know his community. A release-every-other-day project run at that pace is a different animal, and pretending my early defaults were timeless would have been its own kind of negligence.

None of this is unique to a stale timeout. The stale window just happens to be a small, safe place to practice the thing maintaining is actually made of: setting a generous default when you are new, then holding it loosely enough that a good contributor, an honest number, and a changed cadence can pry it back open. Spec Kit exists to move intent out of one person's head and into artifacts everyone can argue with. It would be a strange project to run while treating my own config values as beyond argument.

Ken was right. The mean proved it. And 150/180 was never a mistake — it was the grace a newer maintainer owed a community he did not yet know. Keeping it once both the data and the cadence had moved on: *that* would have been the mistake.

If you maintain something and you have a setting you have stopped seeing — or you think 60/90 is far too aggressive and you want to tell me why — send an email to blog (at) manorrock.com. Bring numbers.

---

*Further reading: the [Spec Kit repository](https://github.com/github/spec-kit) and its [Community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md), [Spec Kit Goes AI Source — When Your Contributors Are Agents](../../09/08/spec_kit_ai_source.html), [A Year of Spec Kit — Notes from Washington](../../08/31/spec_kit_one_year_in_washington.html), and [Spec Kit, Part 9: Workflows — KDD in Motion](../../07/09/spec_kit_workflows.html).*
