# Spec Kit, Part 10: Steps — Teaching the Graph New Words

*The tenth entry in a series exploring [Spec Kit](https://github.com/github/spec-kit) and what it can do for spec-driven development.*

[Part 9](../09/spec_kit_workflows.html) described a workflow as a graph of steps and then leaned on eleven built-in step types — `command`, `gate`, `shell`, the control-flow ones — as if that vocabulary were the whole language. It is not, and I said so in the last paragraph: steps are their own primitive, with their own catalog, that you can author and share the same way you author an extension. That is the final building block the series has not opened up. Every earlier post either treated the step vocabulary as fixed or deferred this one. This is the post where I open the box and show you that the eleven words the graph ships with are not privileged — they are just the first eleven, and you can teach it more.

The reason this matters more than "here is one more extensibility point" is that it removes the last ceiling. A workflow can only be as expressive as the steps it can name. If the built-in set were closed, then the graph — for all its branches and loops and gates — would top out at what eleven verbs can say, and anything past that would have to be smuggled in through a `shell` step shelling out to a script. Opening the step set means the graph's vocabulary is *yours to extend*, and, crucially, that a step you write is not a second-class citizen bolted onto the side. It is the same kind of object the built-in ones are.

## A step is an interface, not a category

Here is the claim I most want to land, because it reframes everything Part 9 said. In Spec Kit there is no such thing as a "core step type" that is somehow more real than a plugin one. There is a single Python interface — `StepBase` — and *every* step type implements it. The eleven built-ins are eleven classes that happen to ship in the box; a custom step is a twelfth class that happens to live in your project. The engine cannot tell them apart, because there is nothing to tell apart: they satisfy the same contract and register in the same table.

The contract is small enough to hold in your head:

```python
class StepBase(ABC):
    #: Matches the ``type:`` value in workflow YAML.
    type_key: str = ""

    @abstractmethod
    def execute(self, config: dict, context: StepContext) -> StepResult:
        ...

    def validate(self, config: dict) -> list[str]:
        ...   # return a list of error strings; empty means valid

    def can_resume(self, state: dict) -> bool:
        return True
```

That is the entire surface. A step declares a `type_key` — the word that appears after `type:` in workflow YAML — and implements `execute`, which receives the step's YAML config and a `StepContext`, and returns a `StepResult`. `validate` lets it reject bad config before a run ever starts; `can_resume` lets it participate in the pause/resume machinery. Four members, most of them optional. There is no separate "plugin API" that is thinner or clumsier than what the built-ins use. The built-ins use *this*.

## The proof: even `if` is just a step

If you want to see how completely the engine refuses to privilege its own vocabulary, look at how branching is implemented. You might assume `if`/`while`/`fan-out` are engine features — special syntax the executor understands natively. They are not. They are step types, exactly like the one you would write. Here is the entire heart of the built-in `if` step:

```python
class IfThenStep(StepBase):
    type_key = "if"

    def execute(self, config, context):
        result = evaluate_condition(config.get("condition", False), context)
        branch = config.get("then", []) if result else config.get("else", [])
        return StepResult(
            status=StepStatus.COMPLETED,
            output={"condition_result": result},
            next_steps=branch,
        )
```

That is the whole trick of control flow. A `StepResult` can carry `next_steps` — a list of step definitions to run next — and the `if` step's entire job is to evaluate a condition and hand back either the `then:` branch or the `else:` branch as those next steps. The engine does not know what "branching" is. It knows how to run a step and how to follow whatever `next_steps` that step returns. Looping, fan-out, multi-branch `switch` — all of them are the same idea: a step type that computes a list of steps and returns it. Which means the extension point is not a narrow slot for "leaf" actions. A custom step can emit nested steps too, so you can author your *own* control-flow construct if the built-in loops and branches do not match the shape of your process. The ceiling really is gone.

## What a custom step actually is on disk

Concretely, a custom step type is a small package under `.specify/workflows/steps/<type_key>/` containing exactly two required files: a `step.yml` with the metadata, and an `__init__.py` with a `StepBase` subclass whose `type_key` matches. That is it. Say you want a `notify` step that posts a message to a Slack-compatible webhook — something the built-in vocabulary can only fake by shelling out to `curl`. The metadata file is tiny:

```yaml
# .specify/workflows/steps/notify/step.yml
step:
  type_key: notify
  name: Notify
  version: 1.0.0
  description: Post a message to a Slack-compatible webhook.
  author: your-handle
```

And the implementation is a single class:

```python
# .specify/workflows/steps/notify/__init__.py
from specify_cli.workflows.base import (
    StepBase, StepContext, StepResult, StepStatus,
)
from specify_cli.workflows.expressions import evaluate_expression


class NotifyStep(StepBase):
    """Post a message to a webhook (e.g. Slack)."""

    type_key = "notify"

    def validate(self, config):
        errors = super().validate(config)          # checks the 'id' field
        for field in ("webhook", "message"):
            if field not in config:
                errors.append(
                    f"notify step {config.get('id', '?')!r} is missing {field!r}."
                )
        return errors

    def execute(self, config, context):
        message = config.get("message", "")
        if isinstance(message, str) and "{{" in message:
            message = evaluate_expression(message, context)   # resolve {{ ... }}
        status = _post(config["webhook"], {"text": str(message)})
        return StepResult(
            status=StepStatus.COMPLETED,
            output={"http_status": status},
        )
```

Now a workflow can say `notify` as if it had always been a word:

```yaml
  - id: announce
    type: notify
    webhook: "{{ inputs.slack_webhook }}"
    message: "Spec for {{ inputs.spec }} is ready for review."
```

The `type: notify` in the YAML resolves to your class through its `type_key`, and the whole `StepContext`/`StepResult` machinery from Part 9 applies unchanged: the `output` you return is stored as `steps.announce.output`, so a later step can branch on `steps.announce.output.http_status`, and the `evaluate_expression` call is what lets `{{ inputs.slack_webhook }}` and `{{ steps.plan.output.file }}` resolve inside your step exactly the way they do inside the built-ins. You did not learn a new templating system; you called the same one.

## Registration, and the one thing you cannot do

At startup the engine registers the eleven built-ins into a single table — `STEP_REGISTRY`, mapping `type_key` to a step instance — and then scans `.specify/workflows/steps/` and loads each valid package into the *same* table. To the executor, your `notify` and the built-in `gate` are two entries in one dictionary. That shared table is the whole reason there is no two-tier system: registration is the only mechanism, and everyone goes through it.

There is exactly one asymmetry, and it is a deliberate safety rail: a custom step **cannot shadow a built-in**. The loader skips any package whose `type_key` is already registered, so you cannot quietly redefine what `shell` or `gate` means for a workflow that thinks it is getting the standard one. You extend the vocabulary; you do not get to redefine its existing words out from under someone. That is the right call — a downloaded workflow relying on `gate` should get *the* gate, not a local override that changes what "pause for approval" does.

A couple of implementation details are worth knowing because they shape how you author. The registry holds a *single shared instance* per type, and a `fan-out` running with `max_concurrency > 1` can call `execute` on that one instance from several threads at once. So a step must be stateless and thread-safe: derive everything from the `config` and `context` arguments and never mutate `self` in `execute`. And a broken package — bad `step.yml`, an `__init__.py` that fails to import, no matching `StepBase` subclass — is silently skipped at load time rather than crashing every workflow in the project. Your one typo does not take down the reference sequence.

## Distribution is the Part 7 catalog, again

Here is where the series' habit of reusing its own machinery pays off one last time. A step you write in `.specify/workflows/steps/` is local to your project, but sharing it is not a new subsystem — it is the *same* catalog stack from [Part 7](../07/spec_kit_catalogs.html), pointed at a new component type. There is a full `specify workflow step` command surface that rhymes exactly with `extension` and `preset`:

```bash
specify workflow step list                 # built-in + installed custom types
specify workflow step search notify        # search the step catalog stack
specify workflow step info notify          # inspect one entry's metadata
specify workflow step add notify           # install from the catalog
specify workflow step remove notify        # uninstall
```

And the catalog *sources* resolve through the identical priority stack Part 7 described — `SPECKIT_STEP_CATALOG_URL` overrides everything, then project-level `.specify/step-catalogs.yml`, then user-level `~/.specify/step-catalogs.yml`, then the built-in official and community catalogs — with the same `install-allowed` versus `discovery-only` policy on each source:

```bash
specify workflow step catalog list
specify workflow step catalog add https://catalog.acme.internal/steps.json \
  --name acme-approved --priority 1
```

If you read Part 7, you already know how this behaves, because it is not a parallel implementation — it is the same one. An organization that wants to govern which *step types* its engineers can install does it with the exact priority-and-policy story that governs its extensions and presets. One mental model, five primitives. The step catalog is the fifth, and it did not cost a single new concept.

Installation itself is deliberately narrow: a catalog entry points at a `step.yml` URL and an `__init__.py` URL (plus optional `extra_files` for helper modules), both fetched over HTTPS only, staged, and moved into place atomically — with the install refusing to proceed unless the package's `type_key` matches the catalog ID it claims to be. There is no zip, no build step, no install script that runs on your machine. Two source files and their declared helpers, nothing more.

## The trust boundary, said plainly one more time

I cannot end on distribution without the warning the whole series has been consistent about, because a custom step is the sharpest edge yet. A step type is *arbitrary Python that runs in your process with your privileges*. It is strictly more powerful than a `shell` step: a shell step at least announces itself as "runs a command," whereas a step type can do anything Python can do and looks, in the workflow YAML, like an innocent `type: notify`. There is no sandbox. There never was one, and Part 9 explained why Spec Kit refuses to pretend otherwise — a `requires.permissions` field is rejected precisely because it would imply an enforcement that does not exist.

So the same discipline applies here that applied to community extensions in Part 8 and to workflows in Part 9. The maintainers verify that a catalog *entry* is well-formed; they do not audit the step *code*. `verified` is a statement about metadata, not safety. When you `specify workflow step add` something a stranger wrote, you are agreeing to run their Python — so read the `__init__.py` first. It is two files; that is a five-minute audit, and the design keeps it that small on purpose. And when you author your own and put it behind a workflow that others run, put a `gate` in front of anything destructive, because the gate is the one place the model hands judgment back to a human. The system gives you the power and the information to use it carefully; it declines, honestly, to make the safety call for you.

## Why I like this design

What I appreciate is the same thing I have appreciated at every layer, arrived at from the bottom this time. There is no "step plugin API" that is a lesser version of what the core uses. There is one interface, `StepBase`, and the built-in eleven implement it with no privileges a thirteenth could not have. The clearest evidence is that control flow itself — `if`, `while`, `fan-out` — is not engine magic but step types returning `next_steps`, which means the extension point reaches all the way up to authoring your own branching constructs, not just your own leaf actions. A design that lets a plugin do what the core does, through the exact mechanism the core uses, is a design that composes rather than accretes — the instinct I have pointed at in every one of these posts, now visible at the atom.

And the distribution story lands the series' other running theme: the step catalog is not a new registry, it is Part 7's catalog stack aimed at a new component type, governed by the same priority-and-policy model as everything else. Five primitives — extensions, presets, integrations, workflows, and now steps — and *one* way to find, install, and govern all of them. Most toolkits would have grown five distribution systems; Spec Kit grew one and pointed it five directions.

That is the last building block. Over ten posts I have gone from the top of the stack — a bundle that installs a whole process in one command — down to the single atom that a process is ultimately made of: a step, which turns out to be an open interface you can implement yourself. Bundles, catalogs, presets, extensions, integrations, the reference process, the kit of processes, the community that fills it, workflows that run it, and steps that extend its very vocabulary. The machinery is now all on the table — every primitive the toolkit has today, described.

Which makes the natural next question a forward-looking one, and really two questions at once: where does Spec Kit go from here, and where do *you* go from here? Having spent ten posts on what the toolkit *is*, next time I want to look down the road on both — the directions the project itself is heading, and the move that is now yours to make: the first extension you write, the preset you tailor for your team, the workflow you put in CI, the community entry you finally read the source of and install. I have not settled on which thread I will pull first, and I am open to being pushed. So rather than promise a specific post, I will promise that direction — the road ahead, yours as much as the project's. If there is a way you would like me to take it, or a step type you wish existed, send an email to blog (at) manorrock.com.

---

*Further reading: the [Spec Kit Workflows reference](https://github.com/github/spec-kit/blob/main/docs/reference/workflows.md) (step types and expressions), [What is Spec-Driven Development?](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md), [Part 9: Workflows](../09/spec_kit_workflows.html), [Part 8: A Field Guide to the Community](../08/spec_kit_field_guide.html), [Part 7: Catalogs](../07/spec_kit_catalogs.html), [Part 6: From Spec to Kit](../06/spec_kit_kit.html), [Part 5: The Spec-Driven Process](../02/spec_kit_process.html), [Part 4: Integrations](../01/spec_kit_integrations.html), [Part 3: Presets](../../06/30/spec_kit_presets.html), [Part 2: Extensions](../../06/29/spec_kit_extensions.html), and [Part 1: Bundles](../../06/26/spec_kit_bundles.html).*
