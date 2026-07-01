# Spec Kit, Part 5: The Spec-Driven Process — What Every Layer Was Customizing

*The fifth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

Four posts in, I have described bundles, extensions, presets, and integrations — and every one of them has been about the same thing without ever saying its name. A bundle composes a setup; an extension adds to it; a preset reshapes it; an integration translates it for your agent. But *it* — the process all four are composing, adding to, reshaping, and translating — is the thing I have kept deferring. Part 1 promised I would circle back to "init, specs, the core spec-driven process." This is that post. This time I am not writing about a layer that customizes the process; I am writing about the process itself.

Starting the series at the top of the stack and working down to this was deliberate. The customization machinery is genuinely the freshest, most distinctive part of Spec Kit, and it makes a better opening than a march through first principles. But it only means anything because there is a real, opinionated process underneath it worth customizing. So this entry is the floor beneath the floor: the sequence of commands that turns an intent into working software, which every layer above exists to serve.

## What the process is for

The pitch behind Spec-Driven Development is a genuine inversion. For decades code was the artifact that mattered and the specification was scaffolding — something you wrote to get alignment, then quietly let rot the moment real coding began. Spec Kit refuses that ordering. Here the **specification is the source of truth**, and it stays executable: it is not a document you consult, it is the input that generates the plan, the tasks, and ultimately the implementation. The "what" and "why" are defined before the "how," and the "how" is derived from them rather than invented alongside them.

That is why the process is a sequence of distinct commands rather than one big "build me this" prompt. The whole bet is on **multi-step refinement over one-shot generation**. Each step produces a concrete artifact you can read, correct, and commit before the next step consumes it — so a misunderstanding surfaces in the spec, where it is cheap to fix, instead of in the code, where it is not.

## The core sequence

After `specify init` scaffolds a project, your coding agent gains a small set of slash commands (or skills, depending on the integration — the translation Part 4 was about). Run in order, they walk an idea from principles to running code:

1. **`/speckit.constitution`** — establish the project's governing principles and development guidelines. This comes first because everything downstream is meant to respect it: the constitution is the standing context that shapes how specs, plans, and tasks are written.
2. **`/speckit.specify`** — describe *what* you want and *why*, deliberately not the tech stack. This produces the specification — the artifact the rest of the process treats as the source of truth.
3. **`/speckit.plan`** — now bring in the "how": the tech stack, the architecture, the constraints. The plan is derived from the spec, not written independently of it.
4. **`/speckit.tasks`** — turn the plan into an actionable, ordered task list. This is where a plan becomes something you can execute piece by piece.
5. **`/speckit.implement`** — execute the tasks and build the feature according to the plan.

Read top to bottom, the shape is the whole philosophy in miniature: principles, then intent, then approach, then breakdown, then build — each step consuming the artifact the last one produced. Nothing is skipped ahead. You do not describe architecture while you are still deciding what to build, and you do not write code before the tasks exist.

## The optional steps that make it honest

The five commands above are the spine, but the process gets most of its rigor from a few optional steps you slot in between them — and knowing *where* they go is half the value:

- **`/speckit.clarify`** — run after `specify` and before `plan`. It surfaces the underspecified corners of your spec and asks about them, so you resolve ambiguity while it still lives in the specification rather than discovering it three steps later. (Longtime users may remember this as `/quizme`.)
- **`/speckit.analyze`** — run after `tasks` and before `implement`. It does cross-artifact consistency and coverage analysis: does every requirement in the spec have a task? Does the plan actually cover what the spec asked for? It is the checkpoint that catches drift between the artifacts before any of it becomes code.
- **`/speckit.checklist`** — generate custom quality checklists that validate a spec's completeness, clarity, and consistency. The framing I like best is the project's own: these are "unit tests for English." You are testing the specification the way you would test code.

None of these generate implementation. They exist purely to make the artifacts trustworthy before the next step consumes them — which is exactly what "multi-step refinement" has to mean if it is going to mean anything.

## Beyond the first pass

Two more commands acknowledge that real work is rarely one clean trip through the sequence:

- **`/speckit.taskstoissues`** converts a generated task list into GitHub issues, so the breakdown the process produced becomes trackable, assignable work instead of a list that lives only in your spec directory.
- **`/speckit.converge`** assesses the current codebase against the spec, plan, and tasks, and appends whatever remains as new tasks. This is the step for the second lap and beyond — when the code has moved on and you need the task list to catch back up to reality.

That last one points at something important: Spec Kit is not only a greenfield tool. It names three development phases — 0-to-1 greenfield, creative exploration, and iterative brownfield enhancement — and the process is meant to run in all of them. The guidance for existing projects is to keep two kinds of change separate: refreshing the managed tooling files when you upgrade Spec Kit, and evolving your `specs/` artifacts when the intended behavior actually changes. Confuse the two and you get spurious diffs; keep them apart and the specification stays a faithful record of intent as the project grows.

And this is where I want to be careful with a word. A single trip through the sequence — constitution, specify, plan, tasks, implement — is exactly that: a *sequence*, linear and directional, not something that folds back on itself. But a real product is never one spec. You run the whole sequence for one feature, land it, and then start it again for the next, and the next — a fresh spec, its own plan and tasks, another pass to `implement`. So at the level of a single feature the process is a straight line, while at the level of a project it is unmistakably iterative: the same sequence traversed again and again, once per spec, until the product is built. That repetition — many passes, many specs — is the real rhythm of working this way, and it is the one sense in which "loop" is the honest word. The distinction matters because the two levels want different things from you: within a pass, keep moving forward and let each step refine the last; across passes, keep each spec small enough that a single trip through the sequence actually finishes.

## How the whole series hangs off this

Here is the payoff of having saved this for last. Go back through the previous four posts with the core sequence in front of you and every one of them is defined *relative to it*:

- A **preset** reshapes the templates and commands this process runs — a leaner `speckit.specify`, a compliance section appended to every plan.
- An **extension** adds a step this process did not have — a review pass after `implement`, a diagnostic the core never shipped.
- An **integration** renders these commands into the exact form your agent expects, so `/speckit.plan` is something you can actually type.
- A **bundle** pins a coherent set of the above so a teammate gets the whole customized process in one command.

Every layer in the series is a modifier on the nouns in this post. That is why the resolution stack Part 3 laid out has *core* at the bottom: the process described here is the default that everything else layers over. Bundles compose it, extensions extend it, presets customize it, integrations translate it — and none of that vocabulary means anything until you know what the "it" is.

## Why I like this design

What I appreciate about the core process is that it is genuinely opinionated without being rigid. It could have been a single mega-prompt that tried to go from idea to app in one leap — impressive in a demo, unmaintainable in practice. Instead it is a sequence of small, inspectable steps, each producing an artifact you can correct before it propagates, with optional checkpoints exactly where ambiguity and drift tend to hide. The result is that the specification earns its place as the source of truth, because the process keeps giving you chances to make it right before it turns into code.

Seen against the rest of the series, this is the piece that justifies all the others. I spent four posts on the machinery for customizing, extending, translating, and composing a process — and it was worth building that machinery precisely because the process at the center is solid enough to be worth making your own. It is fitting that the foundation is the last thing I named.

That closes the arc I set out at the start: top of the stack down to the ground it stands on. Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit README](https://github.com/github/spec-kit/blob/main/README.md), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
