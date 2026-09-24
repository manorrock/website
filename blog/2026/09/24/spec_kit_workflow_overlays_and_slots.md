# Spec Kit, Part 15: Workflow Overlays and Slots — Editing the Graph Without Forking It

*The fifteenth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

[Part 13](../21/spec_kit_events.html) opened the series again with agent events, and [Part 14](../23/spec_kit_artifacts.html) followed the outputs a process leaves behind. This time I want to return to the process itself — specifically, to a gap in [Part 9's workflow story](../../07/09/spec_kit_workflows.html). That July post described a workflow as a reusable graph of steps and treated the installed `workflow.yml` as the graph you run. At the time that was the whole public shape. Less than two weeks later, a community contribution added Spec Kit's first workflow overlay resolver, giving a project a layer that can change that graph without changing the installed file. In September, Spec Kit 1.0.5 added the matching author-side idea: a workflow can now declare a place where it expects to be changed.

Those two ideas are **workflow overlays** and **workflow slots**. An overlay is a project-local edit layer over an installed workflow. A slot is a workflow-declared, named no-op step intended to be filled by one of those layers. Together they answer a practical question that Part 9 left open: what happens when a shared workflow is almost right, but your project needs one more check, one fewer gate, or a project-specific step in a place the workflow author anticipated?

The answer is not "copy the YAML and own a fork." It is "describe the difference."

## The upgrade problem hiding inside a good workflow

Suppose your team installs a workflow from a catalog. It does exactly what you want except that, after implementation, your project must run a license scan. The obvious edit is easy:

```yaml
- id: license-scan
  type: shell
  run: "scripts/check-licenses.sh"
```

The hard part is where to put it. If you insert that step directly into `.specify/workflows/<workflow-id>/workflow.yml`, your local copy is no longer just the installed workflow. The next `specify workflow update`, `specify workflow add`, or bundle refresh has to choose between the base file and your edit. Even when the update succeeds, you have turned every upgrade into a small merge exercise.

A workflow overlay moves the change outside the installed package:

```text
.specify/workflows/
├── delivery/
│   └── workflow.yml
└── overlays/
    └── delivery/
        └── project-checks.yml
```

The installed workflow stays under `.specify/workflows/delivery/`. The project's edit lives under `.specify/workflows/overlays/delivery/`. When you run the installed workflow by ID, the resolver loads the base, collects every enabled overlay for that ID, composes a new step list in memory, and validates the result as a normal workflow definition.

That separation is what makes the customization upgrade-safe. Reinstalling or refreshing `delivery` replaces the installed package, not the sibling `overlays/delivery/` directory. Your project keeps its delta; the workflow keeps receiving Spec Kit updates.

## An overlay is a list of edits, not another workflow

An overlay does not repeat the workflow definition. It names the workflow it extends and declares edits against step IDs in the base graph:

```yaml
id: project-checks
extends: delivery
priority: 10
enabled: true
edits:
  - insert_after: implement
    step:
      id: license-scan
      type: shell
      run: "scripts/check-licenses.sh"

  - remove: optional-demo
```

The edit vocabulary is deliberately small:

| Operation | What it does |
| --- | --- |
| `insert_before` | Adds a step immediately before the anchor. |
| `insert_after` | Adds a step immediately after the anchor. |
| `replace` | Substitutes a new step for the anchor. |
| `remove` | Deletes the anchor from the composed step list. |

The key word is **anchor**. Each edit targets the `id` of a step in the *base* workflow. The explicit form (`operation: insert_after` plus `anchor: implement`) is supported too, but I prefer the shorthand above because the edit reads almost like a sentence.

The resolver searches recursively, so an anchor does not have to be a top-level step. It can live inside `then`, `else`, `steps`, `default`, or any branch under `cases.*`. That makes overlays useful on real graphs rather than only on linear pipelines: you can replace a review gate inside one side of an `if`, or insert a notification into one case of a `switch`.

There are boundaries, and they matter:

- Edits apply only to the workflow's step list. They cannot rewrite its name, description, `requires`, inputs, or input schema, or patch expression logic in place; changing a step's behavior means replacing that step.
- Every anchor is resolved against the original base graph. One overlay cannot add a step and give another overlay a new anchor to target.
- A `fan-out` step's template is not an anchorable node. It is multiplied at runtime, so there is no single stable step for the overlay to edit.
- If one edit removes or replaces a composite step while another targets one of its descendants, resolution rejects the ancestor/descendant conflict rather than making the result depend on traversal order.
- New step IDs cannot contain `:`, because the workflow engine reserves that character for generated nested IDs.

Those constraints are not accidental missing features. They keep composition deterministic. An overlay is a patch against a known base shape, not a second program that mutates whatever the previous patch happened to produce.

## Priority: lower numbers still win

If several overlays extend the same workflow, Spec Kit uses the same lower-number-means-higher-priority convention that appears elsewhere in the toolkit:

```yaml
id: organization-policy
extends: delivery
priority: 5
edits:
  - replace: review-release
    step:
      id: review-release
      type: gate
      message: "Security and release engineering must approve."
      options: [approve, reject]
      on_reject: abort
```

An overlay at priority `5` wins a conflicting edit over one at priority `10`. Internally, the composer applies larger numbers first and smaller numbers last, so the higher-precedence layer gets the final say. Equal-priority overlays are ordered alphabetically by overlay ID; when edits conflict, the alphabetically last ID wins.

That last sentence is a useful rule, not a recommendation to build a naming scheme around tie-breaking. Set distinct priorities when precedence expresses policy. If two overlays merely insert compatible steps around the same anchor, both insertions can be composed; if they disagree about whether the anchor should remain, be replaced, or disappear, make the intended winner obvious with priority.

And inspect the answer rather than reasoning from filenames:

```bash
specify workflow resolve delivery
```

`resolve` prints the base-plus-overlay layer stack in precedence order and attributes every resulting step to `base` or the overlay that contributed it. It is the workflow equivalent of asking, "What will actually run?"

## The overlay lifecycle today

You add an overlay from a YAML file:

```bash
specify workflow overlay add project-checks.yml --priority 10
```

Spec Kit validates it and copies it into the project-local overlay directory for the workflow named by `extends`. From there the management surface is small and reversible:

```bash
specify workflow overlay list delivery
specify workflow overlay set-priority delivery project-checks 5
specify workflow overlay disable delivery project-checks
specify workflow overlay enable delivery project-checks
specify workflow overlay remove delivery project-checks
```

Disabling leaves the file installed but removes it from resolution. Removing deletes the project overlay file. Listing includes disabled overlays and shows the order the resolver will use.

That is the command surface Spec Kit exposes today. The base workflow can be discovered and installed from a workflow catalog — or added from another supported source such as a URL, local file, package directory, or archive. The overlay is the project-local part: `workflow overlay add` takes a YAML file and layers it over that installed workflow regardless of where the workflow came from. There is currently no separate `workflow overlay search` or `workflow overlay catalog` command. That describes the current implementation, not a claim about every way overlays could ever be distributed.

## Slots let workflow authors say, "Customize here"

Overlays can target any stable base step ID, but sometimes a workflow author knows in advance that projects will need a hook point. That is what `type: slot` is for:

```yaml
steps:
  - id: implement
    type: command
    command: speckit.implement

  - id: post-implement
    type: slot
    name: "Post-implementation checks"

  - id: release-review
    type: gate
    message: "Approve the release?"
    options: [approve, reject]
```

The `id` is the unique overlay anchor. The required `name` is a human-readable label. If no overlay fills the slot, the workflow does not stop or fail: the slot completes with status `skipped`, records `output: {slot: <name>}`, and execution continues to `release-review`.

That makes a slot a real step in the graph but an intentional no-op in the run. The workflow author is saying, "There is a stable extension point here; projects that do not need it pay no behavioral cost."

The normal way to fill it is an overlay `replace`:

```yaml
id: fill-post-implement
extends: delivery
edits:
  - replace: post-implement
    step:
      id: post-implement
      type: shell
      run: "scripts/check-licenses.sh"
```

Reusing the slot's `id` is more than tidiness. Later expressions or a `fan-in.wait_for` may refer to that ID, so changing it can break the graph. The replacement also has to preserve any output keys later steps consume. An unfilled slot guarantees only `steps.post-implement.output.slot`; if later logic expects more, the replacement has to produce it.

Slots share the fan-out limitation too: they are not supported inside a `fan-out.step` template because an overlay cannot address the runtime-multiplied copies. Put the slot before the fan-out, after its fan-in, or inside a statically declared branch the resolver can reach.

This is a much cleaner contract than telling project teams to "insert something after step seven." The slot gives the extension point a stable ID, a name, and a precise location in the authored graph. The overlay supplies the project's behavior. One side declares the seam; the other side uses it.

## Presets for the graph — but not presets

The closest analogy in the earlier series is [Part 3's presets](../../06/30/spec_kit_presets.html). A preset lets you customize command and template behavior without editing the underlying files. An overlay lets you customize workflow composition without editing the underlying `workflow.yml`. Both preserve the installed base and express project intent as a layer on top. Both use explicit priority so composition is inspectable rather than accidental.

But the analogy has a stopping point.

A preset is an installable primitive with catalog discovery and a broad role in file resolution. It can reshape command and template content contributed by the core or extensions. In the current implementation, an overlay is a project-local set of step-list edits for one installed workflow, while a slot is a built-in workflow step type that marks an extension point. Neither is registered as another component kind beside bundles, extensions, presets, integrations, workflows, or custom step types.

Their role today is **workflow composition**. The `WorkflowResolver` composes the installed workflow with its enabled overlay files, and `type: slot` gives that resolver a stable, explicitly named anchor that is harmless when no overlay replaces it. That is the mechanism the code implements; it does not establish that a broader resolver architecture or a future distribution model would be unnecessary.

## The chronology says something about the design

When I published the original workflows article on July 9, overlays were not part of the story. Community contributor [Markus Wondrak](https://github.com/markuswondrak) brought the problem, the proposal, and the implementation. I worked with Markus through [the design discussion](https://github.com/github/spec-kit/discussions/3473) and review cycles, iterating with him on the shape and language of the features as they developed. The resolver and project overlay implementation [merged on July 21](https://github.com/github/spec-kit/pull/3557). Workflow slots [followed on September 3](https://github.com/github/spec-kit/pull/4352) and shipped in [Spec Kit 1.0.5](https://github.com/github/spec-kit/releases/tag/v1.0.5).

I like that sequence because it shows how collaboration and review can sharpen a community contribution. First came runnable workflow graphs. Then came project-local edit layers once real projects needed to change installed graphs safely. Then came slots once workflow authors had a reason to advertise stable places for those changes.

It is tempting to design all three at once and call the architecture complete. In practice, the useful extension point is easier to see after someone has tried to extend the thing. Overlays established the general mechanism; slots turned recurring overlay targets into an explicit contract.

## Why I like this design

The phrase I keep returning to in this series is "compose, don't accrete," and overlays may be its most literal expression yet. A project can keep its customization separate from the installed workflow and compose the two at resolution time. The resulting graph is still validated as one workflow, and `resolve` will show you where every step came from.

Slots add a little humility on the workflow-authoring side. An author does not have to predict the project-specific behavior — only the place where that behavior belongs. An unfilled slot is harmless. A filled slot is explicit. The contract is small enough to understand by reading two YAML files, and strict enough that upgrades do not quietly retarget an edit at whatever happens to occupy the same position.

That feels like the right follow-up to both workflows and artifacts. A workflow is not only the graph its author published, and an artifact is not the only thing a project owns. The project also owns the *difference* between the shared process and the process it actually needs. Overlays give that difference a file. Slots give it a name.

And that may be the most upgrade-safe customization model there is: do not own the whole thing when all you mean to own is the change.

---

*Further reading: the [Spec Kit Workflows reference](https://github.com/github/spec-kit/blob/main/docs/reference/workflows.md), Spec Kit's [overlay composer](https://github.com/github/spec-kit/blob/main/src/specify_cli/workflows/overlay/composer.py) and [merge implementation](https://github.com/github/spec-kit/blob/main/src/specify_cli/workflows/overlay/merge.py), the [`slot` step implementation](https://github.com/github/spec-kit/blob/main/src/specify_cli/workflows/step/slot/__init__.py), [Part 14: Artifacts](../23/spec_kit_artifacts.html), [Part 13: Events](../21/spec_kit_events.html), [Part 10: Steps](../../07/10/spec_kit_steps.html), [Part 9: Workflows](../../07/09/spec_kit_workflows.html), and [Part 3: Presets](../../06/30/spec_kit_presets.html).*
