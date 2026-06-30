# Spec Kit, Part 4: Integrations — Speaking Your Agent's Language

*The fourth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

The last three posts have a phrase in common that I kept letting slide. Extensions register their commands "with whichever AI coding agent integration you have set up." Presets do the same. Every time the workflow produced a command, something quietly made it show up in the tool you actually type into — and I never said what that something was. This time I will. The **integration** is the layer that takes Spec Kit's neutral commands and templates and renders them into the exact shape your coding agent expects, in the exact place it looks for them.

That is why integrations come now. We have been working down the stack: bundles compose, extensions add, presets customize. But all three of those operate in Spec Kit's own vocabulary — a `speckit.plan` command, a `plan-template.md`. None of that means anything to GitHub Copilot or Claude Code or Gemini CLI until it has been translated into *their* conventions. The integration is the translator, and once you see it the whole series clicks shut: every layer above produces intent, and the integration is where that intent finally lands somewhere you can run it.

## What an integration is for

Spec Kit is deliberately agent-agnostic. The core loop, the extensions, the presets — none of them assume which AI coding agent you use. That neutrality is a feature: it is what lets the same spec-driven workflow run on more than thirty different agents. But neutrality has to be cashed out somewhere, because those agents agree on almost nothing about how commands are stored and invoked.

GitHub Copilot installs skills into `.github/skills` and invokes them as `/speckit-<command>`. Claude Code instead wants skills in `.claude/skills` and context in `CLAUDE.md`; Gemini CLI wants commands in `.gemini/commands` and context in `GEMINI.md`. Codex and Zed even share the `.agents/skills` directory yet invoke their commands differently — `$speckit-<command>` for one, `/speckit-<command>` for the other. Some agents are CLI tools, some are IDE plugins, some are both. An **integration** is the adapter that knows all of this for one agent — the command file format, the context rules, the directory layout, the invocation style — so that when you run `specify init`, the CLI lays down exactly the files that agent will recognize and you can start doing Spec-Driven Development immediately, whatever tool you prefer.

That is the cleanest way I have found to think about it: extensions and presets decide *what* commands exist and *how they behave*; the integration decides *what form they take and where they live* so your particular agent can see them at all.

## Installing one

The most common way you meet an integration is at the very start, when you scaffold a project:

```bash
specify init my-project --integration copilot --integration-options="--skills"
```

But integrations are first-class primitives with a full lifecycle of their own, managed inside an already-initialized project:

```bash
specify integration list      # every available integration, which is installed, CLI vs IDE
specify integration search    # discover integrations across the catalog stack
specify integration install <key>
```

`list` is the one to start with — it shows all the supported agents with their keys, marks which integration is the current default, and tells you whether each one needs a CLI tool installed or works through an IDE. From there the lifecycle mirrors the other primitives, with a couple of verbs unique to this layer:

- `specify integration info <key>` — catalog details for one integration: description, author, license, tags, source catalog, and whether it is currently active.
- `specify integration uninstall [<key>]` — remove an integration's files.
- `specify integration upgrade [<key>]` — reinstall with refreshed templates and commands, e.g. after upgrading Spec Kit itself.
- `specify integration switch <key>` — replace the current default with another agent.
- `specify integration use <key>` — change the default among already-installed integrations without uninstalling anything.
- `specify integration status` — report the project's integration health without touching a file (there is a `--json` form built for CI and coding agents).

## Modified files are preserved, on purpose

Here is the part of integrations I think is the most quietly thoughtful. An integration writes real files into your project — command files, context rules, shared templates — and the obvious risk is that managing those files later will clobber edits you have made by hand. Spec Kit refuses to do that.

When an integration installs, it records every file it created along with a SHA-256 hash of the original content. That hash is what makes the rest honest:

- On **uninstall**, only files that still match their recorded hash are removed. Anything you have edited is left in place, so your customizations survive even as the integration that created them goes away.
- On **upgrade**, locally modified files block the operation rather than being silently overwritten — you have to pass `--force` to say "yes, discard my edits." Stale files from the previous install that are no longer needed get cleaned up automatically.
- On **switch**, the same protection applies to the integration being removed.

This is the same layering philosophy the earlier posts kept hitting — the system tracks provenance precisely so it can be reversed cleanly — applied to the one layer that actually owns files on disk. An integration is reversible the way a preset is, except the stakes are higher because real artifacts are involved, and the hashing is what earns that reversibility.

## More than one agent at a time

You are not limited to a single integration. A project can carry several, which matters for team portability: one developer drives the workflow through GitHub Copilot, another through Claude Code, and the repository serves both. Spec Kit tracks one **default integration** in `.specify/integration.json` alongside the full list of installed ones, and `use`/`switch` is how you move the default around.

The catch is that two agents writing into the same project can step on each other, so Spec Kit only allows multi-install automatically when every integration involved is declared **multi-install safe** — meaning it uses isolated agent directories, a dedicated context file that does not collide with another safe integration, and a separate install manifest. Agents that share a context file or command directory are not safe by default; you can still install them side by side, but you have to pass `--force` to acknowledge that the agents may see each other's instructions. Either way, the shared Spec Kit templates stay aligned to the single default integration, so the core never tries to speak two dialects at once.

## Catalogs and trust

Integrations are discovered through the same priority-ordered catalog stack as every other primitive — environment variable, then project config, then user config, then the built-in official and community catalogs — so an organization can publish its own approved list at a high priority and `search`/`info` will resolve against it first.

The built-in integrations for the major agents ship with Spec Kit itself, which puts integrations on slightly firmer trust footing than community extensions or presets. But the same honesty applies at the edges: catalogs can offer third-party integrations, an integration writes command and context files into your project, and the `verified` versus `community` signal is there to tell you what has been vetted. And if your agent is not on the list at all, the `generic` integration is the escape hatch — point it at a commands directory with `--integration-options="--commands-dir <path>"` and the workflow lands there instead.

## Where integrations sit

It is worth being precise about how integrations relate to the resolution stack from the last post, because they are not another layer in it. Presets and extensions and project overrides all compete *within* one stack to decide which version of a file wins. The integration sits at the end of that pipeline: once resolution has picked the winning template, the integration decides what format it is rendered into and which directory it is written to. Resolution answers "which content"; the integration answers "in whose dialect, and where."

That separation is what lets everything else stay agent-agnostic. An extension author never has to think about Copilot versus Claude, because the integration absorbs that difference downstream. The whole stack above can speak one neutral language precisely because exactly one layer at the bottom is responsible for translation.

## Why I like this design

What I appreciate about integrations is that they make agent support a boring, swappable detail instead of a fork in the road. It would have been so easy for Spec Kit to pick a favorite agent and bake its conventions into the core, and every other tool would have been a second-class bolt-on forever. Instead the team pushed all of that into one well-defined adapter layer with a real lifecycle, file-level provenance tracking, and a multi-install model — so adding the thirty-first agent is a new integration, not a rewrite, and switching agents is one command, not a migration.

Seen against the rest of the series, integrations are the floor the whole stack stands on. Bundles compose, extensions add, presets customize — and none of it would reach your fingertips without the layer that quietly translates it all into the agent you already use. It is fitting that it is the piece I leaned on for three posts before finally naming it.

Next time I will pick another piece of Spec Kit and do the same thing. If there is a particular capability you would like me to dig into, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Integrations reference](https://github.com/github/spec-kit/blob/main/docs/reference/integrations.md), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
