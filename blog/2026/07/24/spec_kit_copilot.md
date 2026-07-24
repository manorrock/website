# The Spec Kit Copilot Plugin — Driving the Kit Without Leaving the Agent

*A companion to the [Spec Kit series](../../06/26/spec_kit_bundles.html). No part number on this one — the twelve-part arc was about the agent-agnostic kit, and this is about the Copilot bridge that sits beside it.*

The [series](../../06/26/spec_kit_bundles.html) I just finished spent twelve posts on [Spec Kit](https://github.com/github/spec-kit) the way it is designed to be described: agent-agnostic. The whole point of [Part 4](../01/spec_kit_integrations.html) was that the kit does not care which AI coding agent you run — it renders its process into whatever dialect your agent speaks. That is the right way to think about the core project, and I stand behind it.

But there is an asymmetry hiding in that framing, and a small companion — the [spec-kit-copilot](https://github.com/github/spec-kit-copilot) **plugin** — exists precisely to close it. This post is about that plugin: what it actually is, and why the seam it removes is the one between *reasoning over an idea* and *delivering it*.

## The seam integrations left open

Recall the split from Part 4. Integrations translate the kit's *process* into your agent's command palette: after `specify init`, the specify-plan-implement commands show up as things your agent can run directly. So the **delivery track** — the Spec-Driven Development loop — already lives inside the agent. Good.

What does not live inside the agent is the `specify` CLI *itself* — the management surface. Installing an extension, adding a preset, running a workflow, upgrading the tool: all of that is still `specify extension add …`, `specify workflow run …`, typed into a terminal. So the real rhythm of using the kit is a context-switch loop. You leave the agent, go to a shell, run a `specify` command to *shape* your environment, come back to the agent to actually *work* in it. The process is in the agent; the tooling that configures the process is not.

That is the gap. It is not a bug in the kit — the CLI *should* be agent-agnostic — but for the specific case where your agent is GitHub Copilot, there is no reason the agent should not be able to drive the CLI on your behalf. That is the entire thesis of the plugin.

## What it actually is: skills, not a second agent

Here is the part I want to be precise about, because it is easy to assume the wrong architecture. The Spec Kit Copilot plugin is **not** a dispatcher that forwards your prompts to some other Spec Kit agent. It is a [GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli) **skills plugin** — a `plugin.json` manifest and a handful of `SKILL.md` files — and Copilot itself runs the CLI. The plugin's own [`AGENTS.md`](https://github.com/github/spec-kit-copilot/blob/main/AGENTS.md) states the decision flatly: *"Skills are guidance, not dispatch."* A skill is a document that tells Copilot **when** a `specify` command group is relevant and **how** to invoke it correctly — nothing more.

The layout is exactly as small as that implies:

```text
spec-kit-copilot/
├── plugin.json                    # the plugin manifest
├── .github/plugin/marketplace.json
└── skills/
    ├── speckit-cli-setup/SKILL.md
    ├── speckit-init/SKILL.md
    ├── speckit-check/SKILL.md
    ├── speckit-extension/SKILL.md
    ├── speckit-preset/SKILL.md
    ├── speckit-bundle/SKILL.md
    ├── speckit-workflow/SKILL.md
    ├── speckit-workflow-step/SKILL.md
    └── speckit-self/SKILL.md
```

One skill per `specify` command group. Each `SKILL.md` is YAML frontmatter plus a Markdown body: the frontmatter carries a deliberately discovery-oriented `description` — a `USE FOR:` clause and a `DO NOT USE FOR:` clause — and the body lists the exact subcommands, flags, and gotchas. So when you say "add the git extension," Copilot's skill selector matches `speckit-extension` (and not `speckit-preset`, whose `DO NOT USE FOR` explicitly waves you off), loads its body, and runs the real `specify extension add git`. The `USE FOR`/`DO NOT USE FOR` discipline is the same disambiguation instinct the core kit applies to command resolution, turned on *skill selection* instead. It is how the agent avoids reaching for the wrong tool.

Install it, and the whole thing is one marketplace registration:

```bash
copilot plugin marketplace add github/spec-kit-copilot
copilot plugin install spec-kit-copilot@spec-kit-marketplace
```

## The mechanism that makes it more than a wrapper

If the plugin only shelled out to `specify`, it would be a convenience and not much else. The piece that makes it genuinely close the seam is in the `speckit-init` skill, and it is a single non-negotiable flag pair:

```bash
specify init . --integration copilot --integration-options="--skills"
```

Skills mode. When Copilot scaffolds a project this way, the spec-kit commands — and, crucially, the commands from any extension you add *later* — install as Copilot Agent Skills at `.github/skills/speckit-<cmd>/SKILL.md`, not as the default `.agent.md` files. That matters because `SKILL.md` is exactly what Copilot CLI discovers. So the plugin does not merely let the agent *run* the CLI; it arranges for everything the CLI *produces* to show up in the same skill system the plugin itself lives in. Add the `assess` extension, and its five new commands become five new skills. Run `/skills reload`, and they are live *in the current session* — no restart. The management surface and the generated process converge into one namespace the agent reads natively.

The rest of the plugin is honest plumbing around that idea, and the honesty is the point:

- **`speckit-cli-setup` is the bootstrap.** Every other skill carries a prerequisite note: if `specify --version` fails, defer to `speckit-cli-setup`, which installs the CLI via `uv` (preferred) or `pipx`. The plugin never pretends to *be* the CLI — it is a thin, guided layer over a real external binary, and it says so.
- **One skill is conspicuously absent.** There is no `speckit-integration` skill, and `AGENTS.md` marks that as a decision not to be silently undone. Managing or switching integrations is meaningless *here* — the agent is already Copilot. Leaving that skill out is the same "compose, don't accrete" restraint the whole series kept praising, applied to the plugin's own surface. It ships the nine skills that mean something for Copilot and not the tenth that would only mean something for someone else.

## Where the "idea → delivery" part comes in

Everything above buys you a smoother delivery loop. The reason I think this project is worth a post, though, is what happens when you put the *discovery* track in front of it — and the plugin makes that track reachable from the same conversation.

Delivery, the SDD loop, answers *"how do we build it?"* It assumes you have already decided the thing is worth building. But most ideas arrive raw — a GitHub issue, a Slack thought, a "could we…" — and the honest first question is *"is this even worth building?"* Spec Kit answers that with the [`assess` extension](https://github.com/github/spec-kit/tree/main/extensions/assess): a five-stage **Idea Assessment Pipeline** — intake → research → define → shape → decide — that turns a raw idea into a defensible **go / needs-clarification / kill** verdict *before* a single line of spec is written. Its own README calls it "the missing discovery track that sits in front of the SDD delivery track," and it is explicitly a funnel: killing an idea with a documented reason is a *successful* outcome.

Now watch what the plugin lets that look like from a single Copilot session. Every line below is *exactly what the user typed* into Copilot — nothing else. This is the complete list of prompts the user entered in the [community walkthrough](https://github.com/mnriem/spec-kit-copilot-plugin-demo) the plugin points at, verbatim (the `#` notes are mine, added to show what each one triggers):

```text
You: Initialize the project as a Spec Kit project using Copilot and skills mode
You: /skills reload
You: Add the assess extension
You: /skills reload
You: Now do an intake using https://github.com/github/spec-kit/issues/3423
You: Continue          # → research: evidence for AND against
You: Continue          # → define: users, goals, non-goals, metrics
You: Continue          # → shape: 2–3 options with appetite
You: What about D a separate page describing option 4 in more detail?   # refine: adds Option D, makes it the pick
You: Continue          # → decide: go → hand off to /speckit.specify
You: Continue          # → specify
You: Yes               # → plan
You: Yes               # → tasks
You: Do the implementation
```

Every one of those `You:` lines is a prompt a human typed — that is the entire input to the run. And it is worth being honest about what that means: this is one *actual* conversation, not a script to paste. Most of it is terse `Continue`s and `Yes`es, but notice the line that breaks the rhythm — *"What about D a separate page describing option 4 in more detail?"* — where the developer stops to argue with the `shape` stage and add a fourth option. That is the point, not a blemish on it: a `shape` step that asks its own clarifying question, or a `research` finding you want to push back on, will have you typing something different every time. The exact words differ each session because it *is* a conversation. What is stable is the shape — intake, through the funnel, to a verdict, into delivery — not the incantation.

Read that top to bottom. A raw issue goes in; a shipped change comes out. And at no point does the developer leave Copilot to type a `specify` command by hand. Copilot drives the CLI to install the discovery track, reloads its own skills to pick up the five new assess commands, walks the funnel, and — only on a `go` — hands the surviving idea to the delivery track it already knew how to run. The two tracks were always separate pieces of the kit; the plugin is what lets you reason across both of them in one continuous conversation.

That is the sense in which it makes "reasoning over an idea, then delivery" *easier*. Not by inventing a new process — every step there is stock Spec Kit — but by collapsing the context-switches between the tooling and the work down to zero. Discovery and delivery stop being two tools you alternate between and become one thing you talk your way through.

## The recursion I find satisfying

There is a tidy symmetry worth naming. Part 4 turned the kit's *process* into the agent's own commands — it rendered specify-plan-implement into Copilot's palette. This plugin turns the kit's *management* into the agent's own skills — it renders `specify extension`, `specify workflow`, `specify self` into things Copilot knows how to reach for. Integrations brought the *process* inside the agent; the plugin brings the *tooling that shapes the process* inside too. Put them together and both halves of Spec Kit — the loop you run and the kit you assemble it from — live in the same place you were already working.

There is one more layer of recursion worth pointing out, because it is not incidental. Look at where that walkthrough's issue lives: [`github/spec-kit#3423`](https://github.com/github/spec-kit/issues/3423) is an issue *in Spec Kit's own repository*, and the change the run shipped — a new "spec of specs" concept page wired into the docs — landed in Spec Kit's own docs. So the demo is Spec Kit dogfooding itself: the plugin drove the `assess` discovery track and the SDD delivery track to carry a real Spec Kit issue all the way to a real Spec Kit change. The tool was used to build the tool. One honest caveat on the paper trail: only the shipped docs change went into the Spec Kit repo — the assessment and spec artifacts from the run were not retained there. The record of the *process* — the intake, the funnel, the exact prompts — lives in the [demo repo](https://github.com/mnriem/spec-kit-copilot-plugin-demo) instead. So Spec Kit's history shows the outcome; the demo shows how it got there.

## Why I like this design

What I appreciate is that it is *small*. It would have been easy to answer "make Spec Kit great in Copilot" with a bespoke integration full of custom logic — and it would have rotted the moment the CLI moved. Instead the answer is nine Markdown files that say, in effect, "here is when each `specify` command group matters, and here is how to call it correctly," plus one insistence on skills mode so the CLI's output lands where the agent can see it. The intelligence is Copilot's; the CLI is the real Spec Kit; the plugin is just the thin, legible seam between them.

It also keeps faith with the trust story the series kept circling. A skill that runs `specify` is still running a real binary with your privileges, in no sandbox; the plugin is guidance, not a safety layer, and it does not pretend otherwise. That is the same honesty the core kit shows about extensions and `shell` steps — the bridge inherits the boundary rather than blurring it.

That is the same lesson the whole series kept arriving at from other directions: compose, don't accrete; be honest about the trust boundary; let small pieces snap together rather than growing a monolith. It is a good sign when the bridge to a tool is built out of the same values as the tool. The kit taught its agents to speak its process. This is the piece that lets one of those agents also drive the kit — and, once you have both, the distance from *"is this worth building?"* to *"it is built"* is a conversation, not a tab-switch.

If you try it — especially the discovery-then-delivery run from a real issue — I would like to hear where it felt seamless and where it did not. Send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Copilot plugin](https://github.com/github/spec-kit-copilot) and its [`AGENTS.md`](https://github.com/github/spec-kit-copilot/blob/main/AGENTS.md) design notes, the [`assess` extension](https://github.com/github/spec-kit/tree/main/extensions/assess), the [end-to-end walkthrough](https://github.com/mnriem/spec-kit-copilot-plugin-demo), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 6: From Spec to Kit](../06/spec_kit_kit.html), [Part 9: Workflows](../09/spec_kit_workflows.html), and the [full series](../../06/26/spec_kit_bundles.html).*
