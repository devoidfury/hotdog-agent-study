# grinta-coding-agent -- T2 deep review

Subject: Grinta Coding Agent (`grinta`, Python, MIT, original). Census: 222,970 non-test / 119,774 test LOC, T2, shallow clone (1 commit), head 2026-09-22.
Verified with cloc/find: Python total 283,461 code LOC; non-test .py = 218,066 (census fine); **test .py = 162,854 across 764 files -- census test_loc undercounts by ~36%** (same glob-miss pattern as the crush/cline anchor artifacts). Shallow clone: no activity claims made either direction; head is 7 days old.

**Anchor question:** closer to **cline** -- a single-surface agent whose tested approval loop, real-but-default-permissive enforcement, and CI rigor are the same register; Grinta then out-clines cline on verification (mutation + eval workflows) and OS sandboxing, and out-clines it on compaction QA, while under-clining it on interop (no SDK/IDE surfaces) and durability (solo maintainer vs 350 contributors).

## Core loop (line-level read)

- Two-plane split, documented at the source of the naming confusion: per-step LLM agent `Orchestrator` in `backend/engine/orchestrator.py:68` vs session control plane `SessionOrchestrator` in `backend/orchestration/session_orchestrator.py` (`orchestrator.py:3-6`). Protocol-first: planner/executor/safety/memory are Protocols, `NoopSafetyManager` default (`orchestrator.py:41-45,91`).
- Step is a typed error cascade (`backend/engine/orchestrator_helpers/step.py:267-297`): ContextLimitError -> reactive peel -> condense -> retry; ToolExecutionError -> diagnostic; malformed-call shapes -> recoverable handler; provider errors re-raise with counter reset. Circuit breaker at `step.py:188-197` with a last-resort graceful-degradation ladder (`step.py:225-265`): shrink largest tool outputs, trim old error observations, re-condense, only then `AgentRuntimeError`.
- Reactive overflow peel before full condensation: `step.py:146-176` (`peel_oldest_api_round_groups`, gated on >=500 tokens saved).
- Sync/async discipline is explicit: `_step_sync` refuses silent loop-orphaning fallbacks (`step.py:300-350`, the docstring records why the old fallback was removed).
- Event stream is the sole controller<->agent channel (`orchestrator.py:13`); durable writer with WAL-first enqueue (`backend/ledger/stream/durable_writer.py:155-188`), persistence health states ok/degraded/failed (`event_stream.py:617`), session lock acquisition (`event_stream.py:98`), backpressure snapshot (`event_stream.py:434`).
- No god files: largest non-test py is `backend/cli/tui/widgets/scan_line/cards.py` 1555 LOC; loop files are all <650 (`orchestrator.py` 460, `step.py` 522, `state.py` 980). Well under the errata's 5k-LOC docking factor. Mixin sprawl (`executor_mixins/`, `event_rendering/*_mixin`) is the main architectural smell; `SessionOrchestrator` service swarm (26 files under `backend/orchestration/services/`) is granular but shallow.

## Compaction / context (line-level read)

- Six-layer composition pipeline run in sequence, each dimension one layer: microcompact body-clearing, snip cap, LLM summary, recent-keep, post-compact file reattach, reactive overflow drop (`backend/context/compactor/strategies/composition_pipeline.py:1-11,78-80`). Microcompact is age-based shedding of `CmdOutput`/`FileRead` bodies with cleared-id bookkeeping (`microcompact.py:13-19`).
- Budget triggers measure the projected boundary, not raw history: `ContextBudget.from_events` computes effective_window minus fixed-prompt reserve minus reserved-summary tokens (`backend/context/context_budget.py:30-59`).
- Cache discipline: Anthropic `cache_control` on system + last tool def (`backend/inference/mappers/anthropic.py:40-80`), four-mode vendor cache resolution incl. gateway providers (`backend/inference/caching/prompt_caching.py:11-26`), and -- the interesting one -- **absolute-position cache anchors** on the 4th/8th user messages with an explicit monotonicity comment: anchors do not shift as messages append, so prefix reuse survives across turns (`backend/engine/memory_prompt_cache.py:10,29-33`).
- **Compaction continuity gate** (`backend/context/continuity_eval.py`): scores whether high-value facts survive condensation; `test_result`, `failed_approach`, `failed_outcome` are blocking loss categories, transient `error` deliberately non-blocking with the trade-off argued at `:20-36`; deterministic fallback summary retains failed approaches; gated by `DEFAULT_CONTINUITY_GATE_MIN_SCORE`. Tests assert the gate blocks missing test results and demotes non-critical misses to telemetry (`backend/tests/unit/context/test_continuity_eval.py:69,90,102`). Nothing in the anchors does this.
- Pre-condensation snapshot builder 1250 LOC (`pre_condensation_snapshot.py`) with file cap aligned to continuity checks (`:41`).

## Permissions / sandbox (line-level read)

- Three execution profiles: `standard` / `hardened_local` / `sandboxed_local` (`backend/core/config/security_config.py:45-57`); default is **standard** (no interposition).
- `sandboxed_local` wraps argv per-OS: bubblewrap with `--die-with-parent`, workspace bind or `--ro-bind` (`backend/execution/sandboxing.py:76-126`); hand-written seatbelt policy with `(deny default)`, explicit read/write subpaths, `(deny network*)` unless `allow_network_commands` (`:132-176`); Windows via a 446-line AppContainer runner module (`:180-193`, `sandbox_helpers/appcontainer_runner.py`). **Fail-closed**: missing backend raises instead of falling back (`:78-81,134,207,219,237`) -- the opposite of the nanocoder fail-open anti-pattern.
- `readonly_workspace` mounts the tree immutable rather than classifying shell commands, with the reasoning stated verbatim: shells do both read and write, classification is unreliable, make the filesystem immutable instead (`security_config.py:67-80`) -- designed for read-only delegated workers.
- Policy layer is pattern heuristics, not AST: shlex + regex with de-obfuscation that reduces `$(printf %s rm) -rf /` and takes worst-of raw vs normalized risk (`backend/security/command_analyzer.py:1-4,42-51`; `bashlex` is a dep but used in `execution/utils/shell/bash_support.py`, not the analyzer). Tests exist incl. Windows (`tests/unit/security/test_command_analyzer.py`, `test_windows_security.py`).
- `SafetyValidator` claims to bind even in full autonomy, blocking critical/high and optionally queueing for human review (`backend/orchestration/safety_validator.py:1-5,45-52`); autonomy ladder conservative/balanced/full (`backend/core/autonomy.py:21-23`); workspace trust store with atomic write (`backend/core/workspace_trust.py:14,38-41`) surfaced via a first-run prompt (`cli/workspace_trust_prompt.py`).
- Honest gaps stated in code: **interactive terminals intentionally bypass process isolation** (`sandboxing.py:8-10`) -- a real boundary Grinta does not hide, but it is a boundary.

## Verification

- 764 test files / ~163k LOC under `backend/tests` (710 unit, 30 integration, 4 e2e, 7 stress). Names read like property specs: `test_no_step_progress_watchdog.py`, `test_confirmation_state_ownership.py`, `test_destructive_command_middleware.py`, `test_compaction_continuity_gate_blocks_missing_test_result` (`test_continuity_eval.py:69`).
- CI: sharded coverage gates on Linux + required cross-platform unit gates on Windows/macOS + extended integration/e2e tiers (`py-tests.yml:29-34,36-`), CLI regression matrix 3-OS (`e2e-tests.yml:17-21`), bandit, CodeQL, pip-audit, dependency-review, **mutation testing via mutmut on every PR and main push** (`mutation-testing.yml:5-9,38-46`), plus **model evals in CI**: DeepSWE protocol runs on release publish and label-triggered PRs (`run-eval.yml:4-10,41`). Per the frozen errata, file:line eval-in-CI evidence clears the verification-9 bar; nuance: evals are label-gated, not a per-PR mandatory gate. No fuzzing/hypothesis anywhere (grep empty).
- Replay with divergence detection: replays trajectories and compares hashed observations against the original, recording `ReplayDivergence` (`backend/orchestration/replay.py:22-47`).

## Orchestration / operability

- Subagent delegation via first-class `DelegateTaskAction`/`DelegateTaskObservation` (`ledger/action/agent.py:311`, `ledger/observation/agent.py:144`) with TUI rendering (`cli/event_rendering/delegate.py`), blackboard (`orchestration/blackboard.py`), action scheduler, retry queue, iteration guard, circuit breaker service.
- Stuck detection well beyond crush's breaker: repeating action/observation patterns (`stuck/detector.py:263-281`), **semantic-loop check via intent diversity and failure rate** (`:339-455`, `classify_shell_intent`), token-repetition (`:512`), **cost-acceleration** (`:541`), think-only loops (`:583`); tested (`tests/unit/orchestration/test_stuck_detector*.py`).
- Rate governor tracks the most conservative observed TPM limit per provider/model and throttles token generation accordingly (`orchestration/rate_governor.py:50,114-122`) -- cost visibility plus active shaping.
- Checkpoint/rollback: manifest-based workspace checkpoint with save/restore and quarantine of workspace extras (`execution/rollback/workspace_checkpoint.py:72,109,219-263`) + ShadowGit pinned dep (`pyproject.toml` shadowgit @ git+sha). Sessions: `/resume /sessions /compact /retry` (`cli/repl/slash_command_dispatch.py:26`, `run_helpers_dispatch.py:157`), non-interactive mode blocks TTY-only commands (`repl/noninteractive.py:181,210`), doctor CLI (`cli/doctor/`), terminal sanitize/restore modules.

## Interop

- MCP client only (persistent session, tool aliases, error collector -- `backend/integrations/mcp/`, fastmcp/mcp deps); no MCP server. No ACP. No published SDK. VS Code workflow exists but is disabled: "repository no longer contains VSCode extension assets" (`vscode-extension-build.yml:4-5`). LSP client 1173 LOC (`utils/lsp/lsp_client.py`). Broad provider plane (openai/anthropic/google-genai + catalog) and a standout: **Codex app-server as model transport** -- reuses ChatGPT managed OAuth and the subscription Responses endpoint while Grinta owns loop/tools/approvals (`backend/inference/clients/codex_app_server.py:1-8,29`). Headless non-interactive CLI with a command safety taxonomy.

## Originality / docs / durability

- Unique-or-rare in corpus: continuity gate on compaction; absolute-position cache anchors; replay divergence hashing; cost-acceleration stuck signal; validated finish gate -- composite `TaskValidator` (test-passing, diff, file-exists, LLM evaluator) so the agent cannot declare FINISHED on vibes (`backend/validation/task_validator.py:1-6,149,166,249,430,564,711`); honest eval framing (case study "must not be presented as a DeepSWE leaderboard score", `evaluation/deepswe/README.md:3-10`).
- 45-item docs tree incl. ARCHITECTURE (256 lines), ADRs, CI tier map, TROUBLESHOOTING, VOCABULARY; a narrative "Book of Grinta" journey corpus (`docs/journey/`) -- unusual, some puffery risk, but the technical docs match the code.
- Durability: single lead maintainer (`MAINTAINERS.md:7`), contributors=1; but a full governance kit (GOVERNANCE, CODE_OF_CONDUCT, SUPPORT, SECURITY with 48h/1wk/2wk disclosure SLAs and safe-harbour, SECURITY.md:9-33) and release infrastructure (PyPI, GHCR, release-drafter). Shallow clone -- no history-based claims.

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 8 | orchestrator.py:3-13 protocol-first planes; step.py:267-297 typed cascade; max non-test file 1555 LOC; mixin sprawl in `cli/event_rendering/` |
| verification | 9 | mutation-testing.yml:38 every-PR mutmut; run-eval.yml:4-10 release/label DeepSWE; 764 behavioral test files; no fuzzing |
| safety-enforcement | 7 | fail-closed 3-OS sandbox sandboxing.py:78-81,132-176; readonly_workspace rationale security_config.py:67-80; but default profile standard (:45) and interactive bypass sandboxing.py:8-10 |
| token-economy | 8.5 | composition_pipeline.py:1-11 six layers; memory_prompt_cache.py:29-33 monotonic cache anchors; continuity_eval.py:20-36 blocking fact-survival gate; context_budget.py:30-59 projected trigger |
| orchestration | 8 | DelegateTaskAction ledger/action/agent.py:311; replay.py:22-47 divergence hashing; durable_writer.py:155-188 WAL; detector.py:541 cost-acceleration |
| interop | 7 | MCP client only; codex_app_server.py:1-8 subscription-as-transport; vscode workflow disabled vscode-extension-build.yml:4-5; no ACP/SDK |
| operability | 8 | workspace_checkpoint.py:72,109 + quarantine :219-263; event_stream.py:617 persistence health; /resume+/compact; doctor CLI |
| originality | 8 | continuity gate, cache anchors, finish-gate validators, cost-acceleration loop signal -- all verified in code |
| durability | 5 | MAINTAINERS.md:7 solo lead vs SECURITY.md/GOVERNANCE kit and real release infra; shallow clone noted |
| docs-dx | 8 | docs/ 45 items, py-tests.yml:29-34 CI-tier map matching docs/CI.md; journey docs inflate volume slightly |

**Weighted total: 78.5 -- band A** (12 + 13.5 + 7 + 8.5 + 8 + 7 + 8 + 8 + 2.5 + 4).

Strongest dimension: **verification** (only subject so far with mutation testing AND model evals in CI, each with workflow file:line). Weakest: **durability** (single maintainer, bus factor 1, despite an unusually mature governance kit for a solo project).

Census correction: test_loc 119,774 -> ~162,854. Provenance: census "original" accepted; grep for openhands/opendevin residue in code is clean (docs/journey mentions are comparisons only); vocabulary (EventStream, CondensationAction, AgentSkillsRequirement) is OpenHands-lineage design but no code remnant found -- flagging for synthesis only, not asserting a fork.

Calibration notes: verification 9 awarded strictly per the frozen errata (eval-in-CI file:line = run-eval.yml; precedent: openhands-2); the eval trigger is label/release-gated rather than mandatory, noted in finding nuance. Total sits exactly on the A boundary: any single-rung drop on verification or safety moves it to B. No calibration demotions applied.
