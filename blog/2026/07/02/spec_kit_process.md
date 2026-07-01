# Spec Kit, Part 5: The Spec-Driven Process — What Every Layer Was Customizing

*The fifth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

Four posts in, I have described bundles, extensions, presets, and integrations — and every one of them has been about the same thing without ever saying its name. A bundle composes a setup; an extension adds to it; a preset reshapes it; an integration translates it for your agent. But *it* — the process all four are composing, adding to, reshaping, and translating — is the thing I have kept deferring. Part 1 promised I would circle back to "init, specs, the core spec-driven process." This is that post, and I want to ground it in what the docs actually specify rather than the one-line pitch, because the process has far more structure than "write a spec, then build it."

Starting the series at the top of the stack and working down to this was deliberate. The customization machinery is the freshest, most distinctive part of Spec Kit, and it makes a better opening than a march through first principles. But it only means anything because there is a real, opinionated process underneath it worth customizing. So this entry is the floor beneath the floor: the sequence of commands that turns an intent into working software, which every layer above exists to serve.

## The full sequence, not the slogan

After `specify init` scaffolds a project, your coding agent gains a set of slash commands (or skills, depending on the integration — the translation Part 4 was about). The quickstart documents the complete recommended sequence, and it is worth seeing in full before dissecting it:

```text
/speckit.constitution → /speckit.specify → /speckit.clarify → /speckit.plan →
/speckit.checklist → /speckit.tasks → /speckit.analyze → /speckit.implement →
/speckit.converge
```

That is nine commands, and there is a tenth — `/speckit.taskstoissues` — that branches off to the side. Most write-ups (mine in the earlier posts included) collapse this to "specify, plan, implement" and move on. But the commands the docs put *between* those headline steps are where spec-driven development earns its name, so I want to walk the whole thing.

Two commands set the ground and the goal:

- **`/speckit.constitution`** establishes the project's governing principles — the standing rules every later step is meant to respect. This is why it comes first and why it is not per-feature: it is the context that shapes how specs, plans, and tasks get written across the whole project.
- **`/speckit.specify`** describes *what* you want and *why*, deliberately not the tech stack. It produces the specification, the artifact the rest of the process treats as the source of truth.

Three commands turn intent into an executable breakdown:

- **`/speckit.plan`** brings in the *how* — stack, architecture, constraints — derived from the spec rather than written independently of it.
- **`/speckit.tasks`** turns the plan into an ordered, actionable task list.
- **`/speckit.implement`** executes those tasks and builds the feature.

## The quality gates are the point

The commands I skipped in that walk — `clarify`, `checklist`, `analyze` — are the ones that make the difference between spec-driven development and a fancy prompt. The docs are explicit that there are two ways to run the process. Quick experiments can take the **lean path**: `specify → plan → tasks → implement`. But for production features or any work with meaningful ambiguity, three commands become regular **quality gates**, and *where* they sit is the whole trick:

- **`/speckit.clarify`** runs after `specify` and before `plan`. It hunts down the underspecified corners of the spec and asks about them, so ambiguity gets resolved while it still lives in the specification — not three steps later in the code. (Longtime users knew this as `/quizme`.)
- **`/speckit.checklist`** runs after `plan`. It generates custom quality checklists that validate the requirements for completeness, clarity, and consistency — the project's own framing is "unit tests for English." Note the position: it validates against the plan, not the bare spec, which is a detail I got wrong until I read the docs closely.
- **`/speckit.analyze`** runs after `tasks` and before `implement`. It does cross-artifact consistency and coverage analysis: does every requirement have a task, does the plan actually cover the spec? The docs are firm that the first analysis belongs *before* `implement`, so gaps are caught while the plan and tasks can still change — you may re-run it afterward as an extra review, but not instead.

None of these three generate implementation. They exist purely to make the artifacts trustworthy before the next step consumes them, which is what "multi-step refinement" has to mean if it means anything.

## Feature context rides on your branch

One detail that is easy to miss but changes how the process feels day to day: Spec Kit commands detect the active feature from your current Git branch (something like `001-photo-albums`). Each feature gets its own directory under `specs/` with its own `spec.md`, `plan.md`, and `tasks.md`, and switching features is as simple as switching branches. So the sequence above is not a global mode you are in — it is scoped to one feature at a time, and the artifacts on disk are the durable record of where each feature stands.

## Implement is not one shot

The command I most under-described before was `implement`, and the docs devote a whole page to why. Large features tend to sail through `specify`, `plan`, and `tasks`, then degrade midway through a long `implement` run — the agent loses the plan or starts to hallucinate as the context window fills. Crucially, `implement` accepts free-form input, so you can scope it without any tooling change:

```text
/speckit.implement only execute tasks T001-T010, then stop and report progress
```

Because completed tasks are marked `[X]` in `tasks.md`, the next invocation picks up where the last left off. From there the docs escalate: delegate parallel `[P]` tasks to sub-agents so each gets a focused context; and when even a single phase is too big, decompose the feature into smaller specs — the "spec of specs" approach, where the first pass breaks a massive feature into self-contained specs that each run through their own specify/plan/tasks/implement cycle. The headline command hides a real operational discipline.

Two more commands live at the edges of the core sequence. **`/speckit.taskstoissues`** converts a task list into GitHub issues so the breakdown becomes trackable, assignable work. And **`/speckit.converge`** assesses the current codebase against the spec, plan, and tasks and appends whatever is missing as new tasks — then you run `implement` and `converge` again, and keep going until it reports the feature has actually converged.

## What happens when requirements change

Here is the part the docs treat as first-class and the marketing pitch never mentions: Spec Kit deliberately does *not* decide what happens to `spec.md`, `plan.md`, and `tasks.md` after requirements change. It names three **spec-persistence models** and leaves the choice to your team as a convention, not a CLI setting:

- **Flow-back** — edits can start in any artifact (even during implementation), and the team reconciles the set afterward. Fast, but risks silent drift if a change never makes it back into the spec.
- **Flow-forward** — completed feature directories are immutable history; a new requirement means a *new* directory under `specs/`. Great for auditability, at the cost of duplication.
- **Living spec** — `spec.md` is the contract, and `plan.md` and `tasks.md` are regenerated from it whenever it changes. Keeps everything consistent, but you can lose implementation rationale if you discard derived artifacts carelessly.

This is exactly where the distinction I raised last time lands. A single trip through the sequence — constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge — is a *sequence*: linear and directional, not something that folds back on itself. But a real product is never one spec. You run the whole sequence for one feature, land it, and start it again for the next, once per spec, until the product is built. At the level of a feature it is a straight line; at the level of a project it is unmistakably iterative — the same sequence traversed again and again. That cross-spec repetition is the one honest sense in which "loop" is the right word, and the persistence models are precisely the rules for how each new pass relates to the ones before it.

## How the whole series hangs off this

Go back through the previous four posts with the full sequence in front of you and every one of them is defined *relative to it*:

- A **preset** reshapes the templates and commands these steps run — a leaner `speckit.specify`, a compliance section appended to every plan.
- An **extension** adds a step this sequence did not have — a review pass after `implement`, a diagnostic the core never shipped.
- An **integration** renders these commands into the exact form your agent expects, so `/speckit.plan` is something you can actually type.
- A **bundle** pins a coherent set of the above so a teammate gets the whole customized process in one command.

Every layer in the series is a modifier on the nouns in this post. That is why the resolution stack Part 3 laid out has *core* at the bottom: the sequence described here is the default everything else layers over. Bundles compose it, extensions extend it, presets customize it, integrations translate it — and none of that vocabulary means anything until you know what the "it" is, all ten commands of it.

## Why I like this design

What I appreciate about the core process is that it is genuinely opinionated without being rigid. It could have been one mega-prompt that leapt from idea to app — impressive in a demo, unmaintainable in practice. Instead it is a sequence of small, inspectable steps, each producing an artifact you can correct before it propagates, with optional gates positioned exactly where ambiguity and drift hide, an implementation step honest about its own context limits, and a deliberate refusal to dictate how your artifacts age. The result is that the specification earns its place as the source of truth, because the process keeps giving you chances to make it right before it becomes code.

Seen against the rest of the series, this is the piece that justifies all the others. I spent four posts on the machinery for customizing, extending, translating, and composing a process — and it was worth building that machinery precisely because the process at the center is solid enough to be worth making your own. It is fitting that the foundation is the last thing I named.

That closes the arc I set out at the start: top of the stack down to the ground it stands on. Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Spec Persistence Models](https://github.com/github/spec-kit/blob/main/docs/concepts/spec-persistence.md), [Handling Complex Features](https://github.com/github/spec-kit/blob/main/docs/concepts/complex-features.md), the [Quick Start Guide](https://github.com/github/spec-kit/blob/main/docs/quickstart.md), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
