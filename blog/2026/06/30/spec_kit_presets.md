# Spec Kit, Part 3: Presets — Customizing the Process Without Touching the Tooling

*The third entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

In [Part 2](../29/spec_kit_extensions.html) I covered extensions, the primitive that *adds* new capabilities — new commands, templates, and scripts that go beyond the built-in workflow. This time I want to look at the primitive that sits right next to it and does almost the opposite. Where an extension adds things you can do, a **preset** changes how the things you already do behave. The two are complementary, and seeing them side by side is the cleanest way to understand either one.

That pairing is why presets come now. We are still working down the stack from where the series started, and presets are the layer that customizes the core workflow itself — the templates it fills in, the commands it runs, even the words it uses. Introduce extensions first and presets have a natural foil; introduce presets here and the resolution model that ties the whole stack together finally comes into view.

## What a preset is for

Core Spec Kit ships a particular way of working: a set of templates, commands, and terminology that drive the specify-plan-build loop. That default is deliberately neutral. But teams are not neutral — they have house styles, compliance requirements, a methodology they have standardized on, sometimes a language other than English they want the whole experience in.

A **preset** is how you bend the process to fit without forking anything. Presets reshape templates, commands, and terminology — enforcing organizational standards, adapting the workflow to your methodology, or localizing the entire experience — and they do it *without changing any tooling*. The CLI, the extensions, the core loop all stay exactly as they are. A preset just contributes its own version of a file wherever it provides one, and — this is the part "override" undersells — it gets to choose *how* its version combines with what is underneath: replace it outright, prepend to it, append to it, or wrap around it.

That is the distinction I want to land: an extension is additive, a preset is transformative. An extension gives you a command that did not exist. A preset takes the `plan-template.md` the core already uses and reshapes it to match your standards — swapping it wholesale, or layering your content onto it. Nothing new appears in your command list — the existing process simply produces output shaped the way you want.

And here is the part that is easy to miss but is really the whole point: a preset does not only reshape the *core*. Because presets sit above extensions in the resolution stack (more on that below), a preset can override an installed extension's templates and commands just as readily as it overrides the core's. So a preset is the one layer that can reach across everything beneath it — core *and* any extension you have installed — and bring it all in line with your standards. That reach is what makes presets the right tool for "make our whole setup, whatever it is built from, behave our way."

A preset is described by a `preset.yml` manifest. Here is a real one — the `lean` preset that ships with Spec Kit, which trims the workflow commands down to their essentials:

```yaml
schema_version: "1.0"

preset:
  id: "lean"
  name: "Lean Workflow"
  version: "1.0.0"
  description: "Minimal core workflow commands - just the
    prompt, just the artifact"
  author: "github"
  license: "MIT"

requires:
  speckit_version: ">=0.6.0"

provides:
  templates:
    - type: "command"
      name: "speckit.specify"
      file: "commands/speckit.specify.md"
      description: "Lean specify - create spec.md from a description"
      replaces: "speckit.specify"
    - type: "command"
      name: "speckit.plan"
      file: "commands/speckit.plan.md"
      description: "Lean plan - create plan.md from the spec"
      replaces: "speckit.plan"
    # ...and the same for tasks, implement, and constitution

tags:
  - "lean"
  - "minimal"
  - "workflow"
```

Read the `provides` block and the whole idea is right there. Each entry names a command the core already has, points at the preset's own file for it, and declares — via `replaces` — exactly which core command it stands in for. The `lean` preset adds nothing to your command list; it substitutes leaner versions of commands you already had. That is a preset in one manifest: same names, different content.

## Installing one

Presets install through the same catalog machinery as every other primitive:

```bash
specify preset search compliance
specify preset add <preset_id>
```

As with extensions, a preset's commands are automatically registered with whichever AI coding agent integration you have set up — install it and the customization takes effect where you already work. (Same prerequisite, too: preset commands need a project that has been initialized with `specify init` first.)

The rest of the lifecycle will feel familiar, because by design it mirrors the other primitives:

- `specify preset list` — what is installed, with versions, descriptions, template counts, and status.
- `specify preset info <preset_id>` — templates, metadata, and tags for one preset.
- `specify preset enable <preset_id>` / `disable <preset_id>` — exclude a preset from file resolution without uninstalling it; handy for comparing template output with and without it.
- `specify preset remove <preset_id>` — uninstall it and unregister its commands.

There is one extra command here that does not have an exact analog elsewhere, and it is the most useful one of the bunch:

```bash
specify preset resolve plan-template.md
```

`resolve` traces the full resolution stack and tells you exactly which file will be used for a given name. Once you have more than one preset installed, this is how you answer "where is this template actually coming from?" without guessing.

## Priority and stacking

Presets are meant to be combined, and the rule that makes combining them sane is the same one extensions use: when two presets provide a file with the same name, the **lower priority number wins**.

```bash
specify preset add compliance --priority 5
specify preset add team-workflow --priority 10
```

For any file both provide, `compliance` wins because 5 is lower than 10. For a file only one provides, that one is used. For a file neither touches, the core default stands. You change the order after the fact with `specify preset set-priority <preset_id> <N>`, so reordering never means reinstalling.

The important nuance is that priority is evaluated **per file**, not per preset. Each file name is resolved independently against the stack, so a high-priority preset that only overrides one template does not shadow everything a lower-priority preset provides — it wins for that one file and stays out of the way for the rest.

## Where presets sit in the resolution stack

This is the part that ties the whole series together. Every templated file in Spec Kit — templates, commands, scripts — is resolved against one ordered stack, highest precedence first:

1. **Project-local overrides** — `.specify/templates/overrides/`
2. **Installed presets** — sorted by priority
3. **Installed extensions** — sorted by priority
4. **Spec Kit core** — `.specify/templates/`

Read that top to bottom and the relationship between the last two posts snaps into focus. Extensions sit *below* presets — which is precisely why a preset can override an extension and not the other way around. An extension can ship a template, and a preset can override it; a project-local file in `overrides/` beats them both. This ordering is the mechanism behind the point I made up front: a preset's reach extends over the core *and* every installed extension, because everything an extension provides sits lower in the same stack. It is the same layering philosophy I keep coming back to — lower layers are defaults, higher layers win — applied to file resolution instead of configuration.

And presets are not limited to wholesale replacement — the *strategy* is part of what a preset declares, and which strategies are available depends on the file type. By default a file uses **replace**: the first match in the stack wins and is used entirely. Templates and commands get the full set — **prepend** puts preset content before the lower-priority content, **append** puts it after, and **wrap** drops the lower-priority content in wherever the preset writes `{CORE_TEMPLATE}`. Scripts are deliberately more limited: they support only **replace** and **wrap**, the latter using `$CORE_SCRIPT` as the placeholder. The upshot is that a preset can *extend* what is beneath it rather than discard it — append a compliance section to every spec, say, while still inheriting whatever the core (or a lower preset, or an extension) provides underneath.

## Catalogs and trust

Presets are discovered through the same priority-ordered catalog stack as everything else — environment variable, then project config, then user config, then the built-in official and community catalogs. An organization can publish its own catalog of approved presets at a high priority:

```yaml
catalogs:
  - name: "my-org-presets"
    url: "https://example.com/preset-catalog.json"
    priority: 5
    install_allowed: true
    description: "Our approved presets"
```

The same honesty applies as with the rest of the ecosystem: most presets are independently created and maintained by their authors, and the Spec Kit maintainers do not review, audit, or endorse preset code. A preset rewrites the templates and commands that drive your workflow, so read the source before you install — the `verified` versus `community` signal tells you what has been vetted and what has not.

## Why I like this design

What I appreciate about presets is that they make "customize the process" a first-class, reversible act instead of a fork. The temptation in any tool is to let teams hand-edit the core templates until every checkout has drifted somewhere different. Presets give that impulse a clean home: your customization is an installed, versioned, priority-ordered unit that resolves against a stack you can inspect with `resolve`, and removes without a trace. The core stays pristine; your house style lives in a layer on top of it.

Seen together, extensions and presets divide the work neatly. Extensions add what the core lacks; presets reshape what already exists — the core *and* any extension on top of it — and a single resolution stack decides, file by file, who wins. That shared stack is what lets all these primitives coexist without stepping on each other, and what gives a preset the reach to bring an entire setup in line no matter which pieces it is built from — which is exactly what a bundle relies on when it composes them.

Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Presets reference](https://github.com/github/spec-kit/blob/main/docs/reference/presets.md), [Part 2: Extensions](../29/spec_kit_extensions.html), and [Part 1: Bundles](../26/spec_kit_bundles.html).*
