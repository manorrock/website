# Spec-Driven Development in the Wild — What Vidocq Teaches About Spec Kit

*A companion to the [Spec Kit series](../../06/26/spec_kit_bundles.html). No part number on this one — the twelve-part arc was about the kit; this is about a team that practiced the process the kit is named for, and never touched the kit.*

The [series](../../06/26/spec_kit_bundles.html) I finished earlier this month spent twelve posts on [Spec Kit](https://github.com/github/spec-kit) — the bundles, extensions, presets, integrations, and the reference process underneath them. Every one of those posts was, in the end, about the same conviction: that if you get the *spec* right, and keep it the source of truth, the code follows. That is a claim, and claims are more convincing when someone proves them somewhere you were not looking.

So this post is not about Spec Kit at all — until the end. It is about [Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/), a from-scratch Jakarta EE Core Profile 11 + MicroProfile 7.1 runtime that Antoine Sabot-Durand and Yann Blazart built in roughly three and a half months, in pure Java 25 with zero dependencies, as a deliberate **Spec-Driven Development** experiment. They wrote it up as an interview, and what makes it worth your time is not the runtime — it is that they ran SDD at genuine scale, wrote down what actually happened, and, tellingly, *chose not to use Spec Kit*. Reading their account next to the process I spent a series describing is the closest thing to an independent audit of the whole idea that I have found.

## Vidocq's SDD rests on three legs

Strip the write-up down and their method stands on three supports, each load-bearing:

1. **A formal spec as the requirements.** The twenty-year-old Jakarta and MicroProfile specifications — treated as untouchable, unambiguous, versioned. Not a document they wrote and revised; a contract handed to them by standards bodies and held fixed.
2. **An executable oracle.** The TCK — thousands of tests as a single binary pass/fail gate. Their phrase for it is "the only reviewer that never gets tired." The AI is never allowed to *declare* a feature done; the suite is green or it is not, and nothing in between counts.
3. **TDD to connect the two.** Red → green → refactor, with the test or scenario written before the code, so every line the agent produces is answering a failing check rather than a vibe.

Around those three legs they built the scaffolding you would expect from anyone doing this seriously: a `CLAUDE.md` context hierarchy re-read every session, guardian sub-agents watching the invariants that mattered most (JPMS boundaries, the zero-dependency rule, virtual-thread discipline), a design → plan → execution pipeline, human review of the *plan* before any generation, and a trace *after* — the commit, a `BUG.md`, a `lessons.md`, ADRs. Notably, the pipeline was not there from day one. They adopted it only after a mid-June audit found the agent had been drifting silently from its own rules, which is a detail I will come back to because it is the most useful thing in the whole interview.

## Why they skipped Spec Kit — and why that is the interesting part

The obvious question, given everything above, is why a team this disciplined about Spec-Driven Development did not reach for the toolkit built for exactly that. Their answer (question ten of the interview) is worth quoting because it is precise rather than dismissive:

> "Those tools mostly solve the problem of writing a good spec when you don't have one. We had the opposite luxury."

That is not a knock on Spec Kit. It is a clean statement of what Spec Kit is *for*. Vidocq started with two things almost no product team ever has: a mature, unambiguous specification written and hardened over two decades, **and** a ready-made executable oracle in the TCK. The single hardest, most valuable move in Spec-Driven Development — turning a fuzzy intent into a spec precise enough to build against, and then into something that can *prove* "done" — was simply given to them for free.

Most of us do not get that gift. We have a Slack thread, a half-formed issue, and a deadline. The spec does not exist yet, and neither does the oracle. That gap — from "we have an idea" to "we have a spec and a way to verify it" — is precisely the gap Spec Kit is built to close, and it closes it with a sequence I have written about at length: `/speckit.constitution` → `/speckit.specify` → `/speckit.clarify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement`. Vidocq shows you what the far side of that sequence looks like when the spec and the oracle already exist. Spec Kit is how you *get* to that far side when they do not. Same foundation, opposite starting points — which is why I stopped thinking of this as "Vidocq versus Spec Kit" almost immediately. They are two views of one idea.

## Seven lessons that survive the trip

The reason the interview is more than a war story is that its lessons are portable. None of them depend on Jakarta, on Claude, or on having a TCK. Every one of them is something you can apply to a Spec Kit project — or to any AI-assisted workflow — tomorrow.

1. **Spec before code; plan before generation.** Never let the agent code against ambiguity. Their sharpest framing: *"AI produces the plausible, not the correct."* An unlocked spec and an unreviewed plan are just invitations for the model to fill the gaps with something that reads well and is wrong.
2. **You need an executable oracle.** A spec with no binary "done" signal is worthless — in their words, "'done' means nothing" without it. This is the leg most teams are missing, and it is exactly what tests, CI, and Spec Kit's `/speckit.analyze` consistency pass are approximating when a real TCK does not exist.
3. **Express constraints as verifiable rules, not intentions.** "No `ThreadLocal`" is checkable; "be performant" is not. This is the single best piece of advice for anyone writing a Spec Kit constitution: a principle a guardian agent — or a human — cannot mechanically check is decoration, not governance.
4. **Writing the rules down is not enough — audit compliance.** Their June 12 audit found the agent had drifted despite the rule living in every `CLAUDE.md`. The model "optimises locally and drifts silently." Rules in a file are a hope; a sentinel that re-checks them is a control.
5. **Humans own design; AI executes it.** They measured roughly a 30× speedup on scaffolding and close to 1× on architecture decisions. The bottleneck was never typing — it was *supervision*. The tool made the cheap part cheaper and left the expensive part exactly as expensive.
6. **Trace everything.** The only difference between deliberate technical debt and an accident is the recorded reasoning: the commit, the `BUG.md`, the ADR. This is the same instinct behind Spec Kit's versioned artifacts on disk — the spec, plan, and tasks are the paper trail, not scratch files.
7. **Guard against doc rot.** A status document not backed by an executable proof becomes poison for the next AI pass, which will read it as ground truth. They wanted, only half-joking, "CI for docs" — and that is a real and unmet need.

Read those with the series in front of you and the overlap is uncanny. Lesson 3 is the constitution done right. Lesson 4 is why an extension that runs a review pass exists. Lesson 6 is the whole argument for keeping `spec.md`, `plan.md`, and `tasks.md` in the repo. Vidocq arrived at Spec Kit's opinions from the outside, without the tool — which is about the strongest evidence I can imagine that the opinions are the right ones.

## Why I find this reassuring

What I appreciate about the Vidocq write-up is that it is *not* a Spec Kit success story, and it is more useful for that. It would have been easy to read a glowing "we used the tool and shipped" post and discount it. Instead here is a team that had every reason to use the toolkit, understood it well enough to explain exactly what it is for, decided their situation did not need it — and then independently rediscovered nearly every principle the toolkit encodes. Spec before code. A binary oracle. Verifiable constraints. Audited compliance. Human-owned design. Traceable decisions.

That is the outcome you want when you believe in an idea: not that everyone adopts your tool, but that people who solve the same problem without it end up in the same place. Spec Kit is a good way to get to Spec-Driven Development when you are starting from a blank repo and a vague idea. Vidocq is a reminder of where that road leads when you walk it far enough — a precise spec, an executable oracle, and a human keeping their hands on the design. The tool is the on-ramp. The destination is the same either way.

If you have run SDD at scale — with Spec Kit or, like Vidocq, without it — I would genuinely like to hear where the principles held and where they broke. Send an email to blog (at) manorrock.com.

---

*Further reading: [Spec-Driven Development in Practice: an interview with the creators of Vidocq](https://www.linkedin.com/pulse/spec-driven-development-practice-interview-creators-vidocq-goncalves-8tqjf/), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 8: A Field Guide to the Community](../08/spec_kit_field_guide.html), [The Spec Kit Copilot Plugin](../24/spec_kit_copilot.html), and the [full series](../../06/26/spec_kit_bundles.html).*
