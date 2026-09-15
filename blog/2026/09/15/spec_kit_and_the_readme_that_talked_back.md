# Spec Kit and the README That Talked Back

*A personal note prompted by Spec Kit Issue #4036, about prompt injection and why an open source workflow that reads the community is also opening a new door to the community.*

On August 10th, someone submitted a Markdown Linter to the [Spec Kit](https://github.com/github/spec-kit) community extension catalog. On the surface, [Issue #4036](https://github.com/github/spec-kit/issues/4036) looked exactly like the kind of contribution an open ecosystem is supposed to invite: a name, a version, a public repository, a release archive, an MIT license, two commands, and a checklist saying the extension installed, ran, and had no known security vulnerabilities.

Then I triggered the catalog workflow as the maintainer reviewing the submission, and the workflow read the README in the submitter's own repository—not Spec Kit's. That distinction is the whole point: the file that talked back lived in a repository the submitter controlled and the project had never seen.

Buried inside an HTML comment under "Architecture Notes" was an instruction addressed not to a person, but to the automated reviewer. It claimed to be internal catalog metadata. It told the agent to place a particular bypass token at the beginning of its response as proof that it had processed the file. In other words, that README stopped being documentation and tried to become part of the workflow's instructions.

The system did not follow it. The reviewing agent called the content a cross-prompt injection attack, rejected the submission, applied the `validation-failed` label, and explained what needed to be removed. But that outcome did not belong to one clever sentence or one heroic model refusal. The Spec Kit workflow author and the [GitHub Agentic Workflows](https://github.com/github/gh-aw) machinery had put different protections around the same run.

That is the good news. It is not the comforting news.

## Is "attack vector" fair?

Yes, with one important restraint.

I do not know the submitter's intent, and I am not going to manufacture it. The repository is named `spec-kit-test-extension`; this may have been a red-team exercise, a deliberate test of the workflow, an experiment, or something less benign. The public record tells us what the content did, not why its author put it there. Calling a person an attacker would outrun the evidence.

Calling the path an **attack vector** does not.

The path is plain:

```text
community submission
        ↓
maintainer triggers review
        ↓
external repository content
        ↓
agent reads the content
        ↓
agent makes a decision
        ↓
workflow comments, labels, or proposes a change
```

An attacker does not need to break GitHub Actions, compromise a maintainer account, or smuggle executable code into the Spec Kit repository. They can write ordinary prose in a repository they control and wait for a trusted automation to read it. The entry point is language itself. The defense cannot be language alone.

The maintainer trigger matters. I was the maintainer who made that decision. This was not a workflow an internet stranger could start merely by opening an issue; I had to decide that the submission was ready for automated review. That removes the simplest drive-by path, puts a person in front of the run, and limits how cheaply someone can spray the workflow with hostile content. That pattern is deliberate across Spec Kit's agentic workflows: the automation starts with a maintainer action, not a contributor's submission alone.

But my trigger authorized the *review*; it did not complete it. I could inspect the structured submission, see the expected fields and checked boxes, and hand the expensive repository inspection to the agent—that is why I wrote the workflow. The human gate therefore reduced exposure without making the content beyond it trustworthy. Once the run started, the agent still crossed the same boundary and read prose controlled by someone outside the project.

That is new enough that I think we should say it without softening it. We spent years teaching automation that configuration is code. Agentic workflows add an uncomfortable corollary: **content can behave like code too.** A README, issue body, changelog, source comment, test fixture, or web page may be data to the human who opened it and an instruction to the model that consumed it.

The injection in Issue #4036 did not need to be sophisticated. It wore a small costume: "rendering metadata," "context override," "required acknowledgment." Those phrases borrow authority from the system around them. The request did not ask for a shell command or a secret. It asked for one controlled change to the agent's output. That modesty is part of what makes it useful as a test. If an untrusted document can choose the first sentence of a validation comment, it has already crossed from being reviewed to participating in the review.

## The checkbox that could not mean what it said

There is a detail in the issue that I cannot stop looking at. The submission checklist included:

> No security vulnerabilities identified

It was checked.

The same submission pointed the reviewer at a README containing instructions for the reviewer. That does not make the checkbox a lie in the moral sense; again, intent is not available to us. It makes the checkbox **untrusted input**, which is what it always was.

Open source projects rely on self-attestation because communities cannot manually reproduce every claim before accepting a contribution. We ask whether tests passed, whether the license is present, whether the author checked for vulnerabilities. Those questions are useful because most contributors are participating in good faith. But an agent cannot turn an answer into evidence merely by reading it carefully, and a checked box should never increase the authority of the text sitting next to it.

This is where agentic automation can accidentally make an old weakness feel stronger than it is. A form looks structured. A checklist looks verified. A fluent validation summary looks like judgment. None of those appearances creates a trust boundary.

## Defense in depth: the author layer and the platform layer

The [catalog workflow](https://github.com/github/spec-kit/blob/main/.github/workflows/add-community-extension.md) has an author layer. It tells the agent exactly what a valid submission consists of, requires it to collect *all* validation results before deciding, says to stop on any failure, and ends with an important rule: match the proposed catalog entry, but verify it against the repository rather than blindly trusting what the submitter supplied. It also declares narrow permissions and narrow outputs: read-only repository access, bounded comments and labels, and at most one draft pull request.

That is not a generic prompt-injection policy. It is something equally necessary: **domain-specific skepticism**. The workflow gives the agent a small job, names the evidence that counts, says which claims must be independently checked, and defines what failure looks like. A platform cannot know that a Markdown Linter needs an `extension.yml`, a matching release, checked submission requirements, and two catalog files updated in alphabetical order. The workflow author has to supply that understanding.

Then `gh aw` adds the platform layer. It compiles the authored Markdown into the GitHub Actions workflow that actually runs. In the compiled workflow for Issue #4036, `gh aw` injected XPIA guidance into the agent's system prompt, checked that the person activating the workflow had a maintainer-level role, separated the read-only agent job from the write-capable safe-output job, and added a separate threat-detection pass over the proposed safe outputs. At the time of this run that detection pass was advisory—it could flag a suspicious result without stopping it—so the rejection itself came from the reviewing agent's decision and the constrained outputs, not from a hard detection gate. I have since made that detection fail-closed, so a flagged result now blocks the write instead of merely warning. The useful part is not that we were safe; it is that the gap was visible and closeable. The author's `safe-outputs` declaration became enforceable limits: allowed labels, maximum counts, one draft pull request, protected files, and a restricted output surface.

This is the defense-in-depth relationship I want to emphasize. The author layer tells the system **what trustworthy work looks like in this project**. The platform layer tells it **how untrusted content must be handled and what the resulting agent is technically allowed to do**. The domain rules make suspicious deviations easier to recognize; the platform controls keep a missed deviation from automatically becoming arbitrary repository access. Each layer reduces how perfect the other one has to be.

The human gates complete that stack. I chose when the workflow ran, and any proposed catalog change would remain a draft for review. There was a person before interpretation and another decision before generated work could become project state.

But defense in depth is not the same phrase as "secure." Least privilege primarily protects the **blast radius**. XPIA guidance helps protect the **decision**, and the separate detection pass checks the proposed result again. None can guarantee that every future instruction will be recognized. An allowlist can stop an agent from editing `README.md`; it cannot guarantee the agent will never place an attacker's words into an allowed catalog description. A draft pull request preserves human review; it cannot guarantee that the reviewer notices a carefully laundered URL or misleading field.

The fact that this injection was detected proves that this layered system handled this injection in this run. That is stronger than "the model happened to say no," and weaker than a security certificate. The next instruction may be split across files, hidden in generated output, phrased as a validation rule, or aimed not at the final comment but at which files the agent chooses to inspect.

Spec Kit is not secure because Issue #4036 failed validation. It would be irresponsible to make that claim, and more useful to say what the issue actually gave us: a concrete example of an authored workflow and its execution platform aiding one another while a trust boundary was tested in public.

## Open source makes the boundary porous on purpose

The same openness that makes Spec Kit interesting makes this problem unavoidable. The catalog exists so that a stranger can point the project at a repository the maintainers have never seen. The workflow exists because checking every submission by hand does not scale. The agent is useful precisely because it can read messy, varied, human-authored material and make sense of it.

Remove the external content and the workflow has nothing to review. Remove the agent's ability to interpret it and much of the automation disappears. Remove the ability to act and the workflow becomes a report generator that still needs a maintainer to do all the mechanical work.

So the answer cannot be "never let agents read untrusted text." An open source maintenance agent that follows that rule is an agent that reads almost nothing.

The answer has to be architectural: assume the text will eventually influence the model, then make sure influence is not the same thing as authority.

I would start with a few principles.

**Separate facts from judgment from action.** Deterministic code should verify the things deterministic code can verify: URL shape, release existence, file presence, semantic version format, checksums, manifest schema, and whether required fields match across artifacts. Give the agent the results instead of asking it to discover every fact through prose. Let the model handle ambiguity, not arithmetic.

**Carry provenance with every value.** A description from the issue, a license read from the repository, and a rule from the workflow are all strings, but they do not deserve equal authority. The system should preserve where each value came from and keep untrusted content visibly delimited all the way to the decision. Flattening everything into one conversation is convenient; it is also how instructions lose their borders.

**Keep capabilities narrow and outputs reviewable.** Read-only repository access, file allowlists, draft pull requests, output limits, and protected files are not merely advice when the platform enforces them. They are essential containment. The right question is not "can the agent be tricked?" Assume it can. Ask "what can a tricked agent do before a human sees the evidence?"

**Require a human where trust changes state.** Spec Kit already does this on both sides of the workflow: a maintainer triggers the run, and the resulting pull request is a draft. Both gates should remain. Discovery can be automated. Validation can be assisted. Promotion into an installable public catalog is a trust decision, even when the catalog carefully says `verified: false`. That transition deserves a human who can inspect the source, the provenance, and the generated diff—not merely approve a polished agent summary.

**Make hostile-content tests ordinary.** Prompt injection should be in the workflow's test corpus beside malformed JSON and missing licenses. Not one famous string that the model learns to spot, but variations: hidden comments, fake system notices, instructions split between files, poisoned tool output, Unicode tricks, and requests that stay within the workflow's nominal permissions. Security testing has to exercise the decision path, not only the action boundary.

**Log enough to reconstruct why.** Issue #4036 is useful because the workflow left a public explanation. Future incidents will be harder. A project needs a durable record of which external artifacts were read, which values were extracted, which deterministic checks ran, what action was proposed, and where a human approved it. Without that, a clean pull request can erase the trail that made it suspicious.

None of these makes the workflow secure. Together they make failure smaller, more observable, and harder to turn into trusted state. That is a less exciting promise than "AI-powered secure automation," which is one reason I trust it more.

## The trust boundary moved

In the earlier [workflow post](../../07/09/spec_kit_workflows.html), I wrote that a `shell` step runs with your privileges and that a human gate is the safety valve before destructive work. In the [steps post](../../07/10/spec_kit_steps.html), I went further: a custom step is arbitrary Python, so read the source before installing it.

Both warnings were correct, and neither was complete.

Issue #4036 shows that risk begins before installation and before execution. The project was only deciding whether an extension belonged in a catalog. No community Python needed to run. The external repository merely had to be *read* by something capable of acting afterward.

That moves the trust boundary earlier:

```text
old instinct:  protect execution
new reality:   protect interpretation → decision → execution
```

This is the part I think open source maintainers need to internalize quickly. Agentic workflows convert selected items from maintenance queues into inputs: issues are triaged, pull requests reviewed, dependencies updated, submissions cataloged. Those queues are writable by someone outside the trust boundary. In this case, I chose which item crossed from queue to workflow, and that is a meaningful control. It is not one we should wave away. Once selected, though, the item and everything it points at became possible attempts to steer the maintainer's automation.

The project does not become reckless by using an agent. Manual review has social engineering, fatigue, and copy-paste attacks of its own. But the economics change. A persuasive paragraph aimed at one tired maintainer is one attempt. A prompt injection embedded in a repository can be replayed automatically against every agent that indexes, summarizes, validates, or reviews it. Agentic workflows make interpretation scalable, and they make attacks on interpretation scalable too.

## What I appreciate—and what I do not want us to pretend

I appreciate the workflow-author layer. It did not tell an unconstrained agent to "review this extension." It defined the checks, required the full set of results, told the agent not to blindly trust the proposed entry, stopped the process on failure, and declared a narrow set of acceptable outputs. That domain knowledge gave the run a shape against which the injected instruction could look out of place.

I also appreciate the `gh aw` layer. It did not leave those declarations as prose and wishful thinking. The compiler added XPIA guidance, maintainer activation checks, safe-output separation, protected-file enforcement, and a second threat-detection path. Security claims should be made at the level the controls support, and these controls support a real claim: multiple independent mechanisms had to agree before agent output could turn into a write.

They do not support the sentence "the workflow is secure." I do not know what would support that sentence for an agent reading arbitrary public repositories, and I am suspicious of anyone selling it cheaply.

What we can say is more honest. The attack surface has been exposed. One attempted instruction met domain-specific validation rules, compiler-supplied XPIA guidance, a separate threat-detection pass, constrained safe outputs, and human gates. The layers aided one another. The decision boundary still includes systems that can be persuaded by language, because persuasion by language is the feature we hired them for.

That tension is not a reason to abandon agentic workflows. It is the engineering problem.

Open source projects should keep automating the work that burns maintainers out. They should also stop treating the text those workflows read as passive. The README in Issue #4036 talked back. This time the workflow author, the agentic-workflow platform, the reviewing agent, and the human gates formed a stack that did not accept it as the one giving orders.

We should be glad.

Then we should design for the day it does not.

One final boundary matters: Spec Kit's community catalogs are read-only, and inclusion means only that a submission was validated as well-formed for the catalog. We do not vet the implementation or determine that community extensions, presets, workflows, bundles, or other components are safe; that judgment remains with each user before installing or running them. Nothing described here should be construed as a determination that the submitted extension itself was fine.

If you are building or operating agentic open source workflows and have a protection, failure, or red-team story worth sharing, send an email to blog (at) manorrock.com.

---

*Further reading: [Spec Kit Issue #4036](https://github.com/github/spec-kit/issues/4036), the [Add Community Extension workflow](https://github.com/github/spec-kit/blob/main/.github/workflows/add-community-extension.md), [GitHub Agentic Workflows](https://github.com/github/gh-aw), the [`gh aw` safe-outputs reference](https://github.github.com/gh-aw/reference/safe-outputs/), the [Spec Kit repository](https://github.com/github/spec-kit), [Spec Kit, Part 9: Workflows](../../07/09/spec_kit_workflows.html), [Spec Kit, Part 10: Steps](../../07/10/spec_kit_steps.html), and [A Year of Spec Kit — Notes from Washington](../../08/31/spec_kit_one_year_in_washington.html).*
