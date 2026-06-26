# Spec Kit, Part 2: Extensions — Adding Capabilities Without Bloating the Core

*The second entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

In [Part 1](../26/spec_kit_bundles.html) I wrote about bundles, the layer that composes everything else into a one-command install. A bundle is only as interesting as the primitives it bundles, though, so the rest of this series works *down* the stack. The most foundational primitive — the one that actually adds new things you can do — is the **extension**. So that is where we go next.

I want to be deliberate about the order here. Bundles compose primitives, so before we look at any of the fancier composition it pays to understand the building block underneath it all. Extensions are the source of new commands and templates in Spec Kit — everything else in the series either packages them, configures them, or builds on top of them. Introduce them first and the later layers have something concrete to refer back to.

## What an extension is for

Core Spec Kit gives you the Spec-Driven Development workflow: the built-in loop of specifying, planning, and building. That loop is intentionally general. The moment your work gets specific — a particular domain, an external tool you depend on, a quality gate your team insists on — you need capabilities the core does not ship.

An **extension** is how you add them. Extensions introduce new commands — plus the templates, scripts, and hooks that support them — that go beyond the built-in workflow: domain-specific commands, external tool integrations, quality gates, and more. The key word is *add*. An extension does not change how the existing process behaves — it gives you new things to do alongside it.

That is the cleanest way I have found to think about an extension: it is a self-contained unit of capability you opt into. The core stays small and general, and anything opinionated or domain-specific lives in an extension you install only if you want it.

## Installing one

Extensions install through the same catalog machinery as everything else:

```bash
specify extension search quality
specify extension add <name>
```

When you install an extension, its commands are **automatically registered with whichever AI coding agent integration you have set up**. There is no separate "now wire it into my agent" step — install it and the new commands show up where you already work. (The one prerequisite: extension commands need a project that has been initialized with `specify init` first.)

The rest of the lifecycle is exactly what you would expect, and it mirrors the other primitives so there is nothing new to learn:

- `specify extension list` — what is installed, with status, version, and command counts.
- `specify extension info <name>` — description, version, commands, and configuration for one extension.
- `specify extension update [<name>]` — refresh one, or all of them.
- `specify extension enable <name>` / `disable <name>` — turn an extension off without uninstalling it; a disabled extension is not loaded and its commands disappear.
- `specify extension remove <name>` — uninstall it (configuration is backed up by default; pass `--keep-config` to leave it in place, or `--force` to skip the confirmation prompt).

## Priority

Extensions take a `--priority`, and the rule is simple: when two extensions provide a command with the same name, the **lower priority number wins**. That gives you a deterministic way to resolve overlap when you have more than one extension installed — set the priorities and you know exactly which command answers to a given name.

You change it after the fact with `specify extension set-priority <name> <N>`, so reordering installed extensions never means uninstalling and reinstalling them.

## Configuration is a first-class feature

Here is the part of extensions I think is most underappreciated. Because extensions add real capabilities — often ones that talk to external tools — they need real configuration, and Spec Kit gives them a proper layered model rather than a single editable file.

Most extensions ship a small set of config files in their install directory. Here is the built-in `git` extension as an example:

```text
.specify/extensions/git/
├── git-config.yml         # Project config (version controlled)
├── config-template.yml    # Template to copy from
└── local-config.yml       # Local overrides (gitignored)
```

And the values are merged in a defined order, highest priority last:

1. **Extension defaults** (from `extension.yml`)
2. **Project config** (`<ext>-config.yml`, e.g. `git-config.yml`) — committed, shared with the team
3. **Local overrides** (`local-config.yml`) — gitignored, yours alone
4. **Environment variables** (`SPECKIT_<EXT>_*`) — top of the stack, perfect for CI and secrets

This is a genuinely nice design. The committed project config is how a team agrees on settings; the gitignored `local-config.yml` is how an individual deviates without polluting the shared file; and environment variables let CI or a secret manager override everything without a file at all. To get started you just copy the template the extension ships:

```bash
cp .specify/extensions/git/config-template.yml \
   .specify/extensions/git/git-config.yml
```

The same layering philosophy that governs file resolution governs configuration. Once you have internalized "lower layers are defaults, higher layers win," it applies everywhere in Spec Kit.

## Catalogs and trust

Extensions are discovered through the familiar priority-ordered catalog stack — environment variable, then project config, then user config, then the built-in official and community catalogs. An organization can add its own catalog of approved extensions at a high priority:

```yaml
catalogs:
  - name: "my-org-catalog"
    url: "https://example.com/catalog.json"
    priority: 5
    install_allowed: true
    description: "Our approved extensions"
```

`search` even has a `--verified` flag so you can limit results to org-curated, verified entries. As with the rest of the ecosystem, the honesty holds: most extensions are created and maintained by their authors, and the Spec Kit maintainers do not review, audit, or endorse extension code. An extension adds commands that run in your environment, so read the source before you install — the `verified` versus `community` signal tells you what has been vetted and what has not.

## Why I like this design

What I appreciate about extensions is the restraint. It would have been easy to let "add a capability" mean "patch the core," and Spec Kit would have slowly turned into a pile of special cases. Instead, capability lives *outside* the core, installed through the same catalog-and-registration machinery as every other primitive, with a clean config model and a defined place in the resolution stack. The core stays small and general; the interesting, opinionated, domain-specific stuff lives in extensions you opt into.

That is also what makes the rest of the series coherent. Bundles can compose extensions because extensions are self-contained, installable units with a known place in the system. Get extensions right and everything built around them has something solid to stand on.

Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Extensions reference](https://github.com/github/spec-kit/blob/main/docs/reference/extensions.md) and [Part 1: Bundles](../26/spec_kit_bundles.html).*
