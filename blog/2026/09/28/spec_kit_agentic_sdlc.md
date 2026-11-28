# Spec Kit's Agentic SDLC — No Single Route, No Single Actor

*A reflection on the official guide to [how Spec Kit develops Spec Kit](https://github.github.com/spec-kit/guides/agentic-sdlc.html).*

I have spent much of this series taking Spec Kit apart: [the spec-driven process](../../07/02/spec_kit_process.html), [the workflows](../../07/09/spec_kit_workflows.html), and the layers that let people change both. But describing the parts leaves another question hanging: what does developing Spec Kit actually look like when the work does not arrive in the same shape every time?

The new official guide, [How Spec Kit Develops Spec Kit: An Agentic SDLC](https://github.github.com/spec-kit/guides/agentic-sdlc.html), maps our public repository workflows and a historical feature across planning, requirements, design, development, testing, deployment, and maintenance. Those repository workflows run on GitHub Actions; they are not the installable Spec Kit workflow primitive I covered in Part 9. Read the guide for the evidence and the specifics of what runs where. What stands out to me is not a single route through those stages. It is **how many choices the map has to leave open**: where work enters, which stages it needs, and who or what does the work inside each one.

## The boxes tell you where, not how

The familiar SDLC diagram has arrows from planning to requirements to design and onward. Draw seven boxes, and it is tempting to imagine an issue entering box one and emerging as a release from box seven. Add agents, and the temptation gets stronger: assign one actor to each box and pass the result along.

That is not what our guide describes. A feature request may enter at planning, a change that already exists may need testing, and a bug report may start in maintenance. A stage can overlap another, be revisited, or never be needed for a particular change. The arrows name a familiar path, not a queue everyone must join at the beginning.

But even saying "a feature starts in planning" tells only half the story. A feature issue can carry the initial problem, requirements, and a design discussion in one place. Another change may call for a separate spec and plan. Those are different ways to do requirements and design, not merely different entry points. The guide points to a preserved snapshot of the historical `specify bundle` work as a visible example of the deeper SDD path on Spec Kit itself. Generated `specs/` artifacts are normally gitignored, though, so the absence of those artifacts from another PR does not tell us whether its contributor used SDD.

That is why I would not describe this as "start in the right box, then follow its instructions." First ask what the work needs. Then ask how much structure, interpretation, and verification that particular work calls for. The answer can change even while the stage label stays the same.

## The same stage can change hands

There is another kind of variation inside each box: **who does the work**. Contributors can write or refine requirements in an issue; for substantial work, a contributor can use Spec Kit's SDD commands with an agent. Design can be a human discussion or a deeper planning exercise assisted by an agent. Development can be written by a person, written with an agent's help, or proposed by an agent for a person to review. Those are not competing definitions of the stages. They are ways the same stage can be carried out, depending on the change and the people involved.

Some stages mix all three modes the guide names: **human work**, **agent automation**, and **regular automation**. In testing, an agent may select tests for a bug, deterministic commands run them, contributors provide reproduction evidence, and maintainers judge whether the result addresses the problem. No one actor "owns testing" just because one workflow has `test` in its name. Conversely, the project's release delivery uses conventional Actions and maintainer decisions, not an agent simply because this is an *agentic* SDLC.

Even "agent" is not one execution path. A contributor can invoke a Spec Kit command with an agent or use their own agent; a maintainer can label an issue to start a repository-owned workflow. Those paths have different triggers and provenance. Running one on GitHub Actions does not turn its interpretation into a deterministic script. And because a diff may not reveal when a contributor used an agent, the [contribution policy](https://github.com/github/spec-kit/blob/main/CONTRIBUTING.md#ai-contributions-in-spec-kit) asks contributors to disclose their AI assistance.

The guide also resists turning those choices into a prescription. It distinguishes our feature-assess workflow, which uses Spec Kit's bundled `assess` extension, from our bug workflows, which currently do not use the bundled `bug` extension. Similar-looking work does not imply identical machinery. "We ship a process" and "this is how the repository did this work" are different statements.

## The handoffs depend on context, too

If the boxes do not dictate the method, the arrows cannot dictate the next move. A feature assessment can return `go` without starting SDD or opening a PR. A small fix can go through the ordinary issue and PR process without that assessment. A bug can move through separate assessment, fix, and test workflows, with a maintainer choosing when to request each one. What happens next depends on the evidence, the scope of the change, and the judgment of the people responsible.

For repository-owned agentic workflows, context includes trust. An issue does not launch an agent just because someone filed it; a maintainer chooses when to apply the label. I [wrote recently about an external README that tried to instruct an agent reviewing it](../15/spec_kit_and_the_readme_that_talked_back.html). A maintainer trigger does not make external content trustworthy, but it makes that first handoff deliberate. A proposed bug fix returns as a draft PR for review, not as accepted code. Other teams may choose different gates under different trust models; these are our project's boundaries, not a law of Spec Kit.

## What I like about this approach

I do not think "agentic" gets more credible as the number of stages with agents goes up. It gets more credible when we can explain why an agent is helping *here*, what evidence it should return, and who judges that evidence. Sometimes the answer is a contributor working directly. Sometimes it is an agent interpreting a request. Sometimes it is a repeatable check or release action that does not need an agent at all.

What I like is that this leaves room to tell the truth about what we can see and what we cannot. We can point to a preserved SDD example without treating the public artifacts as a complete count of who used the process. We can say an agent helped select tests without implying that it replaced the tests, or that the maintainer who approved a workflow personally wrote the agent's output. The stages give us a shared vocabulary; they do not erase the differences between the people, tools, and decisions inside them.

That is the reading of [the official guide](https://github.github.com/spec-kit/guides/agentic-sdlc.html) I would take into another project. Do not copy a seven-stage pipeline or assign a permanent actor to every box. Ask where this work begins, what this stage needs *in this context*, which ways of doing it are appropriate, and what would justify the next handoff. The map gives us vocabulary for those decisions. It does not make them for us.

If your team has found a different way through a familiar stage, or put a human gate somewhere we have not, send an email to blog (at) manorrock.com. I would like to know what changed the choice.

---

*Further reading: [How Spec Kit Develops Spec Kit: An Agentic SDLC](https://github.github.com/spec-kit/guides/agentic-sdlc.html), [Spec Kit, Part 5: The Spec-Driven Process](../../07/02/spec_kit_process.html), [Spec Kit, Part 9: Workflows](../../07/09/spec_kit_workflows.html), and [Spec Kit and the README That Talked Back](../15/spec_kit_and_the_readme_that_talked_back.html).*
