# Spec Kit Goes AI Source — When Your Contributors Are Agents

*A note from the maintainer's chair about what happens when the pull requests start writing themselves — and why I now run a triage session every single day.*

For twenty-odd years I have known what "open source" meant on the receiving end. Someone finds a bug, or wants a feature, or reads the code on a slow afternoon and decides they can make it a little better. They open a pull request. There is a person on the other side of it — a person with a reason, a repro, a story about how they hit the thing.

That is not the shape of my inbox anymore. And I mean that literally: I watch [Spec Kit](https://github.com/github/spec-kit) with every notification turned on, so each activity on the repository — every pull request, issue, review, and comment — arrives as its own email. My inbox is a live readout of the project's pulse, one message per event. Spec Kit is a project *about* disciplined AI-assisted development, and it has become one of the loudest examples of the thing it was built to organize: the source is increasingly written by agents. Open source is quietly turning into **AI source**, and it is my front-row seat to what it feels like when the pull requests arrive faster than any human could have typed them.

This post is about that shift, and about the answer I have landed on so far. I did not reason my way to it in the abstract. I reasoned my way to it in a triage session I now open every day — a running conversation with an agent that pulls the live state of the repo, scores it, and hands me back a picture of who is asking for what and where my attention will do the most good. The reasoning below is that session's reasoning, cleaned up for public consumption.

## The onslaught is real, and it is polite

Let me put numbers on it, because "onslaught" is a feeling and feelings make bad policy.

On the day I sat down to think about this properly, the repository had roughly 149 open issues and 172 open pull requests — 321 open items in total. **155 of those 172 PRs had no triage label at all.** Five different authors were sitting over any reasonable per-person cap, and one of them alone had **37 open pull requests**, most of them near-identical. Around 33 of the PRs were the same species: speculative micro-hardening. Wrap this call in a `try/except`. Swallow that theoretical `OSError`. Guard against a crash mid-write that, as far as anyone can tell, nobody has ever actually hit.

None of it is malicious. That is the disorienting part. Every one of these PRs is *polite*. The diffs are small. The commit messages are tidy. The reasoning in the description is often genuinely correct — the Python footgun is real, the race is real, the edge case exists. An agent can look at a file and produce a plausible "this could theoretically fail" fix in seconds, and then produce thirty more before you have finished reading the first.

The old cost model of open source assumed writing the code was the expensive part, so a well-formed PR was itself a costly signal of intent. That assumption is gone. Generating a plausible change is now nearly free. Which means the well-formed PR has stopped being a signal of anything at all.

## The scarce thing is no longer code — it is evidence

Once I said that sentence out loud in the triage session, the whole policy fell out of it.

If code is cheap and plausibility is cheap, then the thing that is *actually* scarce — the thing an author has to spend something real to produce — is **evidence**. Not "this could fail." A reproduction. A test that fails on `main` and passes with the change. A linked report from someone who hit it. A disclosure of how the change was produced and a sign that the author understands what they submitted, rather than having relayed it from a model without reading it.

So Spec Kit's answer to AI source is not a wall. It is a gate — an **evidence gate** — and it applies the same way to a human and to an agent, which is exactly the point. I do not care whether a contribution was typed or generated. I care whether it comes with proof.

Concretely, every open item gets scored against a small rubric — seven dimensions, zero to two points each, fourteen possible in total:

- **D1 — Real-world evidence.** Is there a repro or a report, or is this reasoned-but-never-observed?
- **D2 — Reachability and severity.** Does the default path hit this, or does it need a locked file on Windows, or a crash mid-write, to fire at all?
- **D3 — Scope discipline.** One concern, or a sprawling batch diff?
- **D4 — Test evidence.** Is there a regression test that *fails on `main`* and passes with the change?
- **D5 — Disclosure and understanding.** Was AI use disclosed, and does the author show they understand their own patch?
- **D6 — Cost vs benefit.** Does the durable value justify the review cost — or is this a slight defensive win that only adds to the flood?
- **D7 — Roadmap alignment.** Does this fix a filed issue or a real gap, or is it unsolicited hardening the project merely tolerates?

A speculative-hardening PR with no repro and no failing-on-`main` test scores low on D1, D2, and D4 almost by construction. That is not a reason to slam it shut. It is a reason to tell the author, precisely, what would move it forward.

## Four buckets, and why "can-wait" is not "no"

The rubric feeds four verdicts, and getting the verdict *language* right mattered more than I expected.

- **Must-have** (`triage-must-have`) — high-value, important work for Spec Kit; do it first. About 10 items.
- **Nice-to-have** (`triage-nice-to-have`) — an evidence-backed fix or a greenlit feature; land it after review. About 28.
- **Can-wait** (`triage-can-wait`) — valid and in-scope, but deprioritized and held behind the evidence gate. The big bucket: around 275.
- **Out-of-scope** (`triage-out-of-scope`) — won't land in core: invalid, duplicate, off-mission, or redirected to an extension. Only about 8.

My first instinct, and I suspect it is a common one, was to reach for "out-of-scope" as a synonym for "make it stop." The triage session pushed back on me, correctly. Almost none of the flood is out of scope. A speculative fix for a real footgun is a *valid contribution that has not cleared the gate yet* — that is precisely what **can-wait** means. Reserving "out-of-scope" for the genuinely off-mission — the invalid, the duplicate, or the work better redirected to an extension than merged into core — keeps the maintainer honest and keeps a well-meaning contributor from being told their work was garbage when it was merely unproven.

That distinction — deprioritized versus rejected — is the whole ethic of doing this at scale without becoming cruel.

## Labels that talk to the author, capped so they don't shout

Two axes of labels came out of this. Verdict labels — `triage-must-have`, `triage-nice-to-have`, `triage-can-wait`, `triage-out-of-scope` — say where an item stands. And `author-*` labels say what the author needs to *do*: `author-needs-tests`, `author-needs-disclosure`, `author-needs-info`, `author-over-cap`, and `author-awaiting`, which marks that the ball is in their court and hands off to the stale workflow that was already running. No new timer. The gate **composes** with machinery the project already had rather than accreting a parallel copy of it — the same design instinct I have praised through the entire Spec Kit series.

But the sharpest lesson was about restraint, and it came out of the daily session itself. Each author-facing label travels with an **ask** — a short, templated comment telling that one contributor exactly what their item needs to clear the gate. The label is the state; the ask is the request. That same day, as the protocol was taking shape, I put the obvious question to the agent — *if we applied this to everything open right now, what would it do?* — and the answer was sobering: ~300 label edits and ~92 asks in a single run. That is not triage. That is a bot carpet-bombing your contributors, hitting rate limits, and reading as exactly the low-signal noise the gate exists to stop.

So the protocol grew two ceilings. **At most 3 author-facing changes per author per day** — which also, neatly, became a proposed contribution norm: no one should have 37 open PRs demanding review, so cap the *asks* the way you would cap the *submissions*. And **at most 100 items touched per day** to cap the raw volume. There is a chicken-and-egg here worth being honest about: you cannot order the queue by verdict until the items *have* a verdict, and producing that verdict is the expensive per-item work the cap exists to ration. On day one everything is unlabeled, so the pass has to walk the untriaged pile in some triage-independent order — issue and PR number, or recency — and only *once* items carry a `triage-*` label can a later pass work the must-haves first. Priority ordering is a property you earn by triaging, not a shortcut you apply before it. Even so, the ceiling drains the backlog steadily instead of in one loud blast — and "steadily" is doing real work in that sentence. The 3-per-author cap sets a hard floor on how fast any single pile can clear: the author with 37 open PRs needs at least ⌈37 / 3⌉ — about thirteen days — before their queue is worked through, and that is *by design*. Draining is measured in weeks, not an afternoon. The slowness is the feature: it self-throttles the flood down to a humane trickle. If the answer to machine-speed contribution is machine-speed moderation, you have just moved the slop from the PR queue to the notification feed.

## What this means for CONTRIBUTING and AGENTS

The daily session convinced me of one more thing: this cannot live only in a maintainer's head or a local `priorities.md`. If the evidence gate is the real acceptance criterion, then `CONTRIBUTING.md` has to *say so* — attach a repro or a failing-on-`main` test, disclose AI assistance, keep the diff to one concern, don't flood — and `AGENTS.md` has to say the same thing in the dialect an agent will actually read before it opens a PR on your behalf. The gate should be discoverable *before* the contribution, not sprung on the author afterward. An expectation you never wrote down is not a policy; it is a grudge.

## The part I actually like

I have ended nearly every Spec Kit post with what I appreciate about the design, and I am not going to break the habit for the uncomfortable one.

What I like is that AI source forces a clarifying question the old world let us dodge: *what were we ever really accepting when we accepted a pull request?* We told ourselves it was code. It was never code. Code was always cheap relative to the thing that made it trustworthy — the evidence that it solved a real problem, the test that proved it, the person who understood it well enough to stand behind it. Agents did not break open source. They just called its bluff.

Spec Kit exists to move intent out of one person's head and into artifacts everyone can argue with. The evidence gate is the same idea pointed back at the contribution process itself: state what "acceptable" means, make clearing it cost something real, and let humans and agents meet the same bar. If AI source is the future — and from where I sit, it plainly is — then the projects that thrive will be the ones that stopped rewarding plausible code and started rewarding proof.

If you maintain a project that is starting to feel the same flood — or you think an evidence gate is exactly the wrong answer and you want to tell me why — send an email to blog (at) manorrock.com. That is an argument I would genuinely like to have.

---

*Further reading: the [Spec Kit repository](https://github.com/github/spec-kit) and its [Community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md), [A Year of Spec Kit — Notes from Washington](../../08/31/spec_kit_one_year_in_washington.html), [Spec Kit Turns One — and Ships 1.0.0](../../08/21/spec_kit_turns_one.html), [Spec Kit Learnings from Vidocq](../../07/29/spec_kit_sdd_in_practice.html), and [Part 12: The Road Ahead](../../07/14/spec_kit_road_ahead.html).*
