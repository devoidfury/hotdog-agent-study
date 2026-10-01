# Agent harness comparison - long horizon test run

There's been such an explosion of harnesses out there, so many interesting ideas. I've been wanting to get a better idea of what's going on in the agent framework/harness community; But there's marketing, docs -- and then there's the reality buried in the code. While I do enjoy reading code, there's too many to dig through by myself with other things going on in my life.

Why not have an agent do it for me? Well, have you _seen_ the size of these codebases? On these local models? You could fit maybe half of one before blowing out the context limit.

Well, how about a loop? I took a first stab at that using `/loop Read and follow the instructions at STUDY-TASKS.md`
This used a variant of my [TASKS.md](https://github.com/devoidfury/hotdog/blob/efc15c0c938c70cc6bc17b0bc0da8c861487aa36/examples/devoidfury/templates/TASKS.md), but I found that all it took was one misbehaving agent in the line to screw up the whole thing by trashing the document.

So that's where workflow graphs fit in. One manager agent (or a YAML-happy developer) can design a parameterized workflow graph of tasks to be done, including active reviewer steps that'll kick it back a couple times before failing the sub-tree.

The following tool call will dispatch up to, in this particular workflow, 6 full agent sessions plus retries to review a large codebase: scout first, then three reviewers split up the work in parallel, and an integrator to check and compile the results.

```
workflow_dispatch {"file":"t3-giant-review.workflow.yaml","run_id":"20260929-0502-t3-giant-review-3","args":{"subject": "gemini-cli", "subject-dir": "/data/samples/agents/gemini-cli"}}
```

That compresses the delegation down a whole lot, and enabled one 262k context model to orchestrate the whole thing.

## Execution

**Hardware:** 2x AMD Strix Halo 128GB as providers  
**Model:** [agentionai/Qwen3.8-Flash-Next ROCmFP4-FAST](https://huggingface.co/agentionai/Qwen3.8-Flash-Next-ROCmFP4-FAST-imatrix-GGUF)  
**Agent harness:** [devoidfury/hotdog](https://github.com/devoidfury/hotdog)

Next I had an agent clone the top ~100 open source harnesses from [bradAGI/awesome-cli-coding-agents](https://github.com/bradAGI/awesome-cli-coding-agents). While that was going, as a trial run, I started by having another agent compare six harnesses. That highlighted some bugs in both the harness and the instructions; I did a round of bug squishing and reworking the [prompt.md](prompt.md).

After it finished I kicked off the manager (`hotdog --profile meta`) with the following prompt (from within the hotdog directory, files refer to those in that repo, except the prompt):

> Please read the attached @experiments/agent-study/prompt.md and conduct the study. @package.json @AGENTS.md.

Afer it finished I followed up with a request for a retro:

> Could you also include a short reflective document about the study itself - how many subagents/workflows were used, challenges, and what we learned.

This [agent profile](https://github.com/devoidfury/hotdog/blob/efc15c0c938c70cc6bc17b0bc0da8c861487aa36/examples/devoidfury/config/profiles/meta.profile.md) was configured with tools to use both workflows as well as one-off direct delegation to subagents.

## Results

During the run it used a total of >240 combined sessions.

The output files, as well as a workflow it generated: [output directory](./output).

In particular, here's the most interesting couple of documents:
- [Agent harness tier list](./output/tier-list.md)
- [Roadmap for hotdog - critique and suggestions](./output/roadmap-for-hotdog.md)
- [Concept-convergence across the coding-agent corpus](./output/convergence.md)
- [Restrospective](./output/retrospective.md)

## What would I have done differently?

The timeouts on the nodes were too short for long-context on my current environment, a number of them timed out and restarted when another 20 minutes padding could've saved a whole session.

Scoring also needs some work; it's tricky to evaluate a complex piece of software, distill it down to a few KPIs, and gain anything genuinely useful from it. A lot of the lower scoring harnesses have great ideas of their own and it doesn't weight originality high enough. The scoring demotion to forks was also on the harsh end.

## But why, really?

I just wanted to see if such a task was possible with open models on consumer hardware - so I built out the workflows feature to enable it, and ran it.

I can't verify the accuracy of all of it, but it seems reasonable for the most part. It also highlighted some good ideas that I'm adding to the [hotdog](https://github.com/devoidfury/hotdog) todo-list.

I'd call that a good outcome.
