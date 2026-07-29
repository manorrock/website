# Practicing Spec-Driven Development Without Spec Kit — What Vidocq Learned

*A companion to the [Spec Kit series](../../06/26/spec_kit_bundles.html). No part number on this one — the twelve-part arc was about the kit; this is about a team that practiced the process the kit is named for, entirely without the kit, and wrote down what it cost them to learn.*

Every post in the [series](../../06/26/spec_kit_bundles.html) I finished earlier this month was, underneath the specific topic, an argument for one conviction: get the *spec* right, keep it the source of truth, and the code follows. But a conviction is only as good as the evidence for it, and the most convincing evidence is a team that reached the same conclusions *without* the tool that encodes them — because then you know you are looking at the idea, not the marketing.

That team is [Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/): a from-scratch Jakarta EE Core Profile 11 + MicroProfile 7.1 runtime that Antoine Sabot-Durand and Yann Blazart built in roughly three and a half months, in pure Java 25 with zero dependencies, as a deliberate **Spec-Driven Development** experiment. They deliberately did *not* use Spec Kit — a detail I will get to, but not the point. The point is that they ran SDD at real scale, kept honest notes on what worked and what quietly went wrong, and those notes read like a field report from the far side of the exact process Spec Kit automates. This post is about what they learned, and how each lesson maps back onto the kit.

## The shape of what they practiced

Before the lessons, the method, because the lessons only make sense against it. Vidocq's SDD stood on three legs:

1. **A formal spec as the requirements.** The twenty-year-old Jakarta and MicroProfile specifications — untouchable, unambiguous, versioned. Not a document they drafted and revised, but a contract held fixed.
2. **An executable oracle.** The TCK — thousands of tests as a single binary pass/fail gate, which they called "the only reviewer that never gets tired." The AI is never allowed to *declare* a feature done; the suite is green or it is not.
3. **TDD to connect the two.** Red → green → refactor, the test written before the code, so every line the agent produced was answering a failing check rather than a hunch.

Around those legs they built the rest: a `CLAUDE.md` context hierarchy re-read every session, guardian sub-agents watching the invariants that mattered most (JPMS boundaries, the zero-dependency rule, virtual-thread discipline), a design → plan → execution pipeline with the *plan* reviewed by a human before any code was generated, and a trace *after* every change — the commit, a `BUG.md`, a `lessons.md`, ADRs. That is the setup. What they learned running it is the interesting part, and it is not what a demo would show you.

## What they learned

None of these lessons depend on Jakarta, on Claude, or on having a TCK. Every one is something a Spec Kit user could act on tomorrow — and most of them are things Spec Kit already tries to do for you, which is exactly why they are worth surfacing.

**1. Spec before code; plan before generation.** Their sharpest line is *"AI produces the plausible, not the correct."* An unlocked spec and an unreviewed plan are just invitations for the model to fill the gaps with something that reads well and is wrong. This is the entire premise of the reference sequence — `/speckit.specify` and `/speckit.clarify` before `/speckit.plan`, and the plan reviewed before `/speckit.implement` — stated by a team who felt the cost of skipping it.

**2. A spec is worthless without an executable oracle.** In their words, "'done' means nothing" without a binary signal. This is the leg most teams are missing, and it is the one Vidocq had for free in the TCK. When you do not have a TCK, tests, CI, and Spec Kit's `/speckit.analyze` coverage pass are what you are assembling in its place — an approximation of the thing that let Vidocq trust the word "done."

**3. Constraints must be verifiable rules, not intentions.** "No `ThreadLocal`" is checkable; "be performant" is not. This is the best single piece of advice for writing a Spec Kit **constitution**: a principle that no agent — and no human — can mechanically check is decoration, not governance.

**4. Writing the rules down is not enough — you have to audit compliance.** This is the most valuable thing in the whole interview. A mid-June audit found the agent had been drifting from its own rules *despite the rule living in every `CLAUDE.md`*. The model "optimises locally and drifts silently." Rules in a file are a hope; a sentinel that re-checks them is a control. It is also why they adopted their design → plan → execution pipeline only *after* the audit — the discipline was a reaction to being burned, not a precaution. Spec Kit's answer to the same failure is a review extension or an `analyze` gate: something that reads the artifacts back and complains.

**5. Humans own design; AI executes it.** They measured roughly a 30× speedup on scaffolding and close to 1× on architecture decisions. The bottleneck was never typing — it was *supervision*. The tool made the cheap part cheaper and left the expensive part exactly as expensive. This is the honest counterweight to every "AI wrote the whole app" demo, and it is why Spec Kit puts a human review between each phase rather than racing from idea to code.

**6. Trace everything.** The only difference between deliberate technical debt and an accident is the recorded reasoning — the commit, the `BUG.md`, the ADR. This is precisely the instinct behind Spec Kit keeping `spec.md`, `plan.md`, and `tasks.md` versioned in the repo: the artifacts *are* the paper trail, not scratch files you delete.

**7. Guard against doc rot.** A status document not backed by an executable proof becomes poison for the next AI pass, which reads it as ground truth. They wanted, only half-joking, "CI for docs" — a real and still-unmet need, and a caution for anyone whose Spec Kit artifacts drift out of sync with the code they describe.

## About skipping Spec Kit

Now the detail I set aside. A team this disciplined about SDD not reaching for the toolkit built for SDD demands an explanation, and theirs (question ten of the interview) is precise rather than dismissive:

> "Those tools mostly solve the problem of writing a good spec when you don't have one. We had the opposite luxury."

That is not a knock on Spec Kit — it is a clean statement of what Spec Kit is *for*. Vidocq began with two things almost no product team ever has: a mature, unambiguous specification hardened over two decades, **and** a ready-made executable oracle in the TCK. The single hardest move in Spec-Driven Development — turning a fuzzy intent into a spec precise enough to build against, then into something that can *prove* done — was handed to them.

Most of us do not get that gift. We have a Slack thread, a half-formed issue, and a deadline; the spec does not exist yet, and neither does the oracle. That gap — from "we have an idea" to "we have a spec and a way to verify it" — is exactly what Spec Kit's `/speckit.constitution` → `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement` sequence exists to walk you across. Vidocq shows you the far side of that sequence; Spec Kit is how you *reach* it when you are not lucky enough to start there. Same idea, opposite starting points — which is why I never thought of this as "Vidocq versus Spec Kit."

## Why their report reassures me

What I appreciate is that this is *not* a Spec Kit success story, and it is more useful for that. Here is a team that understood the toolkit well enough to explain exactly what it is for, decided their situation did not need it — and then, practicing SDD by hand, independently rediscovered nearly every principle the toolkit encodes. Spec before code. A binary oracle. Verifiable constraints. Audited compliance. Human-owned design. Traceable decisions. Doc rot as a real hazard.

That is the outcome you hope for when you believe in an idea rather than a product: not that everyone adopts your tool, but that people who solve the same problem without it end up standing in the same place. Spec Kit is a good on-ramp to Spec-Driven Development when you are starting from a blank repo and a vague idea. Vidocq is a reminder of where that road leads when you walk it far enough by hand — and how much of what they learned the hard way is already sitting in the kit, waiting to be turned on.

If you have practiced SDD at scale — with Spec Kit or, like Vidocq, without it — I would genuinely like to hear which of these lessons held for you and which broke. Send an email to blog (at) manorrock.com.

---

*Further reading: [Spec-Driven Development in Practice: an interview with the creators of Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 8: A Field Guide to the Community](../08/spec_kit_field_guide.html), [The Spec Kit Copilot Plugin](../24/spec_kit_copilot.html), and the [full series](../../06/26/spec_kit_bundles.html).*
