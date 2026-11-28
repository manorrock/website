# Spec Kit, Part 6: From Spec to Kit — More Than One Process

*The sixth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

At the end of [Part 5](../02/spec_kit_process.html) I let a phrase slip that deserves a post of its own. I said the core sequence — constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge — is *the reference process*, "not the boundary," and that it "was the first process — the **Spec** — and everything I spent four posts describing is the **Kit** that grew up around it." That is a tidy line to end on, but it hides the more interesting claim underneath: Spec Kit is not one process with some knobs. It is a *kit of processes*. This post is about taking that seriously.

The reason it matters is that "the spec-driven process" is easy to hear as a single, fixed pipeline that the rest of the toolkit merely decorates. If that were true, everything in Parts 1 through 4 would be cosmetics on one immovable loop. It is not true, and the gap between those two readings is exactly what the name *Spec Kit* is trying to signal. There is a reference process — the one Part 5 walked command by command — and then there is a kit for building, tailoring, and running processes, of which the reference one is simply the first and most central instance.

## Two routes to a different process

Here is the part I want to be precise about, because it is where I have seen the mental model go wrong. "More than one process" is reachable from two genuinely different directions, and they are not variations on the same trick.

The first route is **additive**: you install a whole new process *alongside* the core one, with its own commands and its own artifacts, leaving the reference sequence untouched. This is what extensions do — the Part 2 primitive.

The second route is **transformative**: you reshape the reference sequence itself — its steps, its terminology, even its language — until, from where you sit as a user, you are running a process tailored to you rather than the stock one. This is what presets do — the Part 3 primitive.

Same destination — a process that is not the vanilla core — reached two completely different ways. Worth taking each in turn.

## Route one: an extension can be a whole process

In Part 2 I described extensions as the primitive that *adds* — new commands, templates, scripts, hooks — and I leaned on small examples: a quality gate, a domain command. What I underplayed is that "add a command" scales all the way up to "add an entire process." Because extension commands live in their own namespace, nothing stops an extension from shipping a *coherent set* of commands that, together, constitute a self-contained workflow with its own beginning, middle, and end.

The clearest example ships with Spec Kit today. Install the **bug** extension:

```bash
specify extension add bug
```

and you get a three-step bug-triage process that runs entirely parallel to the core sequence:

- **`speckit.bug.assess`** reads a bug report — pasted text or a URL — judges whether it is a real bug, locates the suspect code paths, and proposes a remediation.
- **`speckit.bug.fix`** applies that remediation and records exactly what changed.
- **`speckit.bug.test`** re-runs the reproduction, then records the verification result.

Look at the shape of it and you will notice it *rhymes* with the core process without being it. Where the reference sequence keeps each feature under `specs/<feature>/` with `spec.md`, `plan.md`, and `tasks.md`, the bug process keeps each bug under its own directory and communicates through three Markdown reports:

```text
.specify/bugs/<slug>/
├── assessment.md   # written by speckit.bug.assess
├── fix.md          # written by speckit.bug.fix
└── test.md         # written by speckit.bug.test
```

The `<slug>` is the one handle the three commands share — the bug's analog of the feature branch that scopes the core sequence. And it carries its own opinions the way the core process carries the persistence models: `assess` and `test` never touch source code, `fix` is the only command allowed to edit it, and a reproduction that was not actually performed is reported as `partial` or `not-run`, never `verified`. That is not a command bolted onto the spec-driven loop. It is a *different loop* — assess-fix-test instead of specify-plan-implement — that happens to reuse the same machinery for namespacing, artifacts, and agent registration.

That is the thing to internalize about the additive route: an extension is not limited to widening the core process. It can stand up a whole one next to it, and the resolution stack keeps the two from colliding because everything the bug extension provides lives under its own names and its own directory. The core process does not even know it is there.

## Route two: a preset can make the core process *yours*

The additive route adds new slots. The transformative route reshapes the slots you already have — and this is the one the reader I was writing for pushed back on, correctly, when I first framed "new processes" as an extensions-only story. From the perspective of the person actually typing the commands, a preset can deliver a process every bit as new.

Recall from Part 3 what a preset is allowed to do: it provides its own command files, which get *registered with your agent* just like the core's, and it can substitute them for the built-in steps outright. The `lean` preset does exactly this — it `replaces` `speckit.specify`, `speckit.plan`, `speckit.tasks`, and the rest with trimmed-down versions. Nothing new appears in your command list, and yet the process you run is materially different: leaner artifacts, fewer ceremonies, a different rhythm. To the user, "the spec-driven process" now behaves a way the stock one never did.

Now stack that with the other two things presets can reshape — **terminology** and **localization of the entire experience** — and the effect compounds. A preset can rename the artifacts to your team's vocabulary, restructure `plan-template.md` to your methodology, append a compliance section to every spec, and render the whole thing in another language. Same command *names*, same underlying specify-plan-implement skeleton, but the process a developer experiences is one tailored to their organization end to end. It is, from their seat, a new process — not because a new pipeline was installed beside the old one, but because the old one was reshaped until it stopped looking stock.

And presets do not stop at the core. Because they sit *above* extensions in the resolution stack — the ordering I laid out in Part 3 — a preset can reshape an installed *extension* just as readily as it reshapes the built-in steps, using the same replace, prepend, append, and wrap strategies. So the two routes are not sealed off from each other. You can add the bug process with an extension, then point a preset at it — trim its reports, re-term its verdicts, localize its prompts — and now even the process you *added* is one tailored to you. The transformative route reaches over everything beneath it, core and extensions alike, which is exactly why a preset is the one layer that can bring a whole setup — however it is assembled — in line with your standards.

So the distinction between the two routes is clean and worth stating plainly: **an extension adds new slots; a preset reshapes the existing ones — whichever layer those slots came from.** Extensions give you a process the core did not have. Presets give you *the* process, made yours. Both leave the user running something other than vanilla Spec Kit, and both are why "one process" is the wrong way to picture the tool.

## Why the Spec became a Kit

Put the two routes together and the name stops being a pun. **Spec** is the first process — the reference sequence, the thing the whole toolkit is named for and organized around. **Kit** is everything that turned that single process into a medium for many: extensions that stand up parallel processes, presets that tailor the central one, integrations that translate whichever result into your agent's dialect, and bundles that pin a coherent set of all of it into a one-command install. The catalogs are what make those processes *shareable* rather than local curiosities — a bug workflow, a compliance-flavored core, a migration stack, all discoverable and installable the same way.

That is the arc the series has quietly been tracing. I started at the top of the stack precisely because the composition machinery is the part that reveals Spec Kit to be a kit at all. Had I opened with the reference process, it would have read as "here is the pipeline, here are some settings." Ending here, with the process in full view *and* the two routes for departing from it, the shape is clearer: the reference process is the center of gravity, and the kit is the set of forces you can apply around it.

It is tempting to give the shift a name. **Spec-Driven Development** — SDD — describes what happens *inside* one process: a specification drives an implementation. But once you are choosing which processes to install, tailoring the central one, and composing the set into something your team runs, you are operating a level up from any single spec. Call it **Kit-Driven Development** — KDD. If SDD is driving the code from a spec, KDD is driving the *process itself* from a kit. I do not offer that as an official rename, and I am not retiring SDD — the reference process is still spec-driven to its core, and every process in the kit still bottoms out in specifications driving implementations. I offer KDD as the honest description of the altitude you climb to the moment you stop consuming Spec Kit's one process and start assembling your own. SDD is what you do with a spec; KDD is what you do with the kit.

## Why I like this design

What I appreciate is that neither route required inventing a "processes" feature. There is no `specify process` command, no second-class plugin type for "whole workflows." The additive route falls out of extensions being namespaced; the transformative route falls out of presets being able to replace command files in a resolution stack. The primitives were general enough that a whole new process, or a wholesale reshaping of the central one, is just those primitives used at full strength. That is the sign of a design that composes rather than accretes — the ambitious use case and the modest one run through the same machinery, and the system never had to grow a special case to allow it.

It also means the ceiling is set by imagination rather than by the toolkit. The bug extension is the nearest example of a process someone authored rather than inherited; there is no reason it should be the last. If you have a repeatable workflow — incident review, dependency upgrades, security triage — the kit already has room for it.

That authored-process idea is the doorway to where the series goes next. Standing up a process is one thing; making it run *automatically*, end to end, repeatably, is another — and that is the layer I have deferred twice now. Next time I will finally pick it up: the automation machinery that turns a process into a run — KDD in motion. If there is a particular capability you would like me to dig into first, send an email to blog (at) manorrock.com.

---

*Further reading: the [bug extension](https://github.com/github/spec-kit/tree/main/extensions/bug), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), the [Spec Kit Extensions reference](https://github.com/github/spec-kit/blob/main/docs/reference/extensions.md), the [Spec Kit Presets reference](https://github.com/github/spec-kit/blob/main/docs/reference/presets.md), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
