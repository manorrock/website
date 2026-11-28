# From 1.0.0 to 150K — The Kit Kept Moving

*A maintainer's look back at what changed after the first birthday — and the community that kept it changing.*

[Spec Kit](https://github.com/github/spec-kit) now shows **150K stars** in GitHub's repository search. It has also moved ahead of [LangChain](https://github.com/langchain-ai/langchain) in star count. As maintainer, I want to pause and appreciate that before getting back to the work that brought us here.

The projects do different jobs, and stars are not a benchmark. But the attention tells me that a great many people have found something here worth keeping an eye on. For a project whose central proposition is still "perhaps we should agree on what we are building before asking an agent to build it," that feels worth celebrating.

The more interesting question, once the counter has had its moment, is what changed while it was climbing. For that, the right place to start is August 21st: Spec Kit's first birthday, and the day we shipped [1.0.0](https://github.com/github/spec-kit/releases/tag/v1.0.0).

## What happened after the birthday

In [the birthday post](../../08/21/spec_kit_turns_one.html), I described 1.0.0 as a moment of coherence rather than a declaration that we were finished. Extensions, presets, integrations, workflows, and steps already existed, along with the bundles and catalogs that let people package and share them. The work was making those pieces feel like one kit.

By November 10th we had reached [1.4.0](https://github.com/github/spec-kit/releases/tag/v1.4.0). The useful story between those two versions is not that we acquired four minor-version numbers. It is that the joints between the pieces became more capable, more visible, and more deliberate.

There is a fairly consistent instinct underneath it: **make the kit more useful without making the core responsible for everything.** The months after 1.0.0 gave us plenty of opportunities to find out whether we meant it.

## From separate pieces to reusable processes

September made composition more practical. Presets gained the ability to declare required extensions, so a process can name the behavior it depends on instead of leaving the user to discover a missing prerequisite. Workflows gained named slots: places where another layer can insert work without copying and maintaining an entire workflow.

Those are small-sounding capabilities with an important consequence. Customizing a process does not have to mean forking it. You can state what you need and where it belongs. I wrote about that in [Workflow Overlays and Slots](../../09/24/spec_kit_workflow_overlays_and_slots.html); it is one of the clearest examples of the kit becoming more useful at its seams.

October took the same idea up a level. An installed workflow can now run inside another workflow through a unified execution tree, with shared status and resume semantics. A reusable process is no longer just something you copy from another YAML file. Bundles then learned to reference exact versions of other bundles, expanding their declarations into one component plan rather than creating independently updating child installations.

For a maintainer, that is satisfying in a way a larger feature list is not. A team can package a common baseline, another team can build on it, and neither has to ask us to put its particular process into core.

## A kit should be able to explain itself

Of course, layers are only pleasant until you need to work out which one changed your command.

September brought **`specify artifact`**: a way to inspect the effective commands, templates, scripts, and hooks, see their composition stacks, and trace contributions back to their owners. That changed the answer to "why does this project behave this way?" from a small archaeological expedition into something the tool can help explain.

October gave some of that inspection a machine-facing transport. The **experimental local stdio MCP server** exposes typed tools for version information, artifact listing and inspection, and installed-workflow listing. These are read-only operations, not workflow execution or installation over MCP. The point is not to imply that every CLI capability has become an MCP tool. It is to let an agent ask supported questions without scraping terminal prose or guessing at private state.

I like this progression: first let the system explain itself, then make that explanation usable by other tools. Composability without inspectability would eventually make us the support desk for everyone's layers.

## More freedom, with fewer hidden assumptions

The catalog story changed too. Exact release selection reached extensions and workflows in September, then presets, custom workflow steps, and bundles in October. "Install this component" can mean a particular published release, not whatever happens to be newest when the next teammate joins.

October also opened the integration lifecycle to **catalog-installed external agent adapters**. Support for another coding agent no longer necessarily has to wait for a core release. That moves useful work closer to the people who know the agent. Adapters, extensions, and workflow functions can execute code, so review their source before installing or using them; a community catalog listing is discovery, not a security audit.

At the other end of the spectrum, the new **`none` integration** installs shared scaffolding without agent-specific delivery. Sometimes the right agent integration for a repository is no integration at all, because its contributors choose their own tools. Agent-agnostic should mean more than a long list of logos.

We applied that same boundary to our own capabilities. September shipped first-party bugfix and assessment bundles with their workflows, offering opt-in processes alongside core SDD rather than turning every task into a feature specification. Assessment can end in a decision not to build. A bug can need a scoped fix, not a ceremony.

Then, in November, **`taskstoissues` left core**; the GitHub-specific capability remains in the bundled, opt-in `github` extension. There is no compatibility alias in core. If you depended on the old command, that is a real change to account for, not something I want to hide in a celebratory post.

A smaller core and a richer kit are not competing goals. The last few months made that less of a design slogan and more of an operating decision.

## The community did not wait for us to finish

At the first birthday, I wrote about more than 150 community extensions and more than 30 presets. What stands out since then is not only the growth, but how much of the kit people are using together.

September's [Agentstandards bundle](https://github.com/bbjwz/agentstandards) brought an architecture council, task-gate preset, and supporting workflow together. It is a concrete example of someone packaging a whole way of working rather than one more command.

Microsoft's [Azure Cosmos DB extension](https://devblogs.microsoft.com/cosmosdb/spec-driven-development-comes-to-azure-cosmos-db-the-first-database-extension-for-github-spec-kit/), added around the birthday and introduced publicly in September, brought a database team's expertise into application planning and implementation.

November's [Spec Kit for Azure DevOps](https://github.com/praveenlavu/spec-kit-ado) connects the process to stories and tasks on an existing board, with human approval before board writes. That is a different kind of extension: fitting the kit into where a team already works.

These projects are maintained by their respective teams, not a set of capabilities the core team can take credit for. What I can take pleasure in is the shape of the participation. People are bringing their own problems, building a way through them, and making that work available to somebody else.

That is the difference between an audience and a community. It is also a much better test of the architecture than my being able to explain it in a blog post.

## The latest chapter: making the machinery more dependable

The most recent month carried that direction into the details. Command and prompt steps gained opt-in output capture. Workflow expressions moved to a proper Lark LALR grammar. Run-scoped definition inspection now shows the workflow a run actually started with. Opt-in strict shell mode rejects direct workflow-expression interpolation in shell source; environment bindings provide a separate channel for dynamic values.

The latest releases added per-step **reasoning effort and context size**, with workflow-level defaults. Copilot and Codex have native mappings; Claude Code supports reasoning effort, while an explicit context-size setting fails with guidance to choose a long-context model instead. Unsupported settings are errors, not suggestions silently dropped on the floor. A quick classification and a substantial design review need not ask the same amount of the same model, but that choice only means something if the integration actually honors it.

And [1.4.0 added workflow functions](https://github.com/github/spec-kit/blob/main/docs/workflows/functions.md): project-installed Python callables exposed as expression filters. If your process needs a particular transformation, it can live in your project rather than becoming everybody's next built-in filter. Again, the extension point is the feature.

Underneath that are the changes that ought to be unremarkable: preserving feature numbering during reinitialization, keeping user files when removing preset skills, retaining user-owned hook configuration, and making bundle-step refresh roll back transactionally. Filesystem and network hardening continued too, from symlink handling to handle-based FileHelper operations and consistent HTTPS requirements for catalog sources.

Documentation belongs in this chapter as well. We consolidated the extension, preset, bundle, and workflow guidance, organized the how-to material, and clarified shared APIs. A more capable kit is not much of an achievement if people cannot work out which piece they need.

This is what the work often looks like from the maintainer's chair. Not another milestone, but another way somebody's existing repository can survive using the tool without an unpleasant surprise.

## What I am actually celebrating

When I wrote about [six months as lead maintainer](../../07/22/six_months_leading_spec_kit.html), the repository had passed 123,000 stars. At the first birthday, it was above 130,000. Now GitHub is putting 150K next to the name.

The kit did not stand still between those numbers. It learned to compose more of the process, explain more of its own behavior, and leave more room for other people's choices. That is the kind of progress I hoped would follow 1.0.0.

I am proud of that. I am also very aware that my name on the maintainer line is not the explanation for it. Contributors, reviewers, extension authors, people answering questions, and people filing the awkward bug with a useful reproduction all helped get us here.

So thank you. For the stars, certainly. But also for the patience, the disagreements, and the work that happens between the milestones. The counter gets to be the headline today. You are the reason there is a project underneath it.

If you have tried the recent releases, send an email to blog (at) manorrock.com and tell me what became easier — or what still made you stop and work around us.

---

*Further reading: [Spec Kit Turns One](../../08/21/spec_kit_turns_one.html), the [September newsletter](https://github.com/github/spec-kit/blob/main/newsletters/2026-September.md), the [October newsletter](https://github.com/github/spec-kit/blob/main/newsletters/2026-October.md), the [Spec Kit releases](https://github.com/github/spec-kit/releases), and the [full series](../../06/26/spec_kit_bundles.html) on the Spec Kit primitives.*
