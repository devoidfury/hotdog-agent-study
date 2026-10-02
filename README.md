# 103 coding agents, reviewed by ~240 autonomous agent sessions.

No cloud APIs. No human in the loop (except one timeout correction and asking for the retro).

## Results

| | |
|---|---|
| subjects | 103 open-source harnesses, shallow-cloned snapshots |
| agent sessions | ~240 over 4 days, one dispatcher, zero hand-written reviews |
| findings | 1,666 records → 678 canonical concepts, 161 converged across ≥2 subjects |
| model | Qwen3.8-Flash-Next ROCmFP4, 262k context, local |
| verdicts | [tier list](output/tier-list.md) · [convergence report](output/convergence.md) · [what broke](output/retrospective.md) |

Every claim is code-and-CI evidence from read-only snapshots, nothing executed. Errata and adjudication ledger included; several false inactivity calls were caught and corrected mid-run. Pick a row you hate and audit it: every score in `output/scores/`, every reviewer finding with file:line citations in `output/findings/`, boundary re-reviews in `output/boundary/`, errata in `output/calibration-notes.md`

It also turned up ideas I'm stealing for the [hotdog](https://github.com/devoidfury/hotdog) todo-list.

### Three things the study says

- **Most CI "eval gates" can't fail.**
   Of 103 harnesses, the audit found only a handful whose behavioral gates block on regression. ([verification9-sweep.md](output/wave6/verification9-sweep.md))
- **The field's dominant verification failure is wiring, not writing.**
   15 subjects ship test suites no CI runs (one has 151,832 test LOC behind a build-only pipeline).
   15 more have phantom safety controls: permission managers with zero call sites, sandbox toggles nothing reads, TUI approval prompts no backend emits. ([convergence.md](output/convergence.md))
- **hotdog is in there at 68.5/B, self-scored.** Treat that as the least trustworthy row in the study.

## Execution

**Hardware:** 2x AMD Strix Halo 128GB as providers  
**Model:** [agentionai/Qwen3.8-Flash-Next ROCmFP4-FAST](https://huggingface.co/agentionai/Qwen3.8-Flash-Next-ROCmFP4-FAST-imatrix-GGUF)  
**Agent harness:** [devoidfury/hotdog](https://github.com/devoidfury/hotdog)

I took a first stab at using a loop: `/loop Read and follow the instructions at STUDY-TASKS.md`. This used a variant of my [TASKS.md](https://github.com/devoidfury/hotdog/blob/efc15c0c938c70cc6bc17b0bc0da8c861487aa36/examples/devoidfury/templates/TASKS.md), but I found that all it took was one misbehaving agent in the line to screw up the whole thing by trashing the document. So that's where workflow graphs fit in. One manager agent (or a YAML-happy developer) can design a parameterized workflow graph of tasks to be done, including active reviewer steps that'll kick it back a couple times before failing the sub-tree. That compresses the delegation down a whole lot, and enabled one 262k context model to orchestrate the whole thing.

[./prompt.md](./prompt.md)
[./t3-giant-review.workflow.yaml](./output/t3-giant-review.workflow.yaml)

The manager (`hotdog --profile meta`) kicked off with the following prompt, (from within the hotdog directory, files refer to those in that repo, except the [prompt.md](prompt.md)):

> Please read the attached @experiments/agent-study/prompt.md and conduct the study. @package.json @AGENTS.md.

After it finished I followed up with a request for a retro:

> Could you also include a short reflective document about the study itself - how many subagents/workflows were used, challenges, and what we learned.

This [agent profile "meta"](https://github.com/devoidfury/hotdog/blob/efc15c0c938c70cc6bc17b0bc0da8c861487aa36/examples/devoidfury/config/profiles/meta.profile.md) was configured with tools to use both workflows as well as one-off direct delegation to subagents.

### Issues and breakages

- Subagent busted the prompt cache during synthesis and the manager looped on context reload until a human raised a timeout and restarted it. Yes, that was the only human touch in four days.
- Long-report generation killed the same deliverable three times: workers streamed one enormous tool-call, stalled, and reported "completed" with nothing on disk.
- The mechanical census phase got test-LOC wrong by 5-6x on multiple subjects. Reviewers caught it; automation alone would not have.
- Two integrators died at the runtime cap mid-merge; resuming the same run dirs lost zero review work.

### What would I have done differently?

The timeouts on the nodes were too short for long-context on my current environment, a number of them timed out and restarted when another 20 minutes padding could've saved a whole session.

Scoring also needs some work; it's tricky to evaluate a complex piece of software, distill it down to a few KPIs, and gain anything genuinely useful from it. A lot of the lower scoring harnesses have great ideas of their own and it doesn't weight originality high enough. The scoring demotion to forks was also on the harsh end.
