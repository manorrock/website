# Spec Kit with Copilot, Part 2: Sub-Agents — Distributing the Process Without Replacing It

*The second entry in a dedicated series about the Copilot-specific pieces around [Spec Kit](https://github.com/github/spec-kit), following [Part 1: Driving the Kit Without Leaving the Agent](../../07/24/spec_kit_copilot.html).*

Part 1 was about collapsing a boundary. The [Spec Kit Copilot plugin](https://github.com/github/spec-kit-copilot) lets Copilot drive the `specify` CLI, so installing a preset, adding an extension, or updating the kit no longer requires leaving the agent to manage the agent's own environment. The process and the tooling that shapes the process can live in one conversation.

But bringing the kit *into* Copilot raises the next question almost immediately: does one Copilot instance now have to do every part of that process itself?

The official [`copilot-sub-agents` preset](https://github.com/github/spec-kit-copilot/tree/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents) answers no — selectively. It teaches eight existing Spec Kit commands where independent work can be delegated to sub-agents, where dependencies require waiting, and where the main agent must pull the results back together. Part 1 made Copilot capable of driving the kit. This preset makes Copilot capable of distributing parts of the drive.

That sounds like a performance feature, and it is. What makes it interesting, though, is how carefully it refuses to become a second process.

## Why this is a preset, not an extension

The distinction matters here. In the agent-agnostic series, I described an [extension](../../06/29/spec_kit_extensions.html) as the primitive that adds a capability the core process did not have — a new command, template, script, or even a process alongside SDD. A [preset](../../06/30/spec_kit_presets.html), by contrast, changes how existing capabilities behave.

Sub-agent delegation does not add a new workflow command. The preset's [`preset.yml`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/preset.yml) names eight commands that already exist:

```text
speckit.specify
speckit.clarify
speckit.plan
speckit.tasks
speckit.analyze
speckit.checklist
speckit.implement
speckit.taskstoissues
```

Every entry uses the same composition strategy:

```yaml
replaces: "speckit.plan"
strategy: "prepend"
```

The word `replaces` can look more invasive than the result actually is. It identifies the command being customized; `prepend` determines how that customization composes. The preset contributes a short Copilot-specific delegation section at the top, then the lower-priority command content follows underneath. It does not fork the plan command, reproduce its workflow, or invent a parallel lifecycle that can drift away from the command stack beneath it. That underlying command still says *what* `speckit.plan` does. The preset adds guidance about *how Copilot can execute independent parts of it*.

That is exactly preset-shaped work: same commands, same artifacts, same SDD sequence, different execution behavior. If the design introduced its own command or parallel lifecycle, an extension would be the more natural fit; this preset instead teaches the existing commands when to use a capability Copilot already provides.

## What gets distributed

The preset is not a generic instruction to "use sub-agents whenever possible." Each command overlay names a specific seam where work is independent enough to delegate, and several of those seams happen before implementation.

### Research and design

The [`speckit.plan` overlay](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.plan.md) has the clearest dependency graph.

During Phase 0, each `NEEDS CLARIFICATION` item in Technical Context can become a separate research assignment. Technology choices that need a best-practices review can be researched independently too. Those results come back as decisions, rationale, and alternatives considered, and the main agent consolidates them into `research.md`.

Phase 1 then runs in two waves:

```text
Wave 1: data-model.md ─┐
                       ├─> Wave 2: quickstart.md
Wave 1: contracts/ ────┘
```

The data model and interface contracts can be produced in parallel because neither needs the other's finished artifact. The quickstart cannot: it is explicitly based on the spec, data model, and contracts, so it waits for both Wave 1 assignments to finish.

That little two-wave shape contains most of the preset's philosophy. Parallelize independence; preserve causality.

The [`speckit.tasks` overlay](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.tasks.md) follows the same rule. It first splits document loading into core documents (`spec.md` and `plan.md`) and supporting documents (`data-model.md`, `contracts/`, `research.md`, and `quickstart.md`). Once that context is assembled, independent user stories can each get their own task-generation pass. The main agent still assigns the final sequential task IDs, resolves cross-story dependencies, and assembles `tasks.md`. Delegation produces candidate slices; one place remains responsible for the shared plan.

### Validation passes

Several commands use sub-agents not to create more material, but to challenge what already exists.

- [`speckit.specify`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.specify.md) delegates a specification-quality check after `spec.md` is written. While the main agent prepares the completion report, a sub-agent checks for user value, testability, measurable success criteria, defined scenarios, bounded scope, and accidental implementation detail. Failures must be addressed before the command finishes.
- [`speckit.clarify`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.clarify.md) divides its ambiguity scan into three perspectives: functional and domain concerns; quality and integration concerns; and UX, edge cases, constraints, terminology, and completion signals. Their candidate questions are merged and prioritized back down to the command's final queue of no more than five.
- [`speckit.analyze`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.analyze.md) fans out six detection passes: duplication, ambiguity, underspecification, constitution alignment, coverage gaps, and inconsistency. The main agent assigns severities and merges the findings into one report, capped at fifty.
- [`speckit.checklist`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.checklist.md) delegates two context reads — one focused on the specification, another on the plan and tasks — before generating a domain-specific checklist.

I like this use of sub-agents more than the obvious "write code faster" story. The quality gates in the [core SDD process](../../07/02/spec_kit_process.html) exist because different kinds of scrutiny catch different failures. The preset gives those perspectives isolated contexts, then requires their evidence to be reconciled. It is not a panel of agents voting on whether a spec feels good. It is a division of inspection work with one accountable synthesis.

### Parallel implementation tasks

Implementation is where the preset uses a signal Spec Kit already records. In `tasks.md`, a task marked `[P]` is parallel-safe. The [`speckit.implement` overlay](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.implement.md) turns each such task in the current phase into a separate sub-agent assignment, with relevant spec and plan context plus the target file.

The constraints are more important than the fan-out:

- Only `[P]` tasks in the **same phase** are dispatched together.
- Tasks without `[P]` stay sequential and run in order.
- A parallel task that depends on a sequential task waits for it.
- The main agent verifies every result before moving to the next phase.
- Only after verification does it mark the task `[X]` in `tasks.md`.

So `[P]` is not decoration and "parallel" is not inferred from enthusiasm. The task graph remains the authority. The preset simply turns the graph's explicit independence into actual concurrent work.

### Issue creation

The side command gets the same treatment. [`speckit.taskstoissues`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/commands/speckit.taskstoissues.md) first preserves the core deduplication pass: fetch existing issues and determine which task IDs are already represented. Only unmatched tasks are delegated for issue creation, in batches of at most five to limit rate pressure. The results return to one report with issue numbers and URLs, including tasks skipped because an issue already existed.

Again, the ordering tells the story. Parallel issue creation comes *after* shared-state inspection, not instead of it. Otherwise the preset would make duplicate mistakes faster.

## The Copilot part is environment-specific

The preset's [README](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/README.md) and every overlay distinguish the two Copilot environments rather than pretending "sub-agent" names one universal mechanism.

In **VS Code**, the instruction is concrete: use the `runSubagent` tool, which starts each assignment in an isolated context.

In **Copilot CLI**, delegation uses subsidiary sub-agent processes that the CLI manages. The CLI can also target custom agents defined in `.github/agents/` for the repository or `~/.copilot/agents/` for the user.

Those are different execution surfaces with the same workflow role. The preset does not flatten them into one fictional API. It names the native mechanism in each environment, then gives both the same bounded assignment and return contract.

The isolation this preset asks for is contextual, not filesystem-level. None of its overlays creates or assigns separate Git worktrees, so the delegated implementation it describes operates against the current checkout. That makes [Spec Kit's `[P]` rule](https://github.com/github/spec-kit/blob/d4229c071c7ea3885b43e8a7739847300f618f13/templates/tasks-template.md#L16-L20) — different files and no incomplete dependencies — especially important: work that can collide in one file must remain sequential.

That is another reason the behavior belongs in a Copilot-specific preset rather than Spec Kit core. The underlying opportunities for independence — separate research questions, `[P]` tasks, distinct analysis passes — are properties of the SDD process. The mechanics for dispatching them are properties of the agent. Core can remain agent-agnostic while this layer translates those opportunities into Copilot's vocabulary.

## The boundary is not "delegate everything"

Sub-agents provide focused contexts and can reduce elapsed time. They also create coordination work, and the preset never hides that cost.

First, delegation is selective. `speckit.constitution` and `speckit.converge` are not overlaid. Neither is every action inside the eight commands that are. The preset targets work with a defensible independence boundary and leaves ordering-sensitive work alone.

Second, dependencies remain real. The plan overlay waits to write the quickstart; implementation stays inside phase boundaries; issue creation waits for deduplication. A sub-agent process does not dissolve a dependency merely because it can run at the same time as something else.

Third, the artifacts are shared even when the contexts are not. Research has to become one `research.md`. Per-story task lists have to become one ordered `tasks.md`. Six analysis passes have to become one severity-ranked report. Parallel implementation happens in the shared checkout, so its combined result has to be verified before progress is recorded. The main agent is not just a dispatcher; it is the reconciliation point.

Finally, parallelism is not correctness. Six agents can produce six compatible-looking mistakes. A task marked `[P]` incorrectly can still collide with another task in a shared file. Faster issue creation can still encode a bad task breakdown. The preset improves the execution shape, but the quality gates, dependency model, verification, and human judgment remain the things that make the result trustworthy.

## Installing it

The preset requires Spec Kit 0.8.0 or newer, the release line that supports preset composition strategies, plus GitHub Copilot CLI or VS Code with sub-agent support. Its README recommends registering the repository's preset catalog once, then installing by ID:

```bash
specify preset catalog add \
  https://raw.githubusercontent.com/github/spec-kit-copilot/main/spec-kit-presets/catalog.json \
  --name spec-kit-copilot --install-allowed

specify preset add copilot-sub-agents
```

After that, the commands do not change names. You still run `speckit.plan`, `speckit.tasks`, or `speckit.implement`. For command overlays like these, [Spec Kit composes the delegation guidance](https://github.com/github/spec-kit/blob/d4229c071c7ea3885b43e8a7739847300f618f13/docs/reference/presets.md#file-resolution) ahead of the lower-priority command content when the preset is installed or the command stack is reconciled. In Copilot skills mode, it materializes that result as the command's `SKILL.md` under `.github/skills/`; Copilot reads the generated skill when the command runs rather than asking Spec Kit to resolve the preset stack again. Templates and scripts are different: Spec Kit still looks those up from the stack when it needs them. Installing this preset changes the commands' execution strategy, not the process users have to remember.

## Why I like this design

What I appreciate is that the preset treats parallelism as a property to discover in the work, not a goal to maximize.

There is a louder version of this feature that could have put "spawn agents" at the top of every command and called the result orchestration. This one instead encodes small, reviewable decisions: these three scans are independent; those two artifacts can be written together; this third artifact must wait; these tasks carry `[P]`; those do not; these issue writes can fan out only after deduplication. The useful intelligence is not simply that Copilot can start more agents. It is knowing where doing so is safe.

It also preserves the architecture I liked in Part 1. The plugin remains guidance rather than a second agent, Spec Kit remains the real process, and the preset remains a thin layer that composes with core instead of copying it. Part 1 removed the tab-switch between managing the kit and using it. Part 2 removes a different bottleneck: the assumption that using it means one agent must hold every independent concern in one context, one after another.

The result is not less disciplined SDD. If anything, it makes the discipline more visible. Dependencies have to be named before work can run concurrently. Return shapes have to be explicit before results can be merged. Shared artifacts have to be treated as contracts rather than scratch space. Sub-agents do not replace the process; the process is what tells them where they can safely help.

There is a natural next question. Once Copilot can manage the kit and distribute the work, how do you make the state of that work visible — the current SDD stage, the artifacts behind it, and the next action — without reducing it to another wall of terminal output? That is where I will go next, with the SDD Canvas.

If you try the preset, I would like to hear which boundaries actually saved time and which ones merely moved the coordination cost around. Send an email to blog (at) manorrock.com.

---

*Further reading: [Part 1: Driving the Kit Without Leaving the Agent](../../07/24/spec_kit_copilot.html), the [`copilot-sub-agents` preset README](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/README.md), its [`preset.yml`](https://github.com/github/spec-kit-copilot/blob/e6f632f9395da94b03b6319341b332e2b09eb108/spec-kit-presets/copilot-sub-agents/preset.yml), [Spec Kit, Part 3: Presets](../../06/30/spec_kit_presets.html), and [Spec Kit, Part 5: The Spec-Driven Process](../../07/02/spec_kit_process.html).*
