# Boundary re-review: grinta-coding-agent (T2, 2026-09-30)

Provisional 78.5 / A (exactly on the floor). Mandate: (1) is the every-PR mutmut run blocking,
is the label-gated DeepSWE eval a release property or decoration; (2) what fraction of the tree
does mutmut actually cover; (3) full independent pass on the other nine dimensions with the
god-file errata applied; plus the rival-subscription-transport taxonomy ruling.

Anchor sentence: closest to **pi** -- solo-maintainer local-first Python/TUI agent with strong
architecture discipline, tested-but-default-off enforcement, compaction craft, and exemplary
honest documentation; grinta is above pi on CI breadth and sandbox machinery, below on community,
interop surface, and originality spread.

## Q1a: Is the mutation run BLOCKING? NO.

`mutation-testing.yml:38` runs `uv run mutmut run` on every push to main and every PR with no
`continue-on-error` -- but the tool itself has no failing semantics for the thing being gated:

- mutmut is pinned `>=3.8` (`pyproject.toml:111`, resolved `mutmut-3.8.0` at `uv.lock:1693-1706`).
- In mutmut 3.8.0 source (wheel inspected, `mutmut/__main__.py`), the `run` command is
  `def run(...) -> None`: it exits non-zero ONLY for harness-level failures -- clean-test failure
  (`:995`), stats-collection failure (`:334`), mutant/test-association mismatch (`:865`). Surviving
  mutants are recorded in `.mutmut-cache` and the process exits 0. There is no `--fail-under`, no
  score threshold, no survivor gate anywhere in the tool (grep: zero hits for fail-under/fail_on).
- The workflow's follow-up step "Report mutation score" pipes `mutmut results --detail 2>&1 | tee`
  (`mutation-testing.yml:47-52`) -- results merely prints, and tee masks status anyway under
  GitHub's default `bash -e` (no pipefail).
- No compensating gate exists: no Makefile or script thresholds the mutation output (grep for
  mutmut outside the workflow: zero).

So a mutant surviving on a PR produces a green CI. The marginal blocking power of the job is a
re-run of 15 selected test files that `py-tests.yml` already runs. This is a mutation METRIC
(displayed in the step summary), not a mutation GATE. **The provisional's headline
verification-9 claim -- "every-PR mutmut" -- does not hold.** It does not enter the
verification-9 ledger (which requires a workflow line that blocks: prime-agent
behavioral-evals.yml:430, smelt ci.yml:269-294); the ledger stays at four.

## Q2: What does mutmut cover? ~1.9% of the tree.

`pyproject.toml:328-332` `source_paths = ["backend/utils/async_helpers", "backend/utils/lsp",
"backend/engine/response_processing"]` -- measured: async_helpers 1,363 + lsp 1,975 +
response_processing.py 860 = **4,198 LOC vs 218,066 non-test backend LOC (1.92%)**, with test
selection narrowed to 15 files (`pyproject.toml:341-355`). The job name is honest -- "reliability
modules" -- and the config is real work (also_copy sandbox plumbing, timeout_multiplier), but a
gate over 2% of the tree could not justify 9 even if it blocked. Two independent reasons the
claim fails: no enforcement, and negligible scope.

## Q1b: Label-gated DeepSWE eval: release property or decoration? DECORATION.

`run-eval.yml:4-10` fires on PR labels / release-published / manual, but the entire job is a
fire-and-forget `curl` `workflow_dispatch` to an OUT-OF-TREE private repo
(`App/evaluation/create-branch.yml`, `run-eval.yml:112-119`) plus a Slack ping (`:121-124`) plus a
PR comment (`:126-138`). Nothing in-tree scores, asserts, or blocks: the trigger job is green the
moment the dispatch is accepted, and the eval runs on `release: [published]` -- after the release
is already shipped, which is a post-hoc report, not a release gate. Contrast prime-agent's
label-gated FAIL-CLOSED 28-task gate. Per the gptme rule refinement (evals count toward 9 only if
the job blocks on failure), this is the existence of a trigger pipeline. Same verdict for the
third leg: `integration-runner.yml` runs LLM evals "on pull_request" but is a dead skeleton --
its detect step (`:31-44`) requires `backend/evaluation/agent_eval_pack.py` and
`evaluation/integration_tests/scripts/run_infer.sh`, NEITHER exists in the tree (verified with ls),
so every PR lands on the "Integration workflow skipped (assets missing)" explainer (`:57-64`).
OpenHands-lineage CI residue (consistent with the vocabulary-without-code-residue note);
`vscode-extension-build.yml:4` is the same shape, self-declared disabled.

**Verification: 9 -> 8.** The 8-rung is well earned: 162,279 test LOC / 716 test files, 14-job
cross-OS x Python matrix (`py-tests.yml:32` "required unit PR gates"), blocking coverage floor
(`py-tests.yml:338` `coverage report --fail-under=74`), compileall syntax gate, bandit + pip-audit
+ codeql + dependency-review workflows, 3-OS CLI regression e2e (`e2e-tests.yml:16-22`). No
fuzzing, no blocking in-CI eval: the anchors' 8-ceiling (per ERRATA) applies.

## Q3: Independent pass, other nine lanes

Core-loop read: `backend/engine/orchestrator.py` (461 LOC thin coordinator; step/astep delegate
to `orchestrator_helpers/step`; protocol-first deps per `:44-50`; event stream as sole channel
`:11`), control plane `backend/orchestration/session_orchestrator.py` (582) + mixins, loop/stuck
detection `backend/orchestration/stuck/detector.py` (772, repeating-action/observation/monologue
patterns), compaction `backend/context/compactor/compactor.py` (derived 80%-of-context budget
`:314-324`; strategy pipeline: microcompact -> observation-masking -> structured summary) and the
production continuity gate wiring `backend/context/context_pipeline/compaction.py:198,381`;
enforcement `backend/execution/aes/security_enforcement.py` + `backend/execution/sandboxing.py`
+ `backend/core/config/security_config.py`; trust gate `backend/cli/workspace_trust_prompt.py:17-22`.

- **architecture 8.** No god files anywhere (largest non-test source file 1,555, `cli/tui/widgets/scan_line/cards.py`; ERRATA >5k rule vacuous) and the loop/state plane is separated from the TUI. Held at 8, not 9: the forwarder-mixin composition (executor + `_executor_streaming_mixin` 1,156 + io_mixins; two types both named Orchestrator, documented collision at `orchestrator.py:3-6`) moves complexity rather than bounding it, and the in-tree `scripts/refactor/run_backend_organization.py` is churn evidence the structure was a live problem.
- **safety-enforcement 7.** Above the 6-rung ("approval, nothing underneath"): HIGH-risk confirmation ON by default (`security_config.py:28-36`), de-obfuscating regex analyzer with tests (`command_analyzer.py:41-51`, `test_command_analyzer.py`), OS jails for all three platforms (bwrap / sandbox-exec / Windows AppContainer, `sandboxing.py:1-10`), CONFINE-OR-REFUSE when the backend binary is missing (`sandboxing.py:205-208,222-224`), workspace-trust conservative-autonomy default for unfamiliar repos, and model-declared-risk escalation. Below 8: the jail is opt-in (`execution_profile='standard'` default), interactive terminals intentionally bypass isolation (honestly documented, `sandboxing.py:7-10`), and analyzer EXCEPTION falls back to model-declared risk, LOW when unknown (`security_enforcement.py:526-541`) -- the exact inverse of deepagents' classifier-unavailable -> require_human fail-closed.
- **token-economy 7.** Tiered pipeline + derived budget + per-provider cache adapters (`inference/caching/prompt_cache.py`) + prompt-tiering; missing pi-rung elements: no projected-next-turn trigger measure, no cache warming/monotonicity discipline.
- **orchestration 7.** Control plane + blackboard + delegated workers (`_event_router_delegate_mixin.py:362`) + stuck detector + replay-with-divergence-verification (`replay.py:1-5`) + recovery service + resume mixins; no queue/budget plane, crash semantics unproven. crush/pi rung.
- **interop 7.** MCP client (`integrations/mcp`, mcp_utils 1,167), LSP (2k LOC + edge suites), non-interactive CLI (`cli/entry.py:147,266-276`), rival-harness provider import (see taxonomy below). No ACP, no published SDK, IDE surface removed (`vscode-extension-build.yml:4`). crush rung.
- **operability 8.** Rollback/checkpoint manager (`execution/rollback/rollback_manager.py`), conversation resumer, `doctor` diagnostics suite, terminal-hygiene modules, onboarding. cline/crush rung; no codex-grade fork/queue tooling.
- **originality 7.** The compaction continuity gate is real and production-wired (blocking categories test_result/failed_approach/failed_outcome, `continuity_eval.py:28-35,146-183`) and has already converged cross-subject (forge-norvialabs-3 cites it); readonly_workspace's "make the FS immutable instead of classifying shell reads/writes" (`security_config.py:71-81`) is a genuinely good philosophy; replay determinism-check is uncommon. Docked from 8 because the gate's twin exists in-corpus and the rest is refinement-grade.
- **durability 6.** Remote-verified ACTIVE (pushed_at 2026-09-29; shallow clone honored per codel lesson), MIT, v1.0.0, PyPI, pages, governance trio (GOVERNANCE/MAINTAINERS/SECURITY/COMMUNITY) -- but solo owner (remote-verified), 31 stars, 8 months. Thin community + thin institution: pi's 7 needs the community.
- **docs-dx 8.** 95 in-repo md incl. ARCHITECTURE, CI.md, ADR.md, module maps, theme contract + deployed docs site + init/onboarding flow. cline rung.
- **verification 8.** See Q1/Q2 above.

## Rival-subscription-transport taxonomy ruling

`codex_app_server.py:1-6,34`: Grinta uses the Codex CLI's managed OAuth, then points the
AsyncOpenAI client at `https://chatgpt.com/backend-api/codex` and runs its OWN loop/tools/approvals
on the user's ChatGPT **subscription** auth. Ruling: **license-risk class, not safety-hole.**
Reasoning: this study's safety-hole kind is for enforcement that is advertised-but-inert,
fail-open, or partial-coverage -- a hole in a protection mechanism. Nothing here misrepresents or
fails to enforce anything inside Grinta; the defect lives in the TERMS under which the credential
is used: OpenAI subscription auth is licensed for first-party Codex surfaces, and riding it from a
third-party harness risks account action against the END USER. That is usage-terms/legal exposure,
the same bin the study already uses for manifest-only licenses, Apache self-grants (kode-cli), and
SONAR-SA scopes. Boundary with the existing claurst-2 finding (`oauth-client-impersonation`,
kind safety-hole): claurst ACTIVE-EVADES provider gates (spoofed client hash, extracted billing
salt, TLS fingerprint matching) -- that finding mixes ToS risk with deceptive-transport risk to
the provider's systems and stays distinct. Grinta declares its posture openly in the module
docstring, does not spoof, and so is the honest-but-ToS-exposed variant: file as
license-risk/`rival-subscription-transport`, concepts-only portables. If a future snapshot shows
Grinta working around detection, reclassifies upward into the claurst family.

## Verdict

| lane | score | weighted |
|---|---|---|
| architecture | 8 | 12.0 |
| verification | 8 | 12.0 |
| safety-enforcement | 7 | 7.0 |
| token-economy | 7 | 7.0 |
| orchestration | 7 | 7.0 |
| interop | 7 | 7.0 |
| operability | 8 | 8.0 |
| originality | 7 | 7.0 |
| durability | 6 | 3.0 |
| docs-dx | 8 | 4.0 |
| **total** | | **74.0 / B** |

A floor NOT crossed: the verification-9 claim fails on both enforcement (mutmut 3.8 cannot fail
on survivors) and scope (1.9% of the tree); the label-gated eval is a trigger-to-comment pipeline,
not a gate; the third eval leg never runs. Even granting the provisional's other-lane readings,
the single swing lane at issue caps at 8 by the ERRATA ceiling and the gptme blocks-on-failure
rule, so no reconstruction of the A floor survives. Band B FINAL recommendation; strongest lane
architecture/operability, weakest durability. No calibration rules invoked (provenance original,
remote-verified active, rule (a)/(b) n/a).
