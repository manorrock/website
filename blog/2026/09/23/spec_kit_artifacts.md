# Spec Kit, Part 14: Artifacts — Seeing What the Layers Actually Built

*The fourteenth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

I ended [Part 12](../../07/14/spec_kit_road_ahead.html) arguing that the next important work was not another primitive. It was coherence: making the five already there feel like one system, and making the trust boundary more legible through clearer provenance and better ways to see what an installed component actually contributes. [Part 13](../21/spec_kit_events.html) followed the new agent-event machinery itself. This post takes the complementary view: not what one new mechanism does, but how Spec Kit now lets us inspect the commands, templates, scripts, and extension hooks its existing layers expose. Two months after Part 12, Spec Kit 1.0.7 shipped almost exactly that kind of surface.

The command is `specify artifact`, and its job sounds modest: list the commands, templates, scripts, and hooks available in a project; explain how one of them was composed; trace a contribution back to the manifest that declared it. It does not install anything. It does not change the resolution order. It does not create a sixth kind of package. It looks at the process Spec Kit has already assembled and tells you, in machine-readable detail, what is really there.

That distinction matters enough to put first. Upstream still names **five composable primitives**: integrations, extensions, presets, workflows, and workflow steps. Those are the things you select, install, author, distribute, and compose. An **artifact** is different. It is the resolved command, template, script, or hook that those layers expose to the project. The primitives are the machinery; artifacts are the inspectable result.

Or, in the language this series has used from the beginning: the kit has finally grown a glass side.

## The question the old commands could not quite answer

Before this surface, you could inspect the ingredients independently. `specify extension list` could tell you which extensions were installed. `specify preset list` could tell you which presets were active. The workflow and workflow-step commands could show their own component types. But that did not answer the question you eventually care about when several layers touch the same thing:

> What will `speckit.specify` actually be in this project, and why?

That command name might begin with the built-in command, be wrapped by one preset, have content prepended by another, and then be replaced by a project-local override. A template or script can have the same layered history. Knowing that four packages are present is not the same as knowing which contribution won, which ones still participate in composition, and which ones have been cut out of the resolved result.

That gap first became concrete in the [Spec Kit Copilot sibling project](https://github.com/github/spec-kit-copilot), where the Wizard had to parse text output and reimplement composition logic to render its inspector. The [feature request that became `specify artifact`](https://github.com/github/spec-kit/issues/4212) brought that need back into core, but the result stayed true to Spec Kit's principles: a general, read-only, agent-agnostic JSON surface rather than Copilot-specific logic. The downstream Wizard got simpler, and any future consumer can now ask core the same question.

`specify artifact` answers at the level where the process runs. Its catalog has four fixed artifact kinds:

| Kind | What is being inspected |
| --- | --- |
| `command` | An agent-facing command such as `speckit.specify` |
| `template` | A template resolved into the project's process |
| `script` | A script available to that process |
| `hook` | An extension hook declaration: a hook event paired with the target command it invokes |

Those artifacts may come from bundled core assets, project overrides, presets, or extensions. The surface puts them in one vocabulary even though the contributing layers remain different. That is the useful abstraction: not "which packages did I install?" but "which runnable and renderable things did the assembled system produce?"

Artifact introspection landed in [Spec Kit 1.0.7](https://github.com/github/spec-kit/releases/tag/v1.0.7) on September 15, 2026. The base `list` and `info` surface, hook contributions, and contribution lookup arrived together in that release. As I write this, 1.0.8 is already current; 1.0.7 is the landing release, not a claim that the project stopped moving.

## Start wide: `artifact list`

The first view is deliberately broad:

```bash
specify artifact list --json
```

The `--json` is not an optional formatting preference. All three artifact subcommands currently require it. The interface is a read-only machine surface first, which makes sense: the immediate consumers are agents, diagnostics, editor integrations, and tooling that need a stable answer rather than a table shaped for a terminal.

The result is an array of artifacts. A normal entry has an ID, a name, a kind, a description, and its complete composition stack:

```json
{
  "id": "command:speckit.specify",
  "name": "speckit.specify",
  "kind": "command",
  "description": "Create or update the feature specification.",
  "stack": [
    {
      "id": "command:speckit.specify",
      "layer": "preset",
      "sourceId": "compliance",
      "presetId": "compliance",
      "presetName": "Compliance Preset",
      "strategy": "wrap",
      "active": true,
      "hidden": false,
      "manifestPath": ".specify/presets/compliance/preset.yml",
      "lookupId": "preset:compliance:command:speckit.specify",
      "sourcePath": ".github/skills/speckit-specify/SKILL.md"
    },
    {
      "id": "command:speckit.specify",
      "layer": null,
      "sourceId": null,
      "presetId": null,
      "presetName": null,
      "strategy": "replace",
      "active": false,
      "hidden": false,
      "manifestPath": null,
      "lookupId": null,
      "sourcePath": null
    }
  ]
}
```

The exact paths and contributors will differ, of course, but the shape is the important part. `list` is not merely an inventory of names. Each row carries the stack that explains the name. The description comes from the highest-precedence contribution that supplies one, and named artifacts are sorted predictably by kind and name. Hook rows add their event name, target command, and whether the hook is registered.

This is already enough to answer several debugging questions without opening a manifest: does this command exist, is the template coming from the layer I expected, did the project override take effect, and is the hook I installed actually registered?

## Then go deep: `artifact info`

For one artifact, ask directly:

```bash
specify artifact info speckit.specify --json
```

A bare name works when it is unambiguous. You can also use the full artifact ID:

```bash
specify artifact info command:speckit.specify --json
```

Or constrain a bare name with `--kind command`, `--kind template`, `--kind script`, or `--kind hook`. A kind prefix and `--kind` must agree; Spec Kit does not quietly guess past a contradiction.

The response repeats the artifact's identity and description, then returns the full ordered stack. Order is precedence: index zero is the selected contribution. For commands, templates, and scripts, only that first row has `"active": true`.

The subtle and useful part is that **inactive does not mean hidden**.

If a higher layer uses `wrap`, `prepend`, or `append`, the lower layers remain part of the composed artifact. They are inactive because they are not the selected top row, but they are not hidden because their content still participates in what the project ultimately sees. A `replace` strategy changes that. The first replacement cuts off the rows below it, and those lower contributions become hidden.

That gives the two flags distinct jobs:

- `active` answers which contribution is selected at the head of the stack.
- `hidden` answers whether a replacement prevents a lower contribution from participating at all.

Without both, introspection would collapse two very different states into one. A wrapped built-in command and a replaced built-in command are both not the winner, but only one still matters to the result.

The rest of each stack row records provenance: layer, source ID, preset identity where relevant, composition strategy, manifest path, source path, and a lookup ID for non-core layers. A lookup ID identifies stack provenance, but only IDs for manifest-backed preset or extension declarations can be dereferenced with `artifact lookup`. Core assets are intentionally represented with null provenance fields rather than pretending there is an installable `core` layer. Project overrides use `layer: "project"` and the synthetic source ID `_`. Presets and extensions retain the identities and paths that let you walk back to what declared them.

This is the trust-boundary work I wanted in Part 12 made concrete. "Read the source" is better advice when the tool can tell you which source to read.

## Follow the provenance: `artifact lookup`

The final command takes the `lookupId` from a stack entry:

```bash
specify artifact lookup preset:compliance:command:speckit.specify --json
```

It returns the owning layer, source ID, artifact kind and name, the manifest path, the source path, and the contribution declaration:

```json
{
  "id": "preset:compliance:command:speckit.specify",
  "layer": "preset",
  "sourceId": "compliance",
  "kind": "command",
  "name": "speckit.specify",
  "manifestPath": ".specify/presets/compliance/preset.yml",
  "sourcePath": ".specify/presets/compliance/commands/speckit.specify.md",
  "contribution": {
    "type": "command",
    "name": "speckit.specify",
    "file": "commands/speckit.specify.md",
    "strategy": "wrap"
  }
}
```

This is not a raw YAML echo. The `contribution` is the validated, normalized declaration Spec Kit actually uses after parsing the manifest—canonical command names, normalized strategy values, defaults filled in. That makes lookup more useful than telling an agent to scrape a file and reconstruct the resolver's interpretation itself.

There is a precise limit to the command. Lookup works only for manifest-backed preset and extension contributions. Built-in rows have no `lookupId`. Project overrides and convention-based preset or extension contributions can still carry deterministic lookup IDs, but passing those IDs to `artifact lookup` returns `unknown contribution` because there is no originating manifest declaration. A non-null `lookupId` therefore identifies stack provenance; it does not by itself guarantee that lookup will succeed.

The three commands form a clean narrowing path:

```text
artifact list
    │
    │ choose an artifact
    ▼
artifact info <name>
    │
    │ choose a manifest-backed stack entry
    ▼
artifact lookup <lookupId>
```

Inventory, resolution, declaration. Each step preserves the identity you need for the next one.

## Hooks bend the stack model on purpose

Part 13 distinguishes the newer agent lifecycle events from the extension hooks that run around Spec Kit commands. Artifact introspection currently exposes the latter: extension hook declarations grouped by their hook event and target command. For that model, one difference matters: hooks are **additive**.

A command, template, or script resolves to one active head, with lower contributions composed or hidden according to strategy. A hook does not have a winner in that sense. Several extensions can declare the same hook event and target-command pair, and several of those declarations can be active together. The hook stack is ordered by normalized priority and then deterministic declaration order, but ordering does not turn the first one into a replacement for the rest.

That is why hook entries report:

- `strategy: "additive"`
- `hidden: false`
- their own `priority` and `optional` values
- `active` for whether `.specify/extensions.yml` contains an enabled runtime binding for the same event and target command, keyed by the extension ID declared in the manifest; that runtime ID may differ from the installed `sourceId` used in the artifact's `lookupId`

The top-level hook artifact reports `registered: true` when any stack entry is active. One more precision point: `active` describes registration, not whether a hook's later runtime condition will evaluate to true on a particular invocation. Introspection tells you what is wired; execution still decides what fires.

This is a small design choice with a large payoff. Spec Kit did not force hooks into the winner-and-shadow vocabulary just to make every JSON object look alike. It kept one top-level artifact model while preserving the semantics of the thing being described. Named artifacts compose by precedence. Hooks accumulate by priority.

## Not a sixth primitive, and not another catalog

This is the category error I expect people to make, because I nearly made it myself: if artifacts have a top-level command and first-class IDs, are they the next primitive after workflow steps?

No.

Integrations, extensions, presets, workflows, and workflow steps remain the project's five composable primitives, each with its own authoring and distribution lifecycle. Their installation and composition mechanics are not identical: bundles install extensions, presets, workflows, and workflow steps while optionally targeting a selected integration.

There is no corresponding:

```text
specify artifact add
specify artifact remove
specify artifact search
```

There should not be. You do not install a command artifact through the artifact surface; it is exposed by core, a project override, a preset, or an extension after those layers have been resolved. Artifact IDs such as `command:speckit.specify` name resolved artifacts; artifact-owned `lookupId` values identify specific non-core contributions within their stacks. They are not a separate package or manifest contribution-ID system. The surface is top-level because the question is top-level, not because a new package ecosystem appeared.

That separation is also why the command belongs beside `extension`, `preset`, and `workflow` rather than underneath any one of them. A command can be touched by more than one primitive and by the project itself. Putting introspection under `preset` would make extensions look secondary; putting it under `extension` would hide presets; putting it under the integration would confuse translation with ownership. `artifact` is neutral ground over the assembled system.

## Why I like this design

I like this feature because it makes composition accountable without changing composition.

For most of this series, I have praised Spec Kit's instinct to compose rather than accrete. Five primitives share catalog machinery; workflows reuse commands; integrations translate rather than fork the process; presets and extensions layer rather than demand a new core feature. But composition has a cost: once several layers participate, the user needs a way to see the answer. Otherwise "composable" becomes another word for "mysterious."

`specify artifact` pays that cost. It says the stack order out loud. It distinguishes selected from still-participating from hidden. It carries provenance from the final artifact back to the normalized manifest declaration. It lets hooks remain additive rather than lying that one won. And it does all of that through a read-only JSON contract that another tool can consume without scraping terminal prose.

There is restraint in what it does not become. No sixth primitive. No artifact catalog. No install lifecycle. No mutation command tucked into an introspection namespace. The feature looks across the whole architecture without trying to own the architecture, which is exactly the coherence work I hoped the project would choose after Part 12.

The most important output may not be a particular JSON field. It may be the change in the question we can ask. Before, when a generated command surprised you, the practical question was "which package broke this?"—a hunt through installed components, overrides, and assumptions. Now the question can be "show me the stack." That is calmer, fairer, and much more answerable. The tool can tell you not only what won, but what remained underneath and where every inspectable contribution came from.

The kit has always been made of layers. With 1.0.7, the layers stopped being something you had to infer.

If you use artifact introspection to untangle a surprising stack—or build an editor, diagnostic, or agent experience on top of the JSON surface—send an email to blog (at) manorrock.com. I would like to see what becomes possible once the process can explain how it became itself.

---

*Further reading: the [Artifacts reference](https://github.com/github/spec-kit/blob/main/docs/reference/artifacts.md), the [artifact-introspection feature request](https://github.com/github/spec-kit/issues/4212), the [Spec Kit Copilot plugin](https://github.com/github/spec-kit-copilot), the [Spec Kit 1.0.7 release](https://github.com/github/spec-kit/releases/tag/v1.0.7), the [Spec Kit history of the five primitives](https://github.com/github/spec-kit/blob/main/docs/history.md), [Part 13: Events](../21/spec_kit_events.html), [Part 12: The Road Ahead](../../07/14/spec_kit_road_ahead.html), [Part 10: Steps](../../07/10/spec_kit_steps.html), [Part 9: Workflows](../../07/09/spec_kit_workflows.html), [Part 4: Integrations](../../07/01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), and [Part 2: Extensions](../../06/29/spec_kit_extensions.html).*
