# Six Months of Spec Kit — What Lead Maintainer Actually Felt Like

*A personal retrospective. No series number on this one — just me thinking out loud about the last half year.*

Six months ago today, on January 22nd, I took over as lead maintainer of [Spec Kit](https://github.com/github/spec-kit). I did not write anything down about it at the time, which in hindsight is a little funny given that the whole project is about writing things down before you build them. So this is the note I owe past me: what the first six months were actually like, what I am proud of, and what I got wrong.

I want to be honest that this is not a launch post or a numbers-go-up post. There are numbers in here — some of them I am genuinely happy about — but the reason I am writing is that a milestone like this is one of the few moments you give yourself permission to stop and look back, and I do not want to waste it on a highlight reel.

The CODEOWNERS change made my role publicly visible on February 23rd — the repo's global owner line became `* @mnriem` — but by then I had already spent enough time in the plumbing that the line mostly formalized what was already happening. I do not take the trust it represented lightly.

## The first commit was a bug fix

My first commit landed on February 10th. It was [#1588](https://github.com/github/spec-kit/pull/1588), and it was not some grand architectural statement — it fixed a path error, `.specify.specify`, that was breaking scaffolding. I remember being slightly self-conscious about that. You inherit a project with tens of thousands of stars and your opening move is a one-line path fix.

I have come around to thinking that was exactly the right first commit. You do not earn the right to reshape something by declaring intentions; you earn it by fixing the thing that is annoying someone today.

## What we actually shipped

The stretch I am proudest of is March. In close to a dozen releases across that month, we landed the pieces that turned Spec Kit from one fixed way of doing spec-driven development into something composable: **multi-catalog extensions**, **pluggable presets**, and **air-gapped deployment** for the folks who cannot reach out to the public internet. Individually, none of those is a headline. Together they are the difference between a tool you use and a kit you build with.

Underneath the whole six months is a family of primitives that is really what I spent my time on, and they did not arrive all at once — each got its own moment. **Presets** were part of that March work: they let you reshape the reference process — its templates, commands, even its terminology — and there are now more than twenty community presets doing exactly that. The others came later. **Workflows** and their **workflow steps** took shape through late spring into June — a workflow promotes a process from something you perform by hand to something you invoke, with human checkpoints and resume-from-where-it-stopped built in, and the step catalog that lets people share custom step types only landed in June. **Bundles** came later still, over June and into July, wrapping a whole way of working — the extension, the preset, the wiring between them — into a single installable thing so onboarding a teammate is one command instead of a fragile sequence. I wrote a [series](../../06/26/spec_kit_bundles.html) taking each of these apart one at a time; what strikes me looking back is how consistently the answer was "compose, don't accrete" — new capabilities as small pieces that snap together, not features bolted onto a monolith.

One release from early on deserves singling out. On April 2nd came **v0.5.0**, and that one I will defend as one of the trickiest releases of the six months, and a necessary one to move the project forward, even though it is, on paper, a *removal*. We stopped shipping template zip bundles. The CLI now does all the scaffolding itself, which means the CLI and the templates can never drift out of sync again — a class of confusing bug that just stops existing. We also added a proper `DEVELOPMENT.md` so that contributors did not have to reverse-engineer how to work on the thing. Deleting the zip bundles was the kind of change that generates an "Important Notice" in the release notes and a few raised eyebrows, and I still think it was correct. Six months in, the clearest lesson I have is that the best maintenance work is often subtraction.

In truth, a commit count is the wrong unit for this job. Over these six months more than a thousand pull requests were closed — over 900 of them merged, contributed by more than 170 different people — and close to a thousand issues were triaged shut. My own commits are a small slice of that; the rest was the work of contributors, reviewers, and a lot of automation, Copilot among it. It was never a solo act. And when I do look at my own commits, the ones I remember are almost never the big features. They are the docs fixes, the error messages that got clearer, the thing a stranger complained about that is now quietly gone.

## Managing the project, this time around

This is not the first project I have helped run in the open. Over the years I have been involved in a number of open source projects, both as a maintainer and as a contributor, and the biggest thing I took away from all of it is simple: community engagement is what makes a project tick. The code matters, but a project lives or dies on whether people feel welcome to show up, ask questions, disagree, and contribute. I came into Spec Kit already believing that, and the last six months have only reinforced it.

One thing that is genuinely different this time around: Copilot. Reviewing incoming pull requests has become a lot easier than it used to be — not because it makes the decisions for me, but because of how it changes the shape of a review. It is more than a first pass. Copilot goes through several review rounds on a change, catching the mechanical things and the "does this even do what it says" problems, and by the time it reaches me most of that noise is already gone. I still do the final review myself — that part does not get delegated — but I am spending my attention on the judgment calls instead of the plumbing. That is genuinely how I operate now. And it is not just review. Authoring documentation and even delivering code have both gotten easier: the blank page is less blank, the boilerplate writes itself, and the gap between "I know what I want to do" and "it is done" has narrowed. For a project this size, that difference is not a nicety — it is a good part of how the pace stayed up.

## The part I did not expect

I expected the code. I did not expect how much of the job is *people*.

Opening the system up is what grew the community. Once extensions, presets, integrations, and the other primitives existed, people finally had somewhere to put their own ideas — and they did. We now support more than 30 coding agents, and the community catalog has passed 140 extensions — actual things actual people built and shared, not demos. The primitives all matter, and the first one to land — extensions — came in as a [community pull request](https://github.com/github/spec-kit/pull/1551) from mbachorik. The rest of the primitives followed, and people were filling the room at the same time they were being built — watching that happen is a strange feeling, because every one of those is someone deciding the primitives were worth their evening.

What people built with those primitives is the part I keep coming back to. Some of it reshapes the process itself — brownfield extensions like *Brownfield Bootstrap* and *BrownKit* that point Spec Kit at an existing codebase and reverse-engineer specs out of it, or *Architecture Workflow* generating 4+1 architecture views. Some of it wires Spec Kit into the tools teams already live in — an *Azure DevOps* integration that syncs stories and tasks to work items, governance extensions that emit repository policy files, analytics that measure how much time the AI actually saved. Some of it is opinionated whole processes — *AIDE*, a structured seven-step build-from-scratch workflow; bugfix and canon-driven workflows; branch-naming conventions. None of these came from the core team. They came from people solving their own problem and then bothering to package it for everyone else. The best way I can describe the shift is that Spec Kit stopped being a thing you *use* and became a thing you *extend*.

On May 8th I got to talk about all of it on GitHub's **Open Source Friday** livestream, which was equal parts thrilling and terrifying; explaining your design decisions live, to people who will immediately go try to break them, is a very good forcing function for humility.

The stars went from around 69,000 when I started to over 123,000 by this milestone. I have deliberately put that number near the end, not because it does not matter — it is a real sign that people find the project worth their attention, and that genuinely means a lot to me — but because it is not what the work feels like day to day. Each star is the easiest form of feedback someone can leave — a quiet way of saying they liked what they found. They are a wonderful thing to see climb. They are not, on their own, the job.

## What I got wrong

I moved too fast on a couple of deprecations without enough warning, and a few people felt it. The zip-bundle removal was right but I could have communicated it earlier and more loudly.

I was also not diligent enough about delivering documentation alongside the code. Documentation is not the slow part anymore — but for most users the docs *are* the feature, and when they lagged behind a release the feature may as well not have shipped yet. What we should be doing, and what I am carrying into the next six months, is treating writing not as the tax on shipping but as part of shipping.

## The next six months

I do not have a roadmap I want to recite here, partly because the honest answer is that the community keeps telling me what matters faster than I could plan it. One thing we will tackle for sure is polishing and hardening what is already there.

Mostly, though, I just wanted to mark the date. Six months ago I fixed a path bug. Today the project is bigger, stranger, and more other-people's than it was, and I am still enjoying it. That last part is not nothing.

If you have used Spec Kit in these six months — thank you, genuinely. If something about it annoyed you, that is the useful kind of feedback; send an email to blog (at) manorrock.com and tell me.

---

*Further reading: the [Spec Kit repository](https://github.com/github/spec-kit), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), and the [full series](../../06/26/spec_kit_bundles.html) on the Spec Kit primitives.*
