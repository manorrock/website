# Practicing Spec-Driven Development Without Spec Kit — What Vidocq Learned

*A look at a real, large-scale Spec-Driven Development experiment — the [Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/) runtime — and what its lessons mean for anyone using [Spec Kit](https://github.com/github/spec-kit).*

Most of what gets written about AI-assisted development is either a breathless demo or a warning. What is rare is a careful field report from a team that ran a disciplined process at real scale and wrote down, honestly, what worked and what quietly went wrong. The [Vidocq interview](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/) is exactly that, which is why it is worth a close read.

## The interview

Antoine Sabot-Durand and Yann Blazart built **Vidocq**, a from-scratch Jakarta EE Core Profile 11 + MicroProfile 7.1 runtime, in roughly three and a half months — pure Java 25, zero dependencies — as a deliberate **Spec-Driven Development** experiment. The interview is their account of how they did it, and their method stood on three legs:

1. **A formal spec as the requirements.** The twenty-year-old Jakarta and MicroProfile specifications — untouchable, unambiguous, versioned. Not a document they drafted and revised, but a contract held fixed.
2. **An executable oracle.** The TCK — thousands of tests as a single binary pass/fail gate, which they called "the only reviewer that never gets tired." The AI was never allowed to *declare* a feature done; the suite is green or it is not.
3. **TDD to connect the two.** Red → green → refactor, the test written before the code, so every line the agent produced was answering a failing check rather than a hunch.

Around those legs they built the rest: a `CLAUDE.md` context hierarchy re-read every session, guardian sub-agents watching the invariants that mattered most (JPMS boundaries, the zero-dependency rule, virtual-thread discipline), a design → plan → execution pipeline with the *plan* reviewed by a human before any code was generated, and a trace *after* every change — the commit, a `BUG.md`, a `lessons.md`, ADRs.

## What they learned

The value of the interview is that its lessons are portable — none of them depend on Jakarta, on Claude, or even on having a TCK.

**Spec before code; plan before generation.** Their sharpest line is *"AI produces the plausible, not the correct."* An unlocked spec and an unreviewed plan are just invitations for the model to fill the gaps with something that reads well and is wrong.

**A spec is worthless without an executable oracle.** In their words, "'done' means nothing" without a binary signal. A specification the AI can interpret but not be *checked* against will always drift toward a plausible-looking finish line.

**Constraints must be verifiable rules, not intentions.** "No `ThreadLocal`" is checkable; "be performant" is not. A guardrail no one — human or agent — can mechanically test is decoration, not governance.

**Writing the rules down is not enough — you have to audit compliance.** This is the most valuable finding in the whole piece. A mid-June audit caught the agent drifting from its own rules *despite the rule living in every `CLAUDE.md`*. The model "optimises locally and drifts silently." Tellingly, they adopted their design → plan → execution pipeline only *after* that audit — the discipline was a reaction to being burned, not a precaution.

**Humans own design; AI executes it.** They measured roughly a 30× speedup on scaffolding and close to 1× on architecture decisions. The bottleneck was never typing — it was *supervision*. The tool made the cheap part cheaper and left the expensive part exactly as expensive.

**Trace everything.** The only difference between deliberate technical debt and an accident is the recorded reasoning — the commit, the `BUG.md`, the ADR.

**Guard against doc rot.** A status document not backed by an executable proof becomes poison for the next AI pass, which reads it as ground truth. They wanted, only half-joking, "CI for docs."

## How it applies to Spec Kit

Read those lessons next to Spec Kit and something striking emerges: a team practicing Spec-Driven Development entirely by hand independently arrived at nearly every principle Spec Kit builds in. The mapping is almost one to one.

- *Spec before code; plan before generation* is the entire reason the reference sequence puts `/speckit.specify` and `/speckit.clarify` ahead of `/speckit.plan`, and reviews the plan before `/speckit.implement`.
- *Constraints as verifiable rules* is the best possible advice for writing a Spec Kit **constitution** — a principle that cannot be checked is not a rule.
- *Audit compliance, do not just write it down* is what an `/speckit.analyze` gate or a review extension is for: something that reads the artifacts back and complains when they have drifted.
- *Trace everything* is precisely why Spec Kit keeps `spec.md`, `plan.md`, and `tasks.md` versioned in the repo — the artifacts *are* the paper trail, not scratch files you delete.
- *Guard against doc rot* is the standing risk for any of those artifacts once the code moves on without them.

The one lesson Spec Kit cannot hand you is the executable oracle. Vidocq had a rare luxury there — a TCK that made "done" a binary fact — and that luxury is exactly why, when the interviewer asked whether they had looked at Spec Kit, they said they had not seriously:

> "Those tools mostly solve the problem of writing a good spec when you don't have one. We had the opposite luxury."

That is not a knock on Spec Kit; it is a clean statement of what Spec Kit is *for*. Vidocq started with a mature spec **and** a ready-made oracle. Most teams have neither — a Slack thread, a half-formed issue, a deadline — and Spec Kit's `/speckit.constitution` → `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement` sequence is how you build the spec from scratch. What Spec Kit does *not* invent for you is the TCK's role: the binary "done" signal. When you do not have a real conformance suite, tests, CI, and `/speckit.analyze` are what you assemble in its place — an approximation of the one leg Vidocq got for free.

## Why their report matters

It would be easy to discount a glowing "we used the tool and shipped" post. Vidocq is more useful precisely because it is *not* that. Here is a team that did not reach for Spec Kit at all — they judged that those tools solve a problem, writing a good spec, they did not have — and then, practicing SDD by hand, rediscovered nearly every principle the toolkit encodes: spec before code, a binary oracle, verifiable constraints, audited compliance, human-owned design, traceable decisions, doc rot as a real hazard.

That is the outcome you hope for when you believe in an idea rather than a product: people who solve the same problem independently end up standing in the same place. Spec Kit is a good on-ramp to Spec-Driven Development when you start from a blank repo and a vague idea. Vidocq is a reminder of where that road leads when you walk it far enough — and how much of what they learned the hard way is already sitting in the kit, waiting to be turned on.

If you have practiced SDD at scale — with Spec Kit or, like Vidocq, without it — I would genuinely like to hear which of these lessons held for you and which broke. Send an email to blog (at) manorrock.com.

---

*Further reading: [Spec-Driven Development in Practice: an interview with the creators of Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), and the [Spec Kit Quick Start Guide](https://github.com/github/spec-kit/blob/main/docs/quickstart.md).*
