# Boundary re-review: auto-code-rover

Provisional 45.5 C, flagged sensitive both directions. Independent pass; provisional
report/scores/findings not read. Protocol: review-protocol.md; anchors read including
ERRATA (product-file >5k LOC docking factor; 9+ verification requires evals-in-CI/fuzzing).

**Anchor sentence:** Between codel (23.0 D) and nanocoder (60.5 C), clearly closer to
codel -- ACR shares codel's near-empty safety/orchestration/interop profile, and only
verification + architecture lift it off the D floor; it never approaches nanocoder's
ACP/daemon/jail substance.

## Fact re-verification (all re-checked, not assumed)

- Size: `cloc app` = 6,968 code LOC, `cloc test` = 6,141; app+test = 13,109; +scripts/
  (1,049, incl. eval tooling) ~ "13.9k" within counting-basis margin. The 927 MB
  `results/` blob dirs (acr-val-sbfl 202M, acr-val-only 179M, swe-agent-results 151M, ...)
  confirmed and excluded.
- Activity: `.git/shallow` present; remote checked per protocol: GitHub API
  `pushed_at: 2025-04-24T07:58:24Z`, `archived: false`, `disabled: false`. 17 months
  quiet, not archived. 3,100 stars, 332 forks, 21 open issues unattended.
- License: LICENSE = "SONAR Source-Available License v1.0" (GitHub: NOASSERTION).
  Concepts-only portables honored below.

## Q1. Are the 6,141 test LOC property-proving? -> verification 6, not 5, not 7

Real and behavior-asserting, but mock-heavy with a soft CI gate.

- 300 `def test_`s; CI on every push/PR to main runs `tox -e unit` with coverage +
  SonarQube (`.github/workflows/pytest.yml:3-11,48-56`). Gate is existence-only:
  "Check Coverage Report Exists" aborts if `coverage.xml` missing -- no fail-under
  threshold, no coverage-drop gate (contrast nanocoder 7 with its `pr-checks.yml:17-21`
  drop-gate + real-bubblewrap spec job).
- Loop semantics genuinely tested: `test/app/test_inference.py:54` drives
  `write_patch_iterative_with_review` through a DummyReviewManager yielding
  fail-then-success and asserts the select/validation loop honors the eval feedback;
  `test/app/search/test_search_backend.py` (63 tests, `:67` temp-dir + monkeypatch
  index build); `test/test_acr_command.py` E2E-wires the CLI with a call_tracker.
- Ceiling: only 2 `@pytest.mark.integration` tests; the live-model one runs a full
  langchain issue end-to-end but asserts a single success string
  (`test/test_acr_command.py:402-404`), and the OpenAI one skips without a key
  (`:415-417`). No fuzzing, no hypothesis, no in-CI benchmark eval. Integration legs
  run only on main-pattern branches (`pytest.yml:52-56`).
- 7 requires the crush/nanocoder package (large corps + hard CI measures); 5
  undersells tested loop behavior. 6 = solid, one standard design (fake/monkeypatch
  unit suite), tested on every push. ERRATA 9+ rule not even in play.

## Q2. Safety: rung 1, not 2. README split does not flip the honesty call

No approval layer exists, and nothing pretends otherwise -- so not 2.

- Enforcement absent, verified by grep sweep of `app/` for
  approv/permission/sandbox/deny and `input(`: zero hits (the "reviewer-approved"
  strings are the model-reviewer patch gate, `app/inference.py:41`, not a user
  permission boundary). No SECURITY.md.
- Consequence is live execution of model-written code on the operator's machine:
  `app/agents/agent_reproducer.py:111` -> `app/task.py:420-446` runs the reproducer via
  `run_script_in_conda` (`:431`, only guard = 120 s timeout); arbitrary test commands
  run at `app/task.py:339`. No confirmation prompt anywhere.
- Why not 2 (codel's rung): 2 needs present-but-misleading -- codel had prompt-text
  policy ("Always auto approve...") and README contradicting behavior. ACR claims no
  control, so there is no misleading artifact to dock; `phantom-safety-control` does
  not apply. Rung 1 "absent, honest by omission" fits.
- Docker-path vs eval-path: `README.md:110` "We recommend running AutoCodeRover in a
  Docker container" is a real, functioning whole-tool isolation path; `README.md:209`
  recommends host setup for SWE-bench mode with a practical reason (per-task conda
  testbeds), not a safety dismissal. The split changes *risk posture*, not the
  honesty verdict: mechanics are documented truthfully in both branches. The residual
  sin is omission -- nowhere does any doc state that ACR executes model-written
  reproducers/test code unsandboxed -- recorded as a safety-hole finding, insufficient
  to promote "absent" to "misleading".

## Q3. Does rule (b) matter at this score? -> No, and moot

Rule (b) caps archived/dead at B; caps cannot push a subject *down* into C or D. The
final total lands in D on the weighted sum alone, so the cap never binds. Two side
effects worth recording: (i) 17-months-quiet feeds durability=2 legitimately; (ii) per
protocol tiering, "dead >12mo at HEAD" means this subject qualified for T0 triage --
the study over-tiered it, which is consistent with a below-provisional final.

## Lane scores (weighted_total 41.5 -> D)

| dim | score | one-line basis |
|---|---|---|
| architecture | 6 | clean small-file split (max 970: `app/search/search_backend.py`), coroutine-generator loops (`app/inference.py:30-59`, `app/search/search_manage.py:29-64`); dinged for global `app/config.py` state + triplicated `validate` (`app/task.py:67,205,526`) |
| verification | 6 | per Q1 |
| safety-enforcement | 1 | per Q2 |
| token-economy | 3 | output-side truncation only (`app/task.py:447-453`, `RESULT_SHOW_LIMIT` search_backend:22), round budget (`app/config.py:12` conv_round_limit=15), per-call cost tracking; no compaction; context overflow = log + re-raise, task dies (`app/model/gpt.py:238-241`, `app/model/common.py:176-178`) |
| orchestration | 3 | fixed role pipeline, subprocess-per-task (`app/main.py:460`), tenacity backoff; no queue/resume/budgets |
| interop | 3 | no MCP/ACP/SDK; composite GH action install-only (`action.yml:5`), JSON-file contract (`selected_patch.json`), 8-provider hub incl. LiteLLM |
| operability | 4 | per-round conversation dumps (`app/search/search_manage.py:63-64`), loguru, `result_analysis.py` + demo_vis forensics; no resume/diagnostics |
| originality | 6 | coverage-SBFL fault localization as an agent tool (`app/analysis/sbfl.py:1-9`, surfaced at `app/manage.py:36`, tested `test/app/analysis/test_sbfl.py`) -- genuinely rare in census; plus reproducer-then-patch pipeline |
| durability | 2 | 17mo quiet (remote-verified), not archived; academic org, no SECURITY.md, thin bus factor |
| docs-dx | 5 | 354-line README matching behavior, 3 modes + FAQ; no docs tree, no SECURITY.md |

9.0+9.0+1.0+3.0+3.0+3.0+4.0+6.0+1.0+2.5 = **41.5 (D)**.

## Calibration notes (recorded, not silent)

- Demotion C->D driven by: safety 2->1 (no phantom controls), verification 7->6 (soft
  gate, smoke integration), token-economy down to 3 (fail-rethrow overflow). Every
  alternate near-miss assignment still totals <45 (max-case token 4 / oper 5 / interop
  4 = 44.0), so D is robust to one-point disagreement.
- Rule (b) considered and non-binding (cap, not floor). No sync-fork relation.
- Census sanity: app/test LOC figures hold under cloc; 927 MB `results/` blobs must
  stay excluded from any size/LOC rollup.
