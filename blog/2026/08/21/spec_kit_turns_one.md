# Spec Kit Turns One — and Ships 1.0.0

*A birthday post and a release post, wearing the same hat. No series number on this one — some milestones don't need a chapter, they need a candle.*

One year ago today, on August 21st, 2025, the very first commit landed in [Spec Kit](https://github.com/github/spec-kit). I went and looked it up, because a project about writing things down before you build them ought to at least know its own birthday. There it was — "Initial commit," authored by [Den Delimarsky](https://github.com/localden), who together with [John Lam](https://github.com/jflam) dreamed the thing up and gave it its first shape. Both have since moved on to other adventures — like a first love from high school, they are off writing other stories now — but the project carries their fingerprints in its bones, and it would be graceless to celebrate a first birthday without saying so. Den and John lit the candle; a great many of us have been keeping it burning. Twelve months later the repo has more stars than I can quite believe, more than 150 community extensions, more than 30 community presets, support for more than 35 coding agents, and — as of today — a version number that starts with a `1`.

So this post is doing two jobs at once. It is a birthday note: happy first birthday, Spec Kit. And it is a release announcement: **1.0.0 is out.** I could have written two posts. But the two things are the same thing wearing different clothes, and pretending otherwise would have been more ceremony than either deserves.

## What actually shipped

Let me get the concrete part out of the way first, because "1.0.0" is a claim and claims should come with receipts.

1.0.0 is not a pile of new features. It is the opposite. It is the release where the five primitives the [series](../../06/26/spec_kit_bundles.html) spent twelve posts pulling apart — extensions, presets, integrations, workflows, and steps — plus the bundles that compose them and the catalogs that distribute them, finally feel like they were designed by one hand for one journey rather than accreted one clever idea at a time. The [Road Ahead](../../07/14/spec_kit_road_ahead.html) post called this out as the work that was not a feature: coherence. This release is a down payment on it. Seams smoothed, terminology reconciled, the sharp edges between layers filed down until installing a bundle, reshaping it with a preset, and driving it through a workflow feels like one tool instead of three that agreed to cooperate.

And here is where, a year ago, I would have written the sentence every 1.0.0 post writes — *the shape holds still now, breaking changes cost a major bump, we promise to warn you before we move anything.* I am not going to write it, because I no longer think it is true or that it matters. That is worth a section of its own.

The full changelog is in the [GitHub Releases](https://github.com/github/spec-kit/releases), and I would rather you read the receipts there than take my summary on faith.

## The promise a 1.0.0 used to make — and why it stopped mattering

For decades a version number was load-bearing. The `1` in the major slot was insurance, and what it insured against was **the cost of change**. A breaking change meant a human opening every call site by hand, working through a migration guide line by line, praying the tests caught what the eyes missed. That was expensive, so semver turned the cost into a signal: a bumped major meant *budget a bad afternoon — or worse*, a frozen major meant *build on this without fear*. The promise mattered because adapting was the hard part.

That is the assumption that quietly stopped being true. Adjusting to a breaking change is not a bad afternoon anymore — it is a sentence. You point an agent at the diff — *the catalog format changed these three fields, update every extension in this repo* — and it fixes the call sites in less time than it took me to write this paragraph. Refactoring, the thing we scheduled around and dreaded, got cheap. The migration guide is no longer something *you* consume; it is something your agent consumes for you.

So the insurance a 1.0.0 sells is insurance against a cost that mostly collapsed. That does not make the number dishonest — it just makes it **a number.** The old 1.0.0 was a handshake: *rely on the shape, I will warn you before I change it.* Ours is more of a wave: *here is a good place to start; if I move something, your agent will follow it faster than a changelog could describe it.* The value moved from stability to adaptability, and adaptability is something you already have.

## So what does ours actually mean?

Two things it does not mean. It does not mean **frozen**: Spec Kit's README still files its ambitions under a heading it has never renamed — **"Experimental Goals"** — and I did not rename it for this release. We shipped 1.0.0 of an experiment, on purpose, and we will keep breaking it when breaking it makes it better, because breaking it is cheap now and improving it is worth more than freezing it. And it does not mean **finished**: the brownfield on-ramp could be smoother, the trust boundary is still guidance rather than enforcement, the non-code communities are still officially delightful accidents rather than first-class citizens. A 1.0.0 was never a maturity certificate, and this one does not pretend to be.

What it does mean is smaller and truer. A year in, the kit is coherent — the five primitives finally feel like one tool designed by one hand — and coherent-enough is exactly the thing a round number is *allowed* to mark in an era where it can no longer honestly mark permanence. That is the whole reason the number moved: not because we finished, not because I froze the surface, but because the kit earned a round number and the era lets a round number just be a round number.

## Why I like this milestone

I have ended nearly every post in the Spec Kit series with what I appreciate about the design, so it would be strange to break the habit on its birthday. What I appreciate here is that the release kept faith with the same instinct as everything else: **compose, don't accrete.** A 1.0.0 could have been an excuse to cram in a dozen new primitives to justify the digit. Instead the most on-brand thing the project could do was ship *fewer* rough edges, not more features — and refuse to dress the number up as something it is no longer honest enough to be.

So: happy birthday, Spec Kit. One year old, more than 130 thousand stars, a community that writes the roadmap faster than I could, and a version number that finally starts with a `1` and means exactly, humbly, as much as a version number is allowed to mean now — which is less than it used to, and that is a good thing.

If you have used Spec Kit in this first year — thank you, genuinely. If you think I have the whole "a version number is just a number now" thesis wrong, that is exactly the argument I enjoy; send an email to blog (at) manorrock.com and have it out with me.

Now if you'll excuse me, there is a candle to blow out and a `1` to get used to typing — even if it no longer means what the `1`s before it did.

---

*Further reading: the [Spec Kit repository](https://github.com/github/spec-kit) and its changelog, [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Six Months of Spec Kit](../../07/22/six_months_leading_spec_kit.html), [Part 12: The Road Ahead](../../07/14/spec_kit_road_ahead.html), and the [full series](../../06/26/spec_kit_bundles.html) on the Spec Kit primitives.*
