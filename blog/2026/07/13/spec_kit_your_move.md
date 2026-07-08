# Spec Kit, Part 11: Where Do You Go From Here?

*The eleventh entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

At the end of [Part 10](../10/spec_kit_steps.html) I said the natural next question was really two questions at once: where does Spec Kit go from here, and where do *you* go from here? I promised the road ahead, "yours as much as the project's." I want to take the second half first, and on purpose. The project's roadmap is interesting, but it is the maintainers' to walk. Yours is the one that actually changes anything this week — and after ten posts describing machinery, the fair thing is to hand you the machinery back and be specific about what to do with it. This is the practical capstone: not what Spec Kit *is*, but what your first move looks like.

The reason this deserves its own post rather than a closing paragraph is that ten posts of architecture can leave the wrong impression — that adopting Spec Kit means adopting *all* of it, that you owe the tool a constitution and a workflow and a custom step type before you have earned anything. You do not. Every layer I described composes precisely so that you can take *parts of it* and stop. The move is small. The whole point of this post is to make the smallest useful move obvious, and then to point at the one resource I have under-sold for ten posts: the community docs, which are the real map of what other people already did so you do not have to.

## The first move is smaller than you think

Here is the honest starting line, and it is lower than the series may have implied. You do not begin by writing anything. You begin by *reading* — one community component's source — and installing it. That is it. If you have a repo and a coding agent, you can have a real Spec Kit process running against real code in an afternoon, using nothing you authored yourself.

The lean path from the quickstart is four commands your agent runs for you, after a one-time `init`:

```text
/speckit.constitution → /speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement
```

That is the reference process from [Part 5](../02/spec_kit_process.html), and it is enough to feel the whole idea. For anything with real ambiguity, the same quickstart tells you to treat `/speckit.clarify`, `/speckit.checklist`, and `/speckit.analyze` as regular gates, and to close with `/speckit.converge`. But do not front-load that. Run the four-step spine once, on something you actually care about, and let the process earn the extra gates rather than adopting them on faith. The kit rewards incremental commitment; take it up on the offer.

## A ladder, not a leap

Everything after that first run is a ladder you climb only as far as you need. Each rung is a primitive from an earlier post, and — this is the part I want to insist on — you can get off at any rung and still have gained something real.

**Rung one: install one community extension.** Start with the [Community Extensions list](https://github.com/github/spec-kit/blob/main/docs/community/extensions.md) — 127 of them in the community catalog as I write this, from around seventy authors — or skim the browsable [community extensions site](https://speckit-community.github.io/extensions/). Sort for a `read-only` component first — there are 26 of them — because as [Part 8](../08/spec_kit_field_guide.html) argued, a read-only extension produces reports and never touches your files, which makes it the lowest-risk thing you can try. Read the `info` output, follow the repository link, read the source — it is usually small — and then `add` it. You have now done the entire consume-side loop, and you have done it safely.

**Rung two: tailor a preset for your team.** When the stock vocabulary or templates do not fit how your team already talks, a preset reshapes the process without you touching any tooling — the argument from [Part 3](../../06/30/spec_kit_presets.html). Browse the [Community Presets list](https://github.com/github/spec-kit/blob/main/docs/community/presets.md) or the [presets site](https://speckit-community.github.io/presets/) before you build one; someone may have already localized or re-flavored the process in the direction you want, and starting from their preset is faster than starting from a blank one.

**Rung three: write your first extension.** This is the first rung where you produce rather than consume, and it is smaller than it sounds — [Part 2](../../06/29/spec_kit_extensions.html) covered the shape. The right trigger is a piece of prompt-craft you have now typed by hand three times. That repetition *is* the spec for your extension. You are not inventing a capability; you are packaging a habit.

**Rung four: put a workflow in CI.** Once a process is worth running the same way every time, [Part 9](../09/spec_kit_workflows.html) is the rung that promotes it from something you perform to something you *invoke* — a stateful, resumable graph with human `gate` steps where judgment belongs, and `--json` output so a scheduler or bot can drive it. This is the rung most teams will actually feel the payoff on, and it is worth climbing to deliberately rather than by accident.

**Rung five: extend the vocabulary.** Only if the built-in step types genuinely cannot say what your process needs do you reach for a custom step, the open `StepBase` interface from [Part 10](../10/spec_kit_steps.html). Most teams never need this rung, and that is a feature, not a shortfall — the ceiling is there so that the rare team who hits it is not stuck.

The ladder's whole virtue is that the rungs are independent. Installing one extension does not obligate you to author one; running the reference process does not obligate you to a workflow. Take exactly the rungs your problem demands and leave the rest for the day you need them.

## The resource I have under-sold: the community docs

If you take one thing from this post, make it this — because I have cited these pages piecemeal for ten posts without ever telling you to just *go read the hub*. The [Community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md) is the single best map of where to go from here, and it is organized exactly along the ladder above:

- **[Extensions](https://github.com/github/spec-kit/blob/main/docs/community/extensions.md)** — the 127 community capabilities in the catalog today, the deepest well of "someone already built this." Read this before you ever write an extension.
- **[Presets](https://github.com/github/spec-kit/blob/main/docs/community/presets.md)** — reflavored and relocalized processes, from language packs to entirely different methodologies.
- **[Bundles](https://github.com/github/spec-kit/blob/main/docs/community/bundles.md)** — role- and team-shaped stacks that install a whole opinionated setup in one command, the [Part 1](../../06/26/spec_kit_bundles.html) idea at community scale.
- **[Walkthroughs](https://github.com/github/spec-kit/blob/main/docs/community/walkthroughs.md)** — step-by-step demonstrations across languages and scenarios, including the stock-process brownfield runs I leaned on in Part 8. If you learn better from a worked example than from reference docs, start here.
- **[Friends](https://github.com/github/spec-kit/blob/main/docs/community/friends.md)** — companion projects that build *on* Spec Kit rather than plug into it: VS Code extensions that give the SDD workflow a visual orchestrator, Claude Code plugins, npm-installable assistants. If the CLI is not where your team wants to live, a Friend may already have built the surface you want.

Two honest notes carried over from [Part 7](../07/spec_kit_catalogs.html) and Part 8, because they matter most exactly at the moment you start installing things. First, everything in these lists is independently authored and **not reviewed, endorsed, or supported by GitHub** — the maintainers verify that a catalog *entry* is well-formed, not that the *code* is safe. `verified` is a statement about metadata. Read the source before you `add`. Second, the community sites — [extensions](https://speckit-community.github.io/extensions/) and [presets](https://speckit-community.github.io/presets/) — are the human-readable front door onto the same data, built for skimming and sorting before you ever touch the CLI. Use them to shop; use the repository link to decide.

## Then close the loop: publish back

The rung past authoring is contributing, and [Part 7](../07/spec_kit_catalogs.html) already showed why it costs almost nothing new: publishing is the same act as consuming, seen from the other side. You write the same kind of catalog record you have been reading in `info` output, point it at your artifact, and submit it — the project provides issue templates precisely so the metadata a reviewer needs is captured up front. The extension you built at rung three, the preset you tailored at rung two, becomes a line in the same list you shopped from. That is how a hundred-plus community entries came to exist: people scratched an itch, and then took ten minutes to make their scratch findable. If you climb the ladder at all, consider leaving the rung behind you a little more crowded than you found it.

And there is a contribution below authoring that is pure gift: the community docs work as well as they do only when someone reads a stranger's source and reports back. Filing an issue that says "this extension's `info` says read-only but it writes to `specs/`" is a real contribution to the trust boundary the whole series keeps circling. You do not have to publish code to make the ecosystem safer; you have to read some.

## Where you actually go

So, concretely, the move. Pick a repo you care about. Run the four-step spine once. Open the [community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md), find one `read-only` extension whose source you are willing to read, read it, and install it. That is a complete first lap — process plus ecosystem — and everything else in this series is a rung above it that you climb only when a real problem asks you to. The architecture I spent ten posts on exists so that this first lap is small and the ladder is optional. Take the small lap.

That is the half of the road ahead that is yours. The other half — where the *project* is heading, what the maintainers are building next, which of the community clusters from Part 8 get pulled inward the way `converge` was — is the one I have deliberately left for another day, because it is speculation and this post was meant to be actionable. I will come back for it. If there is a direction in the project's roadmap you want me to dig into, a community component you think everyone should know about, or a first-move story of your own once you have taken the lap, send an email to blog (at) manorrock.com.

---

*Further reading: the [Community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md) and its [Extensions](https://github.com/github/spec-kit/blob/main/docs/community/extensions.md), [Presets](https://github.com/github/spec-kit/blob/main/docs/community/presets.md), [Bundles](https://github.com/github/spec-kit/blob/main/docs/community/bundles.md), [Walkthroughs](https://github.com/github/spec-kit/blob/main/docs/community/walkthroughs.md), and [Friends](https://github.com/github/spec-kit/blob/main/docs/community/friends.md) pages; the [Community Extensions](https://speckit-community.github.io/extensions/) and [Community Presets](https://speckit-community.github.io/presets/) sites; the [Quick Start Guide](https://github.com/github/spec-kit/blob/main/docs/quickstart.md); [Part 10: Steps](../10/spec_kit_steps.html), [Part 9: Workflows](../09/spec_kit_workflows.html), [Part 8: A Field Guide to the Community](../08/spec_kit_field_guide.html), [Part 7: Catalogs](../07/spec_kit_catalogs.html), [Part 6: From Spec to Kit](../06/spec_kit_kit.html), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
