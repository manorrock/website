# A Year of Spec Kit — Notes from Washington

*A first-year reflection, written after taking a week off and walking through places where a country preserves its founding ideas, its achievements, and the record of how both evolved.*

I took last week off and went to Washington, D.C. I visited several of the Smithsonian museums, stood in front of the founding documents at the National Archives, walked up the steps of the Lincoln Memorial, and did the other things you do in a city where so much of the country's memory has been given a building and an address.

Just before the trip, [Spec Kit](https://github.com/github/spec-kit) turned one on August 21st and shipped 1.0.0. I had already written the proper [birthday post](../../08/21/spec_kit_turns_one.html): the first commit, the round version number, the argument that adaptability matters more now than pretending a `1` can freeze a project in place. Then I closed the laptop and spent a week looking at what happens when people take ideas seriously enough to preserve the record.

## Write down what you mean

The thing that struck me first was not the language on the parchment. It was the existence of the parchment at all.

A country is not sustained by a document alone. The Constitution did not by itself build roads, resolve every argument, or guarantee that the experiment would succeed. What it did was establish a durable framework: one people could return to, interpret, challenge, amend, and measure themselves against. It did not eliminate disagreement. It gave disagreement a shared foundation.

That is the part that feels closest to Spec-Driven Development. A specification is not valuable because it predicts every line of code. It is valuable because it moves intent out of one person's head and gives everyone doing the work — human or agent — the same thing to argue with. The spec says what the system is meant to do. The plan says how we currently think we can do it. The tasks make the work inspectable. None of them are the implementation, and none should pretend to be.

Spec Kit's project constitution exists for the same reason at a much smaller and less consequential scale. It records the principles that should survive individual features: test first, keep dependencies bounded, preserve a trust boundary, or whatever a particular project has decided it refuses to trade away. The point is not to make those principles sacred. The point is to make changing them a visible decision instead of an accidental side effect of the latest prompt.

The National Archives makes this physical. The words are faded. The document itself has required care. Its interpretation has evolved, and its amendment process acknowledges that durability and change can coexist. That may be the cleanest answer I have found to what a 1.0.0 should mean now: preserve the intent, keep the record, and change the machinery when the machinery needs changing.

## The museums are full of iterations

The Smithsonian museums tell a different part of the story. They are full of finished things, naturally, but look closely and the interesting story is rarely the finished object by itself. It is the sequence around it: the instrument that was not precise enough, the material that failed, the prototype that made the next prototype possible, the ordinary tool without which the famous invention would have remained an idea.

American ingenuity looks tidy when reduced to a plaque. It was not tidy while it was happening.

Software gets flattened the same way. We remember the release and lose the dead ends, the decisions, and the constraints that made the release possible. An AI coding agent makes that loss easier because it can produce so much code so quickly. The output arrives polished enough to look inevitable. It was not. There were assumptions in the prompt, alternatives in the plan, and tradeoffs someone made — even if that someone made them silently in the space between two generated tokens.

This is why I keep coming back to durable artifacts as Spec Kit's real product. `spec.md`, `plan.md`, and `tasks.md` are not ceremony around the code. They are the exhibit labels the next contributor needs in order to understand why the thing has this shape. They let someone see not only what survived, but what the team believed when it chose a direction.

That record does not have to be grand. My first commit as lead maintainer fixed a path typo. While I was away, Spec Kit kept moving: catalog additions, tighter input validation, a Windows encoding fix, and a correction to piped event input. That is what a living project actually looks like: not one heroic invention, but many people noticing one rough edge at a time and deciding to leave it smoother than they found it.

## A principle is not its fulfillment

At the Lincoln Memorial, the words are enormous. They are carved into stone, lifted above the visitor, and arranged to outlast everyone reading them. Their permanence asks each visitor to reckon with them.

But permanence creates its own temptation: confusing the statement with the achievement. Writing down a principle does not mean you have lived up to it. Engraving it certainly does not.

Looking out from the Lincoln Memorial, I could see that the Reflecting Pool was under renovation. The place built for reflection was itself a work in progress. That felt more honest than a perfect image in the water. A retrospective can become a polished duplicate of the release notes, or it can expose where intention and reality still diverge. Looking back at Spec Kit's first year matters only if the reflection helps us see what remains unfinished.

Software teams make the same mistake in miniature. A careful specification can feel like progress. A beautiful plan can feel like completion. A constitution can say "test first" while the implementation quietly does something else. This is the lesson from the [Vidocq SDD experiment](../../07/29/spec_kit_sdd_in_practice.html) that I think matters most: writing down the rules is not enough; you have to audit whether the work follows them.

That is why `/speckit.analyze` and `/speckit.converge` matter. One checks the artifacts against one another before implementation. The other checks the implementation against the artifacts afterward and sends the gaps back into the work. Neither assumes that a declared intention fulfilled itself. A spec is a promise; tests and convergence are how you find out whether the code kept it.

The future of Spec Kit has to keep pushing in that direction. Better specifications are useful. Better evidence that an implementation matches them is more useful. The kit should help teams move from *we wrote down what we wanted* to *we can show where the result does and does not conform*. The distance between those sentences is where trust lives.

## The project is more community than monument

Washington can make history look as though it was made by the handful of people whose names fit on buildings. The museums correct that impression if you let them. Progress is crowded. It comes from inventors, craftspeople, organizers, maintainers, operators, critics, and people whose names never reached the plaque.

Spec Kit's first year has the same shape. Den Delimarsky and John Lam conceived it and gave it its recognizable foundation: the Specify CLI, project constitutions, agent-specific scaffolding, and the specify → plan → tasks → implement sequence. In January, GitHub trusted me with the opportunity to lead it and has backed me in doing that work. I would not be here without that trust and support, and I do not take either lightly. But the project that exists now is the work of at least 270 contributors and a community that keeps building beyond anything one maintainer — or one company — would have put on a roadmap.

As I write this, Spec Kit supports 38 coding-agent integrations, and the community catalogs contain 161 extensions and 34 presets. The repository has passed 132,000 stars. Those numbers are gratifying, but the shape behind them matters more. The modular extension system began as a community contribution. People used the primitives to build brownfield processes, architecture gates, governance checks, tracker integrations, assessment funnels, bug-fixing workflows, and processes that have nothing to do with software at all. The community did not wait for permission to decide what the kit could become.

That is a particularly American kind of ingenuity — not because it belongs only here, but because the version preserved in these museums is so often practical, distributed, and compositional. Someone makes a part. Someone else adapts it. A third person finds a use the first two did not imagine. The interesting system emerges from the connections.

It is also the design instinct I have praised throughout the Spec Kit series: **compose, don't accrete.** Extensions add capabilities. Presets reshape behavior. Integrations connect agents. Workflows automate processes. Steps provide reusable actions. Bundles assemble those pieces without becoming a second runtime. The architecture leaves room for people we have not met to solve problems we have not predicted.

## What the second year can hold

Spec Kit quickly moved beyond its birthday release: 1.0.1 landed before the day was over. The round number marked a year, not a finish line.

The next year should be less about adding monuments and more about improving the institutions around the work. The brownfield on-ramp still asks too much of newcomers. The seams between powerful primitives can still disappear further. Community components need clearer provenance and a trust boundary that is easier to inspect before anything runs with local privileges. Specifications need stronger paths to executable evidence. Documentation has to ship as part of the feature rather than arrive as its apology.

And the community should continue to surprise us. Spec Kit began as a way to bring discipline to AI-assisted software development. Its deeper idea — state intent, refine it, plan deliberately, execute in inspectable steps, and compare the result with the intent — is not limited to software. I still think the core should remain narrowly excellent rather than chase every possible use, but the kit should keep enough open space for other communities to test the boundary.

The American experiment is an overlarge metaphor for a one-year-old open source project, and I do not want to strain it. Spec Kit is a tool, not a country, and a Markdown constitution is not *the* Constitution. But after a week in Washington, I keep returning to one useful similarity: the future does not come from getting the first document perfect. It comes from writing intent down clearly enough that other people can participate, preserving enough of the record that they can learn, and building a process capable of correction when reality exposes the gaps.

Spec Kit, though, is not merely an American experiment. It is a global one. Its contributors, users, extensions, presets, and ideas come from countries around the world, spanning different languages, cultures, industries, and institutions. They do not all define good software in the same way — and that is precisely why the kit must remain open, composable, and willing to change. The experiment succeeds only if its process can carry more than the assumptions of the people who started it.

That is what I hope the second year holds. Not a finished kit. A more legible one. Not a process that eliminates disagreement. A process that gives disagreement somewhere productive to go. Not a monument to what shipped in year one, but a workshop sturdy enough for people everywhere to build what year two has not imagined yet.

I went to Washington expecting a week away from Spec Kit. Instead, I came home with a clearer way to think about its first year: write down what you mean, preserve the record, expect revision, and leave room for more hands. The lesson came into focus for me in Washington, but it belongs to the whole world.

So this is the call: bring your language, your constraints, your way of working, and the problem your corner of the world sees differently. Add the integration your agent needs. Write the extension. Shape the preset. Automate the process with a workflow. Publish your bundle. Challenge the defaults. File the uncomfortable issue. Contribute the missing documentation. Show us where the kit assumes too much and where it does too little. Do not wait for the roadmap — the community is the roadmap.

Year one proved that Spec Kit can grow. Year two is where we decide how far it can reach, how openly it can evolve, and how many communities can make it their own. Come join the Spec Kit experiment: build it with us, and put it to work on whatever your corner of the world needs.

As a side note, I came home to more than 1,000 emails from GitHub about Spec Kit — pull requests, issues, reviews, and discussions — waiting for me. Apparently, while I was away reflecting on the project's first year, the community was already busy building its second. I had better start reading.

If you have used Spec Kit during its first year — or have an idea for what its second year should make possible — send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit repository](https://github.com/github/spec-kit), its [project history](https://github.com/github/spec-kit/blob/main/docs/history.md) and [Community overview](https://github.com/github/spec-kit/blob/main/docs/community/overview.md), the [1.0.1 release](https://github.com/github/spec-kit/releases/tag/v1.0.1), [Spec Kit Turns One](../../08/21/spec_kit_turns_one.html), [Six Months of Spec Kit](../../07/22/six_months_leading_spec_kit.html), [Spec Kit Learnings from Vidocq](../../07/29/spec_kit_sdd_in_practice.html), and [Part 12: The Road Ahead](../../07/14/spec_kit_road_ahead.html).*
