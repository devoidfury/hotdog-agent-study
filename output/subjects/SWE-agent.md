# SWE-agent — T1 review

Subject: `/data/samples/agents/SWE-agent` (origin https://github.com/SWE-agent/SWE-agent, MIT, head 2026-07-16, not shallow, 2182 commits / 105 contributors). Python package `sweagent` (pyproject). Research-grade batch harness from the SWE-bench / Princeton-Stanford team; the historical ACI (agent-computer interface) ancestor.

## Census sanity

- `non_test_loc: 18425` is inflated ~2.4x: cloc on `sweagent/` gives **7,714** Python code (122 py files repo-wide incl. tests, tool scripts, docs assets counted elsewhere).
- `test_loc: 3782` vs cloc `tests/` = **1,702** Python test code (the rest is presumably test_data/trajectories). Direction of both errors is inflation, not miss; tier stays **T1** regardless.
- Activity: head 2026-07-16, ~2.5 months old at review, `.git/shallow` absent -> active, no archived cap.
- Provenance: `original`, matches origin. No fork delta to report. Identity check: worked only in this directory; no conflation with mini-swe-agent (same authors, separate repo, not in this corpus census).

## Anchor question

**Which anchor is this closer to, and why?** crush (69.0) -- a cleanly engineered loop with tested error semantics and one standard token story, but SWE-agent has none of crush's product plane (server, auth, TUI polish) and no MCP, landing it slightly under crush and well under cline; clearly above codel-tier by virtue of a real, tested loop.

## Per-dimension scores

### architecture — 7/10 (weight 15)
Clean layered split: `agent/` (loop, models, history, reviewer), `environment/` (env facade over SWE-ReX deployments), `tools/` (parsing + bundles + filter), `run/` (single/batch orchestration), with three-level hook seams (agent/env/run hooks, `sweagent/agent/hooks/abstract.py`, `sweagent/environment/hooks/abstract.py`, `sweagent/run/hooks/abstract.py`). Loop is explicit `while not step_output.done: step_output = self.step()` (`sweagent/agent/agents.py:1287-1288`) with error taxonomy handled in `forward_with_handling` (`agents.py:1062-1216`): format / blocklist / bash-syntax errors requery, cost/context/timeout errors autosubmit-and-exit, control-flow via typed exceptions (`agents.py:200-215`). Every component is a pydantic discriminated-union config (`history_processors.py:390-400`, `agents.py:196`) -- fully serializable/reproducible runs. Largest file `agents.py` 1,294 LOC fuses DefaultAgent + RetryAgent but roles are separated; no god files (nothing >1.3k). Below the crush-7 fuse penalty and above nanocoder's 5: module boundaries are honored; the residual cost is that `agents.py` mixes two agent classes and env coupling is direct (`self._env: SWEEnv`) rather than interface-typed in the agent (AbstractAgent protocol `agents.py:226-240` is nominal).

### verification — 6/10 (weight 15)
137 test functions; the loop specs assert real semantics, not existence: exit-cost, exit-context, exit-format, blocklist, early-exit, autosubmit, function-calling, and step-by-step history verification (`tests/test_agent.py:80-246`) driven by a canned `PredeterminedTestModel` (`sweagent/agent/models.py:529`) against swerex `DummyRuntime` -- faux-provider testing of the full loop. History processors tested against golden trajectories from real runs (`tests/test_history_processors.py:19-35`, `tests/test_data/trajectories/`). CI: pytest matrix py3.11/3.12 + coverage -> codecov (`.github/workflows/pytest.yaml:29-66`); `codecov.yml` present. Docked vs the 7 rung (crush, nanocoder): smaller corps (1.7k test LOC), no `-race`/fuzz analog, no in-CI model evals (SWE-bench eval lives in a runtime hook, `sweagent/run/hooks/swe_bench_evaluate.py`, never CI), and the bash tool scripts under `tools/*/bin/` (the ACI itself) have no tests.

### safety-enforcement — 5/10 (weight 10)
Isolation is delegated and coarse: the default deployment is a Docker container (`sweagent/environment/swe_env.py:27-29` DockerDeploymentConfig via swerex), which is on by default -- better than nanocoder's opt-in jail, but the machinery is an external package (evidence-limited within this repo; sandbox-delegation pattern). What binds *inside* this repo is only the command filter: prefix/exact-match blocklist (`sweagent/tools/tools.py:37-71`, enforced at `tools.py:353-366`, tested `tests/test_agent.py:120-135`) -- ACI hygiene against pagers/interactive commands, trivially bypassable mid-command (`startswith`), and honestly labeled as such (`tools.py:29-31` "Filter out commands ... for example interactive commands"), not a security boundary. No approvals, no per-action policy, no egress control in-repo. Below the 6 rung ("tested approval, nothing underneath") because there is no approval concept at all; 5 = real machinery (default container) + coarse in-loop filter.

### token-economy — 6/10 (weight 10)
No LLM-summarization compaction; context overflow is terminal (`ContextWindowExceededError` -> autosubmit-exit, `agents.py:1175-1178`) -- no projected-window trigger like pi. Instead a composable elision pipeline as serializable configs: `LastNObservations` (classic keep-last-5, with `polling` param to batch history edits and *not* break prompt caching -- `history_processors.py:85-135`), `ClosedWindowHistoryProcessor` (dedupes stale file windows, `:210-258`), `RemoveRegex`, observation truncation with corrective guidance ("use head/tail/grep/redirect", `agents.py:63-77`). Real cache discipline beyond the crush/nanocoder 6 rung in two places: explicit Anthropic `cache_control` ephemeral marks as a pluggable processor (`history_processors.py:261-300`) and per-thread API-key affinity so batch workers keep the provider cache partition stable (`models.py:172-190`). Net: same rung as crush/nanocoder (6) -- one standard story, well executed -- cache extras don't reach pi's 8 because there's no summarization or window projection at all.

### orchestration — 6/10 (weight 10)
Batch runner with ThreadPoolExecutor (`run_batch.py:39,87`), per-instance budgets enforced at the model layer (`models.py:379-382`: instance/total cost + call limits), and a genuinely distinctive `RetryAgent`: multi-attempt retry with an LLM reviewer/chooser and cost-aware budget allocation -- each attempt's per-instance cap is clamped to remaining budget (`agents.py:307-310`), new attempts blocked below `min_budget_for_new_attempt` (`reviewer.py:537-541`). Crash tolerance: trajectory saved after *every step* (`agents.py:1288`), reruns skip completed `.traj` and quarantine corrupt/unfinished ones (`run_batch.py:380-408`, `remove_unfinished.py:13-40`) -- resume-by-journal-scan, no explicit resume API. No subagents, no queue/daemon, no loop detection (crush remains the only anchor with a breaker), threads-not-processes. Rung 6 like crush-without-loop-detection.

### interop — 3/10 (weight 10)
Zero MCP, zero ACP, no IDE surface, no published SDK (grep over `sweagent/` and `docs/` finds no hits). Headless contract is a batch CLI (`sweagent run single/batch`) with URL-composable config overrides; interop is really with the *benchmark ecosystem* -- SWE-bench evaluation hook (`run/hooks/swe_bench_evaluate.py`), SWE-ReX/EnIGMA runtimes, SWE-smith configs (`tests/test_swesmith.py`), trajectory format consumed by external viewers plus the in-repo inspector. Above codel's 2 (two hardcoded providers) via the deployment/runtime abstraction and eval ecosystem, far below crush/nanocoder's 7.

### operability — 6/10 (weight 10)
Strong research-ops story: per-step trajectory journal, a `replay` model that replays any trajectory (`models.py:204,464`), `run_replay.py`, human / human_thought models for step-in debugging (`models.py:231,446`), web inspector for trajectory browsing (`sweagent/inspector/server.py`, `docs/usage/inspector.md`), batch progress manager, `compare_runs.py`. No session resume mid-run (restart semantics only), no checkpoints of the environment beyond reset (`swe_env.py:128-148`), crashes land as `exit_status` strings in the trajectory. Rung 6 (nanocoder-like): good tooling, no crash-recovery posture beyond the journal.

### originality — 7/10 (weight 10)
Verified in code/docs, not assumed: the ACI concept is documented (`docs/background/aci.md:1-14`, paper arXiv:2405.15793) and *implemented in-repo* -- windowed file viewer with open/goto/scroll state (`tools/windowed/bin/`), match-count-only directory search (`tools/search/bin/search_dir`, config.yaml docstrings), anthropic edit + linter bundle (`tools/edit_anthropic/`), legacy ACI config preserved (`config/sweagent_0_7/`). Tools-as-bundles (upload + install.sh + generated command docs, `tools/tools.py:252-275`) is the pattern the field copied; the current v1.x loop itself has since converged toward the authors' minimalist mini-swe-agent style. Unique mechanisms in current code: reviewer-guided retry loop with cost budgeting (`reviewer.py:499-559`), action sampling (best-of-N via sampler, `action_sampler.py:1-60`), serializable history-processor pipelines, per-thread API-key cache affinity, trajectory inspector + replay model. Held at 7 (crush/nanocoder rung) rather than the 9 "field-defining" rung: the ACI is historically field-defining -- the origin *is* documented -- but cross-subject influence in this corpus must be scored by synthesis, and the current codebase's live ideas are these smaller, solid ones.

### durability — 6/10 (weight 5)
MIT, SWE-agent org (Princeton/Stanford SWE-bench team), SECURITY.md with named maintainer contacts, 105 contributors, active 2.5 months to head, CI + codecov + pre-commit-ci (74 bot commits). Docked for bus factor: top contributor has 1,536/2,182 commits (70%); it is a research-org project, no company funding. Rung between nanocoder-4 (thin) and crush-8 (funded + CLA + release cadence).

### docs-dx — 7/10 (weight 5)
Full mkdocs site (~60 md files): architecture/ACI background (`docs/background/`), CLI tutorial, batch mode, custom-tool authoring (`docs/usage/adding_custom_tools.md`), trajectories format, multimodal guide; unusually rigorous CHANGELOG with PR-level provenance back to v0.x (`docs/installation/changelog.md`); `docs/installation/` container/conda paths. Docked from 8: config system has a known footgun surface the docs themselves warn about (empty templates warning, `agents.py:136-145`), and troubleshooting is FAQ-thin.

## Totals

| dimension | score | weight | contrib |
|---|---|---|---|
| architecture | 7 | 15 | 105 |
| verification | 6 | 15 | 90 |
| safety-enforcement | 5 | 10 | 50 |
| token-economy | 6 | 10 | 60 |
| orchestration | 6 | 10 | 60 |
| interop | 3 | 10 | 30 |
| operability | 6 | 10 | 60 |
| originality | 7 | 10 | 70 |
| durability | 6 | 5 | 30 |
| docs-dx | 7 | 5 | 35 |
| **weighted_total** | | | **59.0** |

Band: **C** (45-64). Strongest dimension: **originality (7)** -- the ACI artifact lineage is documented and implemented, plus live-unique reviewer-retry/budget machinery. Weakest dimension: **interop (3)** -- no MCP/ACP/SDK/IDE; ecosystem interop only via SWE-bench/SWE-ReX.

## Calibration notes

- No calibration rules triggered: not a fork, not archived. Scored below crush (69) on shape and totals; the corpus's most historically significant subject is *not* a top scorer for current-generation operational qualities -- synthesis should treat SWE-agent's originality as ancestral-influence evidence, not as a ranking boost (band C stands on current-code merit).
- Boundary check: 59.0 is >2 pts from the 45 and 65 cuts. No boundary risk.
- Shallow-clone hygiene: not shallow; activity claim is safe.
