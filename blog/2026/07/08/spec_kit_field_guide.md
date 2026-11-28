# Spec Kit, Part 8: A Field Guide to the Community — What People Actually Built

*The eighth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

[Part 7](../07/spec_kit_catalogs.html) was about the machinery that makes community work findable — the catalog stack, the trust boundary, the `search`/`info`/`add` verbs that reach a stranger's extension in one command. I ended it with a promise: now that the *mechanism* is on the table, the obvious follow-up is the work itself. Well over a hundred community extensions and a couple dozen presets are out there, and some are genuinely worth your attention. This post is the guided tour I said I owed you — the components, not the catalog that lists them.

Two honest caveats before the tour, because Part 7 earned them. First, everything below is community-contributed: the Spec Kit maintainers verify that the catalog *entry* is well-formed, not that the *code* is safe. I have not audited any of these; nothing here is an endorsement of anyone's source. Read the repository before you type `add`. Second, this is an opinionated slice, not a directory. I picked a handful that make a point, and skipped dozens that are perfectly good — a complete list is what the [community sites](https://speckit-community.github.io/extensions/) are for.

## Reading the catalog as a heat map

Once there are a hundred-plus independent things people built on top of the same reference process, the *distribution* of what they built stops being noise and becomes a signal. Count the extensions by what they do and you are looking at a map of where the core process creates friction — sometimes a capability people wished existed, more often a chunk of manual prompt-craft people would rather not repeat. The community did not coordinate on that map; it emerged from a hundred separate itches. That makes it a more honest account of where the reference process asks the most of you than any changelog, because nobody wrote it on purpose.

So I am not going to march through the catalog alphabetically. I am going to walk the four or five places the community kept returning to, because each cluster is a sentence the community is speaking about the core — and then close on the one cluster that is not about the core at all, which turns out to be the most interesting.

## Brownfield: not a missing capability, an ergonomics tax being paid down

The single loudest theme in the catalog is *brownfield adoption*, and it is the one I most want to get right, because the obvious reading of it is wrong. The tempting story is that the reference sequence assumes a blank repo and the community had to teach it to handle existing code. That is not true, and the docs are explicit about it: "Iterative Enhancement (Brownfield)" is a *named, first-class* use case of Spec Kit, right alongside greenfield. The core process already does brownfield.

The proof is in the community walkthroughs, and the important part is that they use **no brownfield extension at all**. One drives the *stock* constitution → specify → plan → tasks → implement flow straight into a ~420,000-line Jakarta EE runtime spread across 180 Maven modules — with no prior specs or constitution — and the only special move is the very first one. The `speckit.constitution` step is invoked with a prompt that tells the agent to *analyze the existing codebase* rather than assume a blank slate:

```text
As this is a pre-existing brownfield project I need you to analyze the
codebase exhaustively and in-depth... use multiple iterations to do a deep
analysis and use or create principles focused on code quality, testing
standards, user experience consistency, and performance requirements.
```

That one prompt is the whole trick. The constitution agent reads the real code and derives the project's principles *from* it, and every later step — specify, plan, tasks, implement — inherits that grounding. A short natural-language feature description at `specify` ("I want a password-protected web console to manage hosted applications...") is enough from there. No extension, no plugin, no special mode: brownfield is just the core process pointed at existing code with a constitution prompt that says "look first." You **can** do brownfield today with nothing but the stock kit and a good prompt — and the walkthrough proves it on a real, enormous codebase.

So what are the brownfield extensions actually for? Not capability — *ergonomics*. They automate the exact prompt-craft that walkthrough did by hand:

```bash
specify extension search brownfield
specify extension add spec-kit-brownfield
```

**Brownfield Bootstrap** auto-discovers an existing architecture and adopts SDD incrementally — so the deep-analysis pass you would otherwise spell out in the constitution prompt is packaged for you. **Time Machine** retroactively applies the workflow feature by feature; **Improve** audits a codebase and writes prioritized spec prompts under `specs/` for the normal lifecycle to pick up. Three authors, three repos, one shared wish — and it is *not* "make the process capable of brownfield," because it already is. It is "stop making me hand-write the analyze-the-codebase prompt every time." That is a subtler and, I think, more interesting signal: the loudest cluster in the catalog is not patching a hole in the core, it is paying down a repeated manual cost the core leaves on you. Feedback for the maintainers all the same, just about *ergonomics and repeatability* rather than *ability*.

## The post-implementation gate the community shipped first — then the core adopted

The reference sequence used to end at `implement`, and the space right after it — *does the code actually match the spec?* — was the single most-cited criticism of the whole tool. The community filled that space first, as extensions. **Fix Findings** is a literal "analyze-fix-reanalyze loop that resolves spec findings until clean"; reconcile-and-verify extensions did the same in their own idioms. Dozens of authors independently converged (the pun is unavoidable) on the same missing step: a convergence loop that runs after implementation and drives the code back toward the artifacts.

Then the core adopted it. **`/speckit.converge`** shipped as an opinionated ninth step — described back in Part 5 — that runs *after* `implement`, assesses the codebase against `spec.md`, `plan.md`, and `tasks.md`, and appends whatever is missing as new tasks; you re-run implement and converge until it reports the feature has actually converged. The sequence and the ordering are the whole point: the community proved the gate as extensions, and *afterward* the maintainers promoted the pattern into a first-class core command. This is the clearest example in the catalog of the dynamic the rest of this post keeps circling — an extension ecosystem functioning as a roadmap, and the core pulling a winner inward. Converge is what that looks like when it actually happens.

That history is also why the review extensions still worth installing are the ones that answer questions converge deliberately does *not*. Converge checks completeness and consistency against the artifacts; it is not a code-quality reviewer. That is where these come in:

**Cleanup** is a post-implementation *quality* gate — it fixes small issues on the spot (the "scout rule"), files tasks for medium ones, and writes up the large ones for you to decide. **Review** runs specialized agents over the finished change for code quality, tests, error handling, and simplification, and, notably, declares itself `read-only` — a report without touching your files. Neither is trying to be converge; they are the taste-and-craft pass that runs after convergence has confirmed the work is merely *complete*.

My favorite in this group is the smallest, and it is the one closest to converge's territory without being redundant. **Verify Tasks** detects *phantom completions* — tasks marked `[X]` in `tasks.md` that have no real implementation behind them. That is a failure mode I flagged in Part 5: a long `implement` run drifts, the agent marks work done that it only half-did, and `tasks.md` lies to you — which would in turn feed converge a false picture. Someone in the community felt that exact pain and shipped a tightly scoped, read-only extension to catch it. That is the catalog working as intended — a specific, unglamorous fix for a specific, real problem, findable by anyone who hits the same wall.

## `taskstoissues` speaks GitHub. Your company speaks something else.

Part 5 mentioned `/speckit.taskstoissues`, the command that turns a task list into trackable GitHub issues. It is GitHub-only, and Part 7's whole enterprise argument was that real organizations live on whatever tracker they already bought. So the community built the adapters: **Jira**, **Linear**, **Azure DevOps**, **Confluence**, **GitHub Projects**, **Trello** — mirror your specs and tasks into the board your team actually stands around.

The most elegant one is a preset, not an extension, and it is worth pausing on because it closes a loop from earlier in the series. The **Jira Issue Tracking** preset does not *add* a command; it `replaces` `speckit.taskstoissues` outright, so the command you already know now creates Jira epics, stories, and tasks instead of GitHub issues:

```bash
specify preset add spec-kit-preset-jira
```

That is exactly the transformative route from Part 6 — a preset reshaping an existing slot rather than bolting a new one beside it — used for something completely mundane and completely useful. Nothing new appears in your command list; the command you had just speaks a different tracker. The abstract argument I made two posts ago has a one-line install behind it.

## The core is silent on cost. The community put a meter on it.

Nothing in the reference process tells you what a run costs. For a hobby project that is fine; for a team burning tokens across dozens of features it is a budgeting problem. So there is a whole cluster answering "what is this costing me": **Cost Tracker** attaches real dollar figures to SDD workflows with per-feature budgets and finance-ready exports; **Token Budget** and **Token Economy** attack the same problem from the input side, compacting artifacts and scoping per-phase reading to spend fewer tokens in the first place.

I read this cluster as the community doing the core's FinOps homework. It is the least glamorous group in the catalog and, for anyone running Spec Kit at organizational scale, probably the highest-leverage.

## And the cluster that is not about code at all

Here is the group that surprised me, and the one that best vindicates Part 6's thesis that Spec Kit is a *kit of processes* rather than one process for software. Several of the most elaborate community contributions have nothing to do with programming:

- **Fiction Book Writing** — a preset that remaps the entire vocabulary of the process. Features become story elements, specs become story briefs, plans become story structures, and tasks become scene-by-scene writing. It ships *26 templates and 34 commands*; this is not a toy.
- **Screenwriting** — the same idea aimed at scripts, with slug lines, act breaks, beat sheets, and export to Fountain and PDF.
- **Game Narrative Writing** — branching dialogue trees, player-agency systems, state machines.

None of these touch a compiler. They are proof that the specify-plan-implement skeleton is really a *general* discipline — intent, refinement, ordered execution — and that "spec" and "task" were always more generic than "software." Part 6 argued that a preset can reshape the reference process until it stops looking stock; a novelist reshaping it into a chapter-planning pipeline is that argument taken to its logical, delightful end.

One preset earns a mention purely because it makes the mechanism visceral: the **Pirate Speak** preset turns every artifact into pirate vocabulary — specs become "Voyage Manifests," plans become "Battle Plans," tasks become "Crew Assignments." It is a joke, but it is an *instructive* joke: if a preset can localize the entire experience into pirate, the terminology layer Part 3 described is real and total, not cosmetic.

## How to actually try one without regretting it

The tour is not much use without the safe way to act on it, so, tying back to Part 7's trust boundary, the sequence I actually follow:

```bash
specify extension search cost        # find candidates across your catalog stack
specify extension info spec-kit-cost # read the metadata: effect, provides, source URL
```

Read the `effect` field first. A `read-only` component produces reports and never edits your files — that is a category you can try with almost no risk, and a lot of the review, visibility, and analysis extensions live there. A `read-write` component modifies things, so the `info` output's repository link is not optional reading. Follow it, skim the source, *then*:

```bash
specify extension add spec-kit-cost
```

The catalog got you a candidate and its provenance in two commands; the one judgment it deliberately cannot make for you is whether the code is safe. That division of labor is the honest one, and it is the whole reason Part 7's `verified` flag means "the entry is well-formed," not "the code is blessed."

## Why I like this design

What I appreciate, stepping back from the individual components, is that the catalog turned out to be a *feedback channel the maintainers get for free*. Every cluster above is the community telling the core team, in the most credible currency there is — working code someone bothered to publish — exactly where the reference process asks the most of you: a smoother brownfield on-ramp (a capability the core already has, but one that costs you a careful constitution prompt each time), a non-GitHub tracker, a cost meter. No survey would have produced a list that honest, because these are the frictions people cared enough about to *fix*. And the loop actually closes: `converge` is a cluster that was already pulled inward — the community's post-implementation reconcile loops became a core command. A healthy extension ecosystem is a roadmap that writes itself, and the best thing the core can do with it is watch which clusters grow and promote the winners, exactly as it did with convergence.

I also like that the ecosystem polices its own risk with vocabulary rather than gatekeeping. The `read-only`/`read-write` distinction, surfaced right in `info`, lets you sort a hundred strangers' extensions into "safe to try" and "read the source first" before you install anything — the same restraint the whole series keeps running into, where the system gives you the information to make a judgment instead of pretending to make it for you.

Which finally brings me back to the layer I have now deferred three times. Standing up a process, tailoring it, sharing it, and — as of this post — *finding* the good ones: that is the whole kit except for one thing, which is making a process run end to end, automatically and repeatably. That is the automation story, and I have run out of excuses to keep postponing it. Next time: workflows — KDD in motion. If there is a particular community component you think I missed, or a workflow you want me to put through its paces, send an email to blog (at) manorrock.com.

---

*Further reading: the [Community Extensions](https://github.com/github/spec-kit/blob/main/docs/community/extensions.md) and [Community Presets](https://github.com/github/spec-kit/blob/main/docs/community/presets.md) lists, the [Community Walkthroughs](https://github.com/github/spec-kit/blob/main/docs/community/walkthroughs.md) (including the stock-process brownfield demos), the [Community Extensions](https://speckit-community.github.io/extensions/) and [Community Presets](https://speckit-community.github.io/presets/) sites, [Part 7: Catalogs](../07/spec_kit_catalogs.html), [Part 6: From Spec to Kit](../06/spec_kit_kit.html), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
