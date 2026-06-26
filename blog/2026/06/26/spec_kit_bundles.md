# Spec Kit, Part 1: Bundles — A Whole Process in One Command

*The first entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

*Disclosure: I am currently the lead maintainer of Spec Kit, and bundles are a capability we landed recently — so consider this both a tour and a behind-the-scenes note.*

I have been spending a lot of time with [Spec Kit](https://github.com/github/spec-kit) lately, and there is enough going on under the hood that a single post would not do it justice. So I am starting a series. The plan is loose on purpose: each entry picks one capability, explains what it is for, and shows it in action. No grand narrative arc — just a guided tour, one feature at a time.

For the opening entry I want to talk about **bundles**, because they are a great example of how Spec Kit thinks about composition rather than monoliths. Starting the series at the top of the stack is a deliberate choice: rather than build it up from first principles, I am leading with what is freshest and what best captures the design philosophy. We will circle back to the foundational pieces — init, specs, the core spec-driven loop — in later entries.

## The problem bundles solve

Spec Kit is built out of small, composable pieces. There are **extensions** (which add tools and commands), **presets** (which customize how the process behaves by overriding templates, commands, and even terminology, stacked with priority ordering), **integrations** (which wire Spec Kit to a particular coding agent), **workflows**, and **steps**. Each of these is a primitive you can install on its own, and that modularity is genuinely useful — you only pull in what you need.

The flip side of modularity is assembly. If a particular way of working — say, an in-place technology migration — needs a specific extension *and* a matching preset *and* the right priority and strategy wiring between them, then onboarding a teammate means a sequence of install commands in the right order, with the right versions. Miss one and the process is subtly broken.

That is the gap bundles fill.

## What a bundle actually is

A **bundle** composes existing components — extensions, presets, workflows, and steps — into a single, versioned, installable unit. The key thing to understand is what a bundle is *not*: it adds no new runtime behavior of its own. Where extensions and presets are primitives, a bundle is a curated stack that declares everything a team or role needs and installs it in one step **through each component's own machinery**.

In other words, a bundle is a distribution and composition layer over the primitives you already use. Installing a bundle still installs a real extension and a real preset — the bundle just guarantees you get the complete, coherent set, at pinned versions, in one command.

A bundle is described by a `bundle.yml` manifest. Here is a real one, the `aide-in-place` bundle, which pairs the AIDE extension with the AIDE In-Place Migration preset:

```yaml
schema_version: "1.0"

bundle:
  id: aide-in-place
  name: "AIDE In-Place Migration"
  version: "1.0.0"
  role: migration
  description: "Bundles the AIDE extension and the AIDE In-Place
    Migration preset so an in-place technology migration workflow
    (X → Y) installs in a single step."
  author: mnriem
  license: MIT

requires:
  speckit_version: ">=0.11.0"

provides:
  extensions:
    - id: aide
      version: "1.0.0"
  presets:
    - id: aide-in-place
      version: "1.0.0"
      priority: 50
      strategy: replace

tags:
  - migration
  - in-place
  - brownfield
```

Read the `provides` block top to bottom and you can see the entire stack: one extension and one preset, each pinned to a version, with the preset's `priority` and `strategy` spelled out. That last detail matters — the preset wiring that you would otherwise have to remember and apply by hand is captured declaratively in the manifest.

## The lifecycle, end to end

Bundles are discovered through the same catalog stack as everything else in Spec Kit — a priority-ordered set of project, user, and built-in sources. That means the process for finding and installing a bundle mirrors the rest of the CLI.

**Find one:**

```bash
specify bundle search migration
```

Search lists matching bundles with their version, role, source, and a trust indicator — `verified` for org-curated catalog entries, `community` otherwise — so you can judge trust before you install anything.

**Preview exactly what it will do:**

```bash
specify bundle info aide-in-place
```

This is my favorite part. `info` shows the **fully expanded component set** — every extension, preset, step, and workflow with its pinned version, plus preset priority and strategy. It is the same plan that `install` will apply, so there are no surprises. You see precisely what is about to land in your project before you commit to it.

**Install it:**

```bash
specify bundle install aide-in-place
```

One command installs the whole stack through each primitive's own installer. And there is a nice convenience here: if the current directory is not yet a Spec Kit project, `install` initializes one first, so a fresh checkout reaches a working state in a single step. Installation is idempotent — components already present are skipped.

From there the rest of the lifecycle is what you would hope for:

- `specify bundle list` — show what is installed, with versions, component counts, and timestamps.
- `specify bundle update [<id>]` (or `--all`) — re-resolve and refresh components to the bundle's newly pinned versions, while preserving primitive-level overrides like preset priority.
- `specify bundle remove <id>` — uninstall only the components this bundle contributed, leaving anything another installed bundle still needs in place. No collateral damage.

## Authoring your own

Bundles are not just something you consume. The same CLI lets you build them:

```bash
specify bundle validate   # is bundle.yml well-formed and do its refs resolve?
specify bundle build      # produce a single versioned .zip artifact
```

`validate` checks that every declared component reference resolves — against bundled components, the project's installed components, and, when online, the active catalogs. Sensibly, references it simply cannot verify (because you are offline, or a catalog is unreachable) are downgraded to warnings rather than failing the run, so authoring can continue. `build` then packages a bundle directory into one distributable `.zip` that embeds the manifest and can be installed directly with `specify bundle install <artifact.zip>`.

That artifact path is also what makes bundles work in air-gapped and offline setups: build once, hand someone the `.zip`, and they get the whole stack without touching a catalog.

## Sharing with the community

Bundles follow the same community model as the rest of Spec Kit. The project already runs community catalogs for extensions, presets, and more — where maintainers verify that an entry is complete and well-formed, but do not audit or endorse the code itself (that is exactly what the `community` trust indicator signals). Bundles are open to the same: if you build one worth sharing, the project welcomes community submissions, and the trust indicators keep that openness honest by telling installers what has and has not been vetted.

## Why I like this design

What stands out to me is the discipline of the design. Bundles could easily have become a second, parallel way to define behavior — a place where logic leaks in and you end up debugging "the bundle" as if it were its own runtime. Instead, the team drew a hard line: **a bundle composes, it does not execute.** Every component still installs through its own machinery, with full provenance tracking so it can be cleanly removed or refreshed later. The bundle is just the manifest that says "these pieces, these versions, wired this way."

The result is that bundles give you the one thing teams actually need — reproducible, one-command onboarding into a complete process — without giving up the modularity that made the primitives worth having in the first place.

That is a good note to start the series on. Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Bundles reference](https://github.com/github/spec-kit/blob/main/docs/reference/bundles.md) and the [Spec Kit README](https://github.com/github/spec-kit/blob/main/README.md).*
