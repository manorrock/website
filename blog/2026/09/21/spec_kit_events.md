# Spec Kit, Part 13: Events — When the Agent Calls Back

*The thirteenth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

At the end of [Part 10](../../07/10/spec_kit_steps.html) I said the machinery was all on the table. [Part 12](../../07/14/spec_kit_road_ahead.html) went further and called the architecture arc closed: five primitives, one catalog model, twelve posts from the bundle at the top down to the step at the bottom. That ending was a little too neat. I had to come back to the series, because otherwise it would leave out something particularly useful simply because it looks like plumbing: `specify event`, a little gem of a runtime primitive that makes native agent events portable.

Everything earlier in the series starts with you. You install an extension. You invoke a command. You start a workflow. Even automation begins because a person or a system explicitly asks Spec Kit to run. Events reverse that direction. The coding agent starts a session, receives a prompt, prepares to use a tool, finishes using it, stops, or ends the session — and *that lifecycle moment calls back into Spec Kit*. An event is how an extension says, "When the agent reaches this moment, run my command," without its author having to know whether Copilot calls that moment `sessionStart`, Claude calls it `SessionStart`, Cursor stores it in `.cursor/hooks.json`, or OpenCode exposes it through a TypeScript plugin. The extension speaks one Spec Kit vocabulary. The integration translates it into the agent's native one.

That makes `event` a slightly different kind of primitive from the ones before it. It is not another catalog of installable content alongside extensions, presets, workflows, and steps. It is a **runtime primitive** beneath them: a compact contract that turns incompatible native hook systems into one portable lifecycle. The implementation is plumbing, certainly, but that is not a caveat — it is the point. This is the plumbing native events were missing. At its portable core, it is only six names, a command reference, an optional timeout, and a payload moving through standard input and output; that small surface is enough to let extension authors build against an event *once* and have every event-capable integration carry it to the native agent. That is exactly the kind of leverage a primitive is supposed to provide.

## What an event is for

The closest thing Spec Kit already had was an extension **hook**. The git extension, for example, can declare `before_plan` or `after_implement` and attach `speckit.git.commit` to those points. But those hooks live inside the Spec Kit command sequence. They mean "before this Spec Kit command" and "after this Spec Kit command," and the command templates surface them as part of the process.

Events live one layer lower, in the coding agent itself.

That difference is easiest to see with `pre_tool_use`. There is no Spec Kit command called "use a tool." The agent might be about to edit a file, run a shell command, or call an external service while carrying out any command or ordinary conversation. A `pre_tool_use` event lets an extension react at that boundary because the *agent* crossed it, not because the user invoked a particular `speckit.*` verb.

So the two mechanisms answer different questions:

| Mechanism | The trigger is... | Example |
| --- | --- | --- |
| Extension hook | A point before or after a Spec Kit command | Commit changes after `speckit.plan` |
| Agent event | A point in the coding agent's native lifecycle | Validate a tool call before it runs |

Hooks compose the process. Events let extensions participate in the agent lifecycle around it.

## Six names across many dialects

The event layer starts with six canonical names:

| Spec Kit event | Lifecycle moment |
| --- | --- |
| `session_start` | The coding-agent session begins |
| `user_prompt_submit` | The user submits a prompt |
| `pre_tool_use` | The agent is about to invoke a tool |
| `post_tool_use` | A tool invocation has completed |
| `stop` | The agent finishes its current turn |
| `session_end` | The coding-agent session ends |

They are deliberately plain, snake_case names. An extension author declares one of those and stops thinking about agent-specific schemas.

The integration does the translation. For example, Copilot maps `user_prompt_submit` to `userPromptSubmitted`; Claude and Codex map it to `UserPromptSubmit`; Gemini and Tabnine use `BeforeAgent`; Cursor uses `beforeSubmitPrompt`; OpenCode uses `chat.message`. The same spread exists for tool and session events, and the storage format varies just as much: JSON under `.github/hooks/` for Copilot, JSON settings for Claude and Gemini, TOML for Codex and Mistral Vibe, and a generated TypeScript plugin for OpenCode.

That is exactly the kind of variation [Part 4](../../07/01/spec_kit_integrations.html) argued an integration should absorb. An extension should describe *intent* — "run this before a tool" — not learn every agent's capitalization, configuration paths, timeout units, matcher shapes, and output protocols. The event adapter owns that accidental complexity.

Not every agent supports every lifecycle point, and Spec Kit does not pretend otherwise. An adapter publishes the canonical events it can map; unsupported ones are skipped with a warning. The abstraction creates one vocabulary, not imaginary parity.

## What an extension declares

An extension opts into the lifecycle with a top-level `events:` block in `extension.yml`:

```yaml
schema_version: "1.0"

extension:
  id: project-guard
  name: "Project Guard"
  version: "1.0.0"
  description: "Loads project policy and checks tool activity"
  author: your-handle

provides:
  commands:
    - name: speckit.project-guard.context
      file: commands/context.md
      description: "Load project policy into agent context"
    - name: speckit.project-guard.check
      file: commands/check.md
      description: "Check a tool invocation against project policy"

events:
  session_start:
    command: speckit.project-guard.context
    timeout: 30
  pre_tool_use:
    command: speckit.project-guard.check
    timeout: 10
```

The required field is `command`. `timeout` is an optional positive number of seconds, and the event name must be one of the canonical six. There is also an optional `matcher` for narrowing tool events where an agent supports it, but that value belongs to the agent's native tool vocabulary; the portable part is the lifecycle event itself.

There is a subtle but important constraint behind that example. An event command is not the Markdown prompt body being sent back through the model. The runtime resolves the command template's `scripts:` frontmatter and executes the selected script directly:

```markdown
---
description: "Load project policy into agent context"
scripts:
  sh: scripts/bash/load-context.sh
  ps: scripts/powershell/load-context.ps1
  py: scripts/python/load_context.py
---

Load the project's policy context.
```

That keeps the event path deterministic. A session-start handler should not need to launch a second agent conversation to prepare context for the first one, and a pre-tool check cannot ask the same agent that is waiting to use the tool to wander off and interpret another prompt. The command remains the unit extensions already know how to provide, but its script is the executable event handler.

The project's persisted script choice still applies. If the project was initialized for Python scripts, the `py` entry runs; for PowerShell or shell projects, the corresponding variant is selected. Events reuse the command and script machinery rather than inventing a second executable-extension format.

## The dispatcher is the hinge

When an event-capable integration has events to install, initialization generates a small dispatcher at:

```text
.specify/events.py
```

It also merges native hook entries into the agent's own configuration. Those entries invoke the dispatcher with the Spec Kit command name, canonical event name, and handler timeout. The agent sends its native event payload on standard input; the dispatcher finds the command template, resolves the correct script variant, runs it from the project root, and sends the payload through unchanged.

The public command exposes the same runner directly:

```bash
printf '{"tool":"shell","command":"git status"}' |
  specify event run speckit.project-guard.check pre_tool_use 10
```

You are not expected to type `specify event run` often. In normal use the generated native hook calls `.specify/events.py` for you. The command is intentionally plumbing; the primitive is the portable contract around it — canonical event in, native payload through, extension command run, native context shape out.

The dispatcher is self-contained for a practical reason. `specify init` can be run through a temporary `uvx` environment that disappears as soon as initialization finishes. A generated hook that depended on that exact CLI installation would be dead on arrival. So the dispatcher prefers the installed `specify_cli` package when it is available and carries a standard-library fallback resolver when it is not. Either way, it can locate the command and run its script without requiring a permanent `specify` executable on `PATH`.

That is a small implementation choice with a large consequence: the event configuration survives the way the project was initialized. One-time setup still produces durable runtime behavior.

## Standard input in, native context out

Agent hooks agree that lifecycle payloads exist and disagree about nearly everything else.

The input side is the simple half. Spec Kit treats the native payload as UTF-8 text on standard input and forwards it to the handler script. It does not force every agent's JSON into one lossy universal schema. A handler that cares about tool arguments can inspect the native payload; one that only needs to refresh a file can ignore it.

The output side needs more translation. Claude and Codex can consume plain standard output as context. Copilot expects an `additionalContext` JSON field. Cursor uses `additional_context`. Gemini, Qwen, Devin, and Tabnine use a nested `hookSpecificOutput` envelope for lifecycle points that accept context, and require silence where output cannot be injected. Mistral Vibe expects its own structured response. OpenCode needs the dispatcher output adapted inside its generated plugin.

So the same handler can do the simplest possible thing:

```text
The project targets Java 25 and requires tests for every bug fix.
```

The integration wraps that text in the shape its agent understands — or suppresses it on lifecycle points where the agent cannot use it. That is more than configuration-file translation. It is protocol translation, and it is what makes a `session_start` context provider genuinely portable rather than merely installable everywhere.

## Resolution is layered, but events accumulate

The final event set is resolved from several sources:

1. Built-in defaults supplied by the integration.
2. `events:` declarations from installed, enabled extensions.
3. A per-integration override in `.specify/integration-events.yml`.

Built-in and extension declarations accumulate. If two enabled extensions both declare `session_start`, both handlers run; the second does not silently erase the first. That additive behavior is the only one that makes sense for an ecosystem — installing a context extension should not disable a telemetry extension merely because both care about session startup.

The project override is deliberately stronger. When an integration appears in `.specify/integration-events.yml`, its event map replaces the accumulated set entirely:

```yaml
integrations:
  copilot:
    events:
      session_start:
        command: speckit.project-guard.context
        timeout: 20
```

An explicit empty map disables events for that integration:

```yaml
integrations:
  copilot:
    events: {}
```

And `--events false` is the installation-level off switch. The important part is that disabling does not merely stop generating new entries; Spec Kit removes the native hooks it previously installed. Extension add, remove, enable, and disable operations also refresh every installed event-capable integration, so lifecycle configuration tracks extension state instead of leaving stale handlers behind.

That last behavior is easy to underestimate. A disabled extension whose native hook keeps firing is not disabled. Treating refresh and teardown as part of the extension lifecycle is what makes the abstraction true rather than decorative.

## The trust boundary moved earlier

[Part 10](../../07/10/spec_kit_steps.html) called a custom workflow step the sharpest trust edge in the system because it is arbitrary Python running with your privileges. Events deserve the same warning, with one extra emphasis: they can run *without an explicit command from you at that moment*.

A `session_start` handler runs because a session started. A `pre_tool_use` handler runs because the agent is about to use a tool. Installing an extension with `events:` therefore grants it a place in the agent's normal lifecycle, and the handler script has the same local privileges as the process invoking it. There is no sandbox, and a canonical event name is not a permission boundary.

The implementation does real defensive work around that fact. Event script paths must resolve inside the project tree, including after symlinks are resolved. Commands are executed as argument vectors rather than shell-interpolated strings. Hook command components are quoted for the target shell. Payloads must be valid UTF-8 and are capped at 1 MiB. Every handler has a timeout, with the agent's outer timeout buffered so the dispatcher can terminate its child cleanly first. Generated entries are marked so upgrade and teardown remove Spec Kit's hooks without deleting unrelated user configuration.

Those are meaningful protections against malformed input, accidental hangs, path escapes, and destructive config rewrites. They are not a code audit. The rule from the extension and workflow posts survives intact: read the source before installing something that runs on your machine. For an event-bearing extension, read the command's `scripts:` declaration and the script it points to, because that is the code the agent lifecycle will execute.

One failure choice is worth calling out. A missing or unresolvable event command becomes a warning and a no-op rather than preventing the coding agent from starting. A handler that actually runs and returns a non-zero status propagates that failure. The bridge avoids bricking the host over stale configuration, but it does not repaint a real handler failure as success.

## Why I like this design

What I appreciate is how much leverage comes from such a small primitive. Spec Kit did not invent a private event loop and ask every agent to integrate with it. It accepted that the agents already have native lifecycles, named the six moments extensions can reasonably share, and made each integration translate between the two. The native configuration stays native. Copilot still gets Copilot hooks, Codex still gets TOML, OpenCode still gets a plugin. The common layer is the extension author's intent, not a lowest-common-denominator runtime pretending the agents are identical.

It also completes the promise of integrations more fully than Part 4 could. Translating slash commands is the visible half of speaking an agent's language. Translating when the agent starts, prompts, uses tools, stops, and ends — including the shape required to put handler output back into context — is the conversational half. An integration is no longer only how Spec Kit talks *to* an agent. Events make it how the agent talks *back*.

And the reuse is characteristically disciplined. Events point at extension commands. Commands point at the same cross-platform scripts they already used. Initialization owns the generated files. Extension lifecycle operations refresh the native hooks. Integrations own vendor-specific names and output envelopes. No new catalog, no parallel plugin format, no requirement that an extension author become an expert in every agent's hook API.

So yes, the twelve-part architecture arc was missing a post. I had to come back because otherwise the series would leave out one of the most useful ideas in the kit precisely because it looks like plumbing. Part 13 is that little gem: a tiny runtime primitive that makes native agent events portable, and the point where a kit of processes stops waiting to be invoked and becomes present in the lifecycle of the agent carrying those processes out.

If there is an event-driven extension you want to build — context loaded at session start, policy checked before tools, evidence captured after them, cleanup at session end — send an email to blog (at) manorrock.com. It is a small seam, and I suspect the interesting part will be what people attach to it.

---

*Further reading: the [`events.py` runtime](https://github.com/github/spec-kit/blob/main/src/specify_cli/events.py), the [`specify event` command](https://github.com/github/spec-kit/blob/main/src/specify_cli/commands/event.py), [Part 12: The Road Ahead](../../07/14/spec_kit_road_ahead.html), [Part 10: Steps](../../07/10/spec_kit_steps.html), [Part 4: Integrations](../../07/01/spec_kit_integrations.html), and [Part 2: Extensions](../../06/29/spec_kit_extensions.html).*
