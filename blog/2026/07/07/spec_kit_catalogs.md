# Spec Kit, Part 7: Catalogs — How the Kit Becomes a Community

*The seventh entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

At the end of [Part 6](../06/spec_kit_kit.html) I made a claim in passing and then walked right past it. Explaining how the Spec became a Kit, I wrote that "the catalogs are what make those processes *shareable* rather than local curiosities — a bug workflow, a compliance-flavored core, a migration stack, all discoverable and installable the same way." That sentence did a lot of quiet work. Every primitive I have spent six posts describing — extensions, presets, integrations, workflows, steps, bundles — is only useful to *you* if you can find the one someone else built and pull it into your project. The catalog is the machinery that makes that possible, and it is worth a post of its own before I highlight the actual things people have built with it.

The reason to slow down here is that "discoverable and installable the same way" is easy to nod along to and easy to underestimate. It is not a website with a list of links. It is a resolution system — the same kind of priority-ordered stack I described for presets in Part 3, applied to *where components come from* rather than *how they layer*. Once you see it that way, the catalog stops looking like a directory and starts looking like the seam where the outside world plugs into your local setup — and, just as importantly, the seam where an *organization* decides which parts of that outside world its engineers are actually allowed to reach. It is at once the community's front door and the enterprise's governance layer, and those turn out to be the same mechanism.

## What a catalog actually is

A **catalog** is a manifest — a JSON file — that lists installable components of one kind, each with the metadata needed to find, evaluate, and fetch it. Every primitive in Spec Kit has its own catalog type: extensions have one, presets have one, integrations, workflows, and steps each have one, and bundles have one too. They are parallel systems with a shared shape, which is why the commands to work with them rhyme across primitives.

Concretely, each primitive ships two catalogs. There is the **official** catalog (`catalog.json`) — the components the Spec Kit maintainers publish and stand behind — and a **community** catalog (`catalog.community.json`) — the far larger, contributor-supplied list. A single entry in the extensions community catalog looks like this (trimmed):

```json
{
  "name": "Cleanup Extension",
  "id": "spec-kit-cleanup",
  "description": "Post-implementation quality gate that reviews changes...",
  "author": "dsrednicki",
  "version": "1.0.0",
  "download_url": "https://.../spec-kit-cleanup.zip",
  "repository": "https://github.com/dsrednicki/spec-kit-cleanup",
  "license": "MIT",
  "category": "code",
  "effect": "read-write",
  "requires": [],
  "provides": ["speckit.cleanup"],
  "tags": ["quality", "review"],
  "verified": false
}
```

Everything the CLI needs to reason about an extension before installing it is in that record: what it provides, what it requires, whether it only reads your files or also writes them, where the source lives so you can read it yourself, and where to fetch the artifact. `search` reads these entries; `info` prints one; `add` fetches the `download_url`. The catalog is the index; the component lives at the URL.

## The stack, not the file

The interesting part is that there is never just one catalog. Like presets, catalogs resolve through a **priority-ordered stack**, and understanding that stack is understanding the whole feature. For extensions the resolution order is:

1. **Environment variable** — `SPECKIT_CATALOG_URL` overrides everything else.
2. **Project config** — `.specify/extension-catalogs.yml`, checked into your repo.
3. **User config** — `~/.specify/extension-catalogs.yml`, personal to your machine.
4. **Built-in defaults** — the official catalog plus the community catalog.

Lower priority numbers win, and the first match takes precedence — exactly the discipline Part 3 laid out for presets, now governing *sources* instead of overrides. The payoff, which I want to come back to in full because it is the whole enterprise story, is that whoever controls project config controls what the team's `search` and `add` reach first — before anything ever falls through to the community list.

Each source in the stack also carries an install policy, and this is the detail I find most quietly clever. A source is either **`install-allowed`** or **`discovery-only`**. An install-allowed source behaves how you would expect: you can `search` it and `add` from it. A discovery-only source shows up in `search` and `info` but *refuses* installation. That single flag lets a team publish an internal "here's what exists, but don't pull it yet" catalog — a staging list, a curated-but-unapproved set — that is visible without being live. Discoverability and installability turn out to be separable, and the catalog treats them as two different permissions rather than one.

## The same three verbs, whatever the primitive

Because every catalog shares a shape, the commands to drive them share a vocabulary. For extensions:

```bash
specify extension search [query]        # find components across the active stack
specify extension info <name>           # inspect one entry's metadata
specify extension add <name>            # install from the catalog...
specify extension add --from <url>      # ...or straight from a URL, bypassing catalogs
```

Swap `extension` for `preset`, `bundle`, or `workflow` and the shape holds — `search`, an inspect verb, and an install verb, over the same kind of priority stack. Managing the stack itself is just as uniform:

```bash
specify extension catalog list                         # show active sources + priorities
specify extension catalog add <url> --name my-org --priority 5
specify extension catalog remove my-org
```

`add --from <url>` is the escape hatch worth calling out: you are never *required* to go through a catalog at all. If a colleague sends you a link to an extension, you can install it directly. The catalog is the convenience layer that makes a whole ecosystem discoverable; it is not a gate you must pass through. That is consistent with the design philosophy the whole series keeps running into — the primitives compose, and the higher-level convenience never becomes a hard dependency of the lower-level capability.

## Where the community actually lives

Now the part that makes "catalog" mean "community." That built-in community catalog is not a handful of blessed samples. As I write this, the extensions community catalog carries **124** entries and the presets community catalog **23**, contributed by dozens of authors — a bug-fixing workflow here, an Azure DevOps sync there, an architecture-review gate, a cost tracker, a brownfield bootstrapper for existing codebases. These are real, installable components authored by people who are not on the Spec Kit team, distributed through the exact same `search`/`info`/`add` path as the official ones.

That immediately raises a trust question, and Spec Kit answers it with a deliberately sharp line. The maintainers verify that a community catalog **entry** is complete and correctly formatted — that the metadata is valid, the URLs resolve, the schema is right. They do **not** review, audit, endorse, or support the component *code* itself. The `verified` flag on an entry is about the catalog record, not a security blessing. The documentation is blunt about the implication: read the source before you install, and use community components at your own discretion. I think this honesty is a feature, not a hedge. A catalog that pretended to vouch for 124 third-party codebases would be lying; one that clearly separates "this entry is well-formed" from "this code is safe" tells you exactly which guarantee you are getting and leaves the security judgment where it belongs — with the person about to run it.

To make that judgment easier, the community entries are also surfaced on browsable websites — the [Community Extensions](https://speckit-community.github.io/extensions/) and [Community Presets](https://speckit-community.github.io/presets/) sites — so you can skim, sort, and follow the repository link *before* you ever touch the CLI. The catalog is the machine-readable index; those sites are the human-readable front door onto the same data.

## The real answer to that trust problem: host your own

This is where the feature earns its keep, and where a tour of the community catalog risks selling it short. "Read the source and use community components at your own discretion" is honest advice for an individual, but it is a non-starter as *organizational policy*. No security-conscious company is going to tell every engineer to personally audit a third-party extension before typing `add`. They need the opposite: a curated, vetted set that has *already* been reviewed, and a way to make that set the one their teams actually reach. Catalogs are exactly that mechanism — and it is arguably the most important thing they do, not a footnote to the community story.

The moving parts I described in the abstract above are, put together, a complete internal-distribution system. An organization stands up its own catalog — a JSON file listing only the extensions, presets, bundles, and workflows its security and platform teams have vetted, each pinned to an approved version and pointing at an artifact the company hosts itself (an internal URL, an artifact registry, a private repo). It commits that catalog's source into the project at a high priority:

```bash
specify extension catalog add https://catalog.acme-corp.internal/extensions.json \
  --name acme-approved --priority 1
```

From that moment, `specify extension search` and `add` resolve the company's vetted list *first*, because priority 1 beats the built-in community default. Engineers keep using the identical commands they already know — nothing about their workflow changes — but "what's installable" has quietly become "what the org approved." Governance without a gate in anyone's way.

The `install-allowed` / `discovery-only` split is what turns this from a single list into an actual review pipeline. A platform team can run a `discovery-only` catalog of candidate components — things engineers can *see* and evaluate but not yet install — alongside an `install-allowed` catalog of the approved set. Promotion from "proposed" to "approved" is just moving an entry between the two. And because catalogs are checked into project config, the approved set is versioned, reviewable, and diffable like any other file in the repo: a pull request adds an extension to the roster, security signs off in the review, and every engineer picks it up on their next pull. The vetting lives in Git history, not in a wiki no one reads.

Two more properties make this genuinely enterprise-grade rather than a clever hack. First, nothing forces a company onto the public community catalog at all — the built-in defaults sit at the *bottom* of the stack, so an org catalog at priority 1 shadows them completely, and a company that wants a fully closed garden can point its engineers only at internal sources. Second, because every primitive shares this catalog machinery, one governance model covers *all* of them: the same priority-and-policy story that vets extensions vets presets, bundles, and workflows too. A company does not need one system for approved extensions and another for approved presets — it is catalogs, all the way down.

## Publishing into it

The symmetry that makes catalogs a community rather than a store is that contributing is the same shape as consuming. Distribution of a component is two steps: host the built artifact somewhere, and add an entry pointing at it to a catalog. That is true at both scales — the internal org catalog I just described and Spec Kit's public community list are the *same* act of publishing, differing only in who the audience is and where the file lives. For something you want the world to have, the "catalog" is Spec Kit's community list, and you get an entry into it through a submission — the project provides issue templates (a Bundle Submission form, for instance) precisely so the metadata and install evidence a reviewer needs are captured up front, and so the "we verified the *entry*" promise can actually be kept.

Either way, the mechanism is identical to the one you already use to install. You do not learn a separate publishing system; you write the same kind of record you have been reading in `info` output, and you point it at your artifact. Consuming and producing are the same activity seen from two sides, which is exactly the property you want if you are trying to grow an ecosystem rather than a catalog of officially-sanctioned parts.

## Why I like this design

What I appreciate about catalogs is that they did not invent a new idea to solve distribution — they reused the resolution-stack idea the toolkit already leaned on for presets and pointed it at *sources*. Priority ordering, first-match-wins, project-over-user-over-built-in: the same mental model the rest of Spec Kit runs on, now deciding where components come from. That reuse is why the whole thing feels learnable. Once you understand one stack you understand all of them, and the `install-allowed`/`discovery-only` split adds just enough expressiveness to cover the real cases without a pile of new concepts.

The part that most deserves emphasis is that the *same* small mechanism serves two audiences that usually demand separate systems. For a hobbyist it is a way to `add` a stranger's clever extension in one command; for an enterprise it is a governance layer — a vetted, versioned, review-gated internal roster that shadows the public list entirely. Most tools would have shipped a breezy community registry and then, two years later, bolted on an "enterprise catalog" product with its own concepts. Spec Kit gets both from one priority-ordered stack, because the primitive was general enough that "the company's approved set" and "the world's shared set" are just two sources at different priorities. The open ecosystem and the closed garden run on identical machinery.

And it is the layer that finally makes the previous six posts matter to someone other than the person who wrote a component. A brilliant extension nobody can find is a local curiosity; the catalog is what turns it into something a stranger can `add` in one command. The design keeps the trust boundary honest, keeps publishing and consuming symmetric, and keeps the convenience optional. That is a lot of restraint for a feature that could easily have become a walled marketplace.

Which sets up where the series goes next. Now that the *mechanism* for finding community work is on the table, the obvious follow-up is the work itself: with well over a hundred community extensions and presets out there, some of them are genuinely worth your attention, and picking through them is a different kind of post than picking through machinery. Next time I want to do exactly that — a guided tour of community contributions I think are worth installing, not just the system that lists them. (The automation layer I keep deferring — turning a process into a repeatable run — is still coming; I have simply decided the community deserves its spotlight first.) If there is a particular community component you would like me to look at, send an email to blog (at) manorrock.com.

---

*Further reading: the [Community Extensions](https://github.com/github/spec-kit/blob/main/docs/community/extensions.md) and [Community Presets](https://github.com/github/spec-kit/blob/main/docs/community/presets.md) lists, the [Extensions reference](https://github.com/github/spec-kit/blob/main/docs/reference/extensions.md) (catalog management section), [Part 6: From Spec to Kit](../06/spec_kit_kit.html), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
