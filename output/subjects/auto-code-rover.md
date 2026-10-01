# auto-code-rover - T2 review

Subject: /data/samples/agents/auto-code-rover @ 585d3e6 ("Hotpatch (#92)", 2025-04-24)
Upstream: https://github.com/AutoCodeRoverSG/auto-code-rover (census provenance "original": correct;
README image links still point at the old org nus-apr/auto-code-rover, a relocation, not a clone.)
License: SONAR Source-Available License v1.0 (LICENSE:1, non-OSI, "Competing"-services restriction).
All findings below are concepts-only; effort_for_us assumes clean-room. No code porting recommended.

## Census corrections (mandatory rescoping)

- **LOC**: census `non_test_loc 2,614,283 / primary_language "diff"` is an artifact of checked-in
  run outputs: 3,161 `.diff` files under `results/` (927 MB of SWE-bench run blobs, e.g.
  results/acr-run-1/applicable_patch/*/*.diff). Real code by cloc: app 6,968 Python LOC (prod),
  test 6,141, scripts 814; ~13.9k total. That is T1-size, not T3. Dispatcher ran T2 anyway;
  protocol triage (dead >12mo + ~8k prod LOC) would put this at T0. Full line-level review done regardless.
- **test_loc 6,141**: correct (matches cloc exactly).
- **contributors 1 / commits 1**: shallow-clone artifact. Remote contributors API: ~14 contributors,
  top 3 (Marti2203 68, crhf 65, yuntongzhang 63) carry it; NUS APR research group + Sonar (license).
- **Activity**: head_date 2025-04-24 is ~17 months before review date (2026-09-29) and > 10 months,
  so remote-verified before any rule (b) claim: GitHub API `pushed_at 2025-04-24T07:58:24Z`,
  `archived: false`, 21 open issues. Shallow clone was NOT masking recent activity this time.
  Dead-by-corpus-definition (>12mo) => rule (b) cap B applies (moot at final score).

## What it is

Not an interactive harness. A fixed research pipeline for autonomous issue->patch
(arXiv 2404.05427; SWE-bench Lite 37.3% / Verified 46.2% era results in README:29-32):
optional reproducer-test agent -> optional SBFL fault localization -> iterative AST-search
context gathering -> patch agent -> optional reproducer+reviewer counterfactual loop ->
file-based patch selection. Entry `run_one_task` at app/inference.py:98.

Anchor question: closer to **crush (69.0)** - a real, CI-tested mechanism harness with genuine
tooling rather than codel's oracle facade - but with none of crush's permission service, compaction,
resume, or MCP/LSP surfaces, and the same whole-container isolation posture; the trae-agent (44.0 D)
comparison is the nearest already-scored sibling and ACR sits marginally above it on verification
and originality, marginally below on safety.

## Core loop (read at line level)

- Orchestration: `app/inference.py:98-131` - up to `overall_retry_limit=3` (config.py:9) outer
  retries, cycling across `--model` list via `itertools.cycle` (inference.py:108-111); each retry
  gets its own `output_N` dir, no resume into it.
- Search loop: `app/search/search_manage.py:29-186`. The search agent is a Python generator
  (`agent_search.py:87-160`) advanced with `send()`; the manager parses the model's prose into
  JSON API calls via a second LLM ("agent proxy", `agent_proxy.py:45-71`, retries 5), dispatches
  against the tree-sitter `SearchBackend` by reflection (`search_manage.py:160-181`, `assert` on
  arity at :165-167), and loops up to `conv_round_limit=15` (config.py:12). Termination is by
  "bug_locations present and snippets resolve" (`search_manage.py:110-150`), else `return [], thread`
  at :184-186 - patch gen proceeds on empty context.
- Review loop: `app/api/review_manage.py:63-167` - generator coroutine: write patch, run reproducer
  with/without patch (`task.py:420-440`), LLM reviewer judges patch AND test, and feedback is routed
  back into either the patch agent or the test agent (:132-167). `rounds=5` (:62), outer retries=3
  in `write_patch_iterative_with_review` (inference.py:28-60).
- Selection: `inference.py:134-232` - regression-filter candidates, prefer reviewer-approved,
  else LLM majority/select agent. Contains dead code: `if False:` at inference.py:183 and :212
  disables the majority-vote branch; personal absolute paths in `__main__` blocks
  (inference.py:346-348, review_manage.py:239+) are research-grade residue.

## Compaction / context management

None. `MessageThread` is append-only across all 15 search rounds (`agent_search.py:125-141`);
the only defenses are ad hoc truncations: `RESULT_SHOW_LIMIT = 3` per search call
(search_backend.py:22, :305-314), stderr head-50/tail-50 fold (task.py:450-452),
`max_tokens=os.getenv("ACR_TOKEN_LIMIT", 1024)` (common.py:150). On `context_length_exceeded`
the code logs and re-raises (common.py:176-177) - straight into the blind tenacity retry (see
finding -4). No prompt-cache awareness anywhere. Cost visibility is genuinely good: thread-local
per-request token/cost accumulators (common.py:19-24, :56-83) and a tested cost calculator
(test/app/model/test_common.py:26-49).

## Permission / sandbox code

There is none to read - that is the finding. No approval/permission/confirm concept exists in app/
(grep: only Docker daemon error text). LLM-written reproducer code executes as a python script in
the host conda env: `run_script_in_conda([f.name], self.env_name, ...)` (task.py:431-437) with the
120s timeout as the only constraint; patches applied via `patch -p1` (api/validation.py:130).
README:110 recommends Docker for minimal mode but README:209 explicitly recommends running on the
**host** for SWE-bench mode - the primary documented workflow executes model-generated code
unsandboxed by recommendation. No SECURITY.md. Below the cline/pi "tested approval" rung; comparable
to codel's 2 except ACR never claims safety - it just has no gate at all.

## Dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | Clean stage modules (app/agents, search, model, api, analysis); generator-based feedback loops (review_manage.py:63-167); no god file (max search_backend.py 970). Docked: global mutable config (app/config.py:1-45), `if False:` dead branches (inference.py:183,212), hardcoded personal paths (inference.py:346). |
| verification | 6 | 36 pytest files / 6,141 LOC asserting behavior, monkeypatch dummies (test_search_backend.py:19-27, 254 asserts); CI on push/PR runs tox unit, +integration on branch-pattern (pytest.yml:52-57) with coverage gate; style + docker-build workflows. Docked: zero in-CI SWE-bench evals for an eval-driven tool; integration tests hit live API keys (pytest.yml:14-17); orchestration itself thin (test_inference.py: 7 asserts). |
| safety-enforcement | 2 | No approval layer anywhere; LLM-generated code executed on host conda env (task.py:431-437), README:209 recommends host mode; only gates are timeouts (config.py:38, task.py:437). |
| token-economy | 3 | No compaction, 15-round append-only threads (config.py:12, agent_search.py:125-141); ad hoc truncation (search_backend.py:22, task.py:450-452); blind retry of context-overflow (common.py:127+176). Cost accounting (common.py:19-24) is the good half. |
| orchestration | 4 | Bounded generator loops (config.py:9,12; review_manage.py:109), model-cycling retries (inference.py:108-131), full per-round artifact journaling (search_manage.py:66, :196-200) - but no resume, no budgets, no loop detection (bounds only). |
| interop | 4 | Composite GitHub Action (action.yml:1-30), SWE-bench-docker validation (api/swe_bench_docker_validation.py, task.py:288), 8 providers + litellm-generic escape hatch (common.py:206-221), GitHub/local issue modes. No MCP, ACP, SDK, IDE surface. |
| operability | 4 | Per-run output dirs with conversation/review/execution JSON (review_manage.py:203-231), loguru logging, result_analysis.py + demo_vis post-mortem tooling; no session resume (crash = rerun), CLI+globals config (main.py:38+, config.py). |
| originality | 7 | Verified-in-code mechanisms no harness subject has: counterfactual dual-run reproducer+reviewer loop (review_manage.py:104-167 + task.py:420-440); AST-indexed search backend with 8 structured APIs (search_backend.py, requirements.txt:111-115); coverage-based SBFL seeding (inference.py:297-300, agent_search.py:103-106, analysis/sbfl.py:254); LLM json-repair proxy (agent_proxy.py:45-71). |
| durability | 2 | Dead >12mo remote-verified (pushed_at 2025-04-24, ~17mo); not archived, 3.1k stars, NUS APR + Sonar backing and a Discord, but bus factor top-3, 21 open issues unattended. |
| docs-dx | 5 | 353-line README with install/run/eval/model config + troubleshooting (:110, :332), TESTING.md, EXPERIMENT.md; docked for the :110-vs-:209 docker/host contradiction and dead code reachable by a new contributor. |

Weighted: 9 + 9 + 2 + 3 + 4 + 4 + 4 + 7 + 1 + 2.5 = **45.5 -> C** (band boundary: within 2 of 45).
Strongest: originality (7). Weakest: safety-enforcement (2).

## Boundary sensitivity

45.5 sits 0.5 above the C/D cut. A lift case exists on verification 6->7 (behavior tests +
blocking CI + coverage gate are real, and the integration/unit split is disciplined); a drop case
exists on architecture 6->5 (dead `if False` branches + global config are worse than the 5 rung
"works, known gaps"). Both defensible; I keep both scores and flag the boundary rather than
let band assignment ride on rounding. Even the optimistic 47.0 stays far under the rule (b)
dead-cap of B.

## Portability note

Sonar Source-Available v1.0 => every finding is concepts-only; none of the code may be copied.
The genuinely interesting transferable concepts: counterfactual dual-run review (finding -1),
static-analysis context seeding (-7), and the ACI-style structured-search idea (-2).
