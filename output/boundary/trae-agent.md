# Boundary re-review: trae-agent (ByteDance)

Independent pass, 2026-09-30. Protocol + anchors (incl. ERRATA) read; subjects/scores/findings for trae-agent NOT read. Read-only static review. `.git/shallow` present; HEAD 2026-02-05 is recent regardless, no activity inferences needed. App LOC ~9,022 (trae_agent/ + server/), test LOC 1,708 across 10 files; server/ is a Readme stub.

**Anchor sentence:** closer to nanocoder (60.5, C) than to codel (23.0, D) -- same shape (single standard loop, opt-in sandbox, MCP client, mocked-provider tests) but trae-agent sits below nanocoder on every verification/safety/token-economy rung the anchors document.

## Verdict 1: verification -- 6 is NOT defensible; it is 4

What CI actually runs:
- `.github/workflows/unit-test.yml:34-36` runs `make uv-test` on push/PR (plus a pre-commit workflow). That is the entire verification surface: no eval job, no coverage gate, no fuzzing (`grep eval .github/workflows/*.yml` = 0 hits, despite a real SWE-bench-style harness in `evaluation/run_evaluation.py`).
- `Makefile:42` hard-sets `SKIP_OLLAMA_TEST=true SKIP_OPENROUTER_TEST=true SKIP_GOOGLE_TEST=true`. These feed `unittest.skipIf` at `tests/utils/test_ollama_client_utils.py:22-24`, `tests/utils/test_openrouter_client_utils.py:23-25`, `tests/utils/test_google_client.py:23-25`. The "Makefile:42 skips 3 provider suites" allegation: **true** -- 3 of 10 files / 563 of 1,708 test LOC (33%) can never execute in CI.
- Worse, the skipped corpus is rotted: `test_google_client.py:28,45,83,111` patch `trae_agent.utils.google_client.genai.Client`, but the module is `trae_agent.utils.llm_clients.google_client`. The patch targets do not exist; un-skipping would fail on every test. Zero feedback signal, confirmed independently of the skip.
- Core loop has zero coverage: `execute_task` while-loop (`base_agent.py:163-185`), `_run_llm_step` (`:205-236`), `_tool_call_handler` (`:313-352`) -- no test in `tests/` references any of them (grep: no hits for `execute_task|_run_llm_step|_tool_call_handler`). `tests/agent/test_trae_agent.py` exercises only TraeAgent helpers (git diff, patch filtering, tool list init, property protection) with `LLMClient` mocked out (`:35-37`).
- What CI does run: real behavioral specs for edit/json-edit/bash tools, config parsing, MCP client (`tests/tools/*`, `tests/utils/test_config.py:1-236`, `test_mcp_client.py`) -- ~1,145 effective test LOC against ~9k app LOC.

Placement: the 6 rung ("solid, one standard design, tested") requires the loop's semantics to be under test; they are not, and a third of the corpus is CI-dead with evidence of rot. The 4 rung ("works on happy paths, known gaps") is the honest fit: real CI, real tool-level specs, but the loop itself -- including the substring completion detector (`base_agent.py:261-270`) and the reflection injection path -- is asserted by nothing. **Verification = 4.** The claimed 5-vs-6 hinge is moot; even a generous 5 overstates.

## Verdict 2: --docker is a phantom safety control; safety stays 3

- Mechanism: with any `--docker-*` flag, `base_agent.py:65-71` builds a `DockerToolExecutor` with a hardcoded allowlist `docker_tools=["bash", "str_replace_based_edit_tool", "json_edit_tool"]` (`base_agent.py:68`). The claim of exactly 3 routed tools: **true**.
- Escape by design: `docker_tool_executor.py:61-65` -- any tool name not in the set silently falls through to the host `ToolExecutor`. MCP-discovered tools are appended to the same tool list (`trae_agent.py:70`) and so execute **on the host**, as does `ckg` (and any future tool). `--docker-container-id` (README:194) also gives no containment guarantee over what the agent's own host-side tools touch.
- Default mode has no gate at all: no approval/permission/confirm flow exists anywhere in `trae_agent/` (grep for approv/permission/confirm: only unrelated Docker-daemon error text, `cli.py:78,398`). Bash runs straight on the host unless the user opts into Docker.
- Advertising: README.md:181-203 sells "Docker Mode" as "run the task in a new container" with no statement that non-allowlisted tools bypass the container. Not an outright lie (codel's 2-rung "reads misleadingly"), but the containment is materially narrower than the feature description implies.
- Secondary: host-side command assembly for in-container edit tools uses naive single-quote interpolation (`docker_tool_executor.py:117-127`) -- a `'` in file content breaks the command; executed inside the container so it is mostly correctness, not escape.

Placement: below nanocoder's 5 (its jail covers every bash exec, just opt-in; trae's covers 3 named tools and MCP walks around it) and above codel's 2 (real container mechanism exists, honestly framed as a mode). **Safety-enforcement = 3.** Worth a `safety-hole` finding under the existing `phantom-safety-control` concept: yes.

## Verdict 3: token-economy -- unbounded history + blind retry, both confirmed

- History is append-only: `anthropic_client.py:66` (`self.message_history + anthropic_messages` with `reuse_history=True` default) and appends at `:111,:122`; the loop never trims (`base_agent.py:163` feeds the same growing list). No compaction/summarization/trim exists anywhere (grep compact|summariz|truncat: only tool-output truncation).
- Only mitigations: per-tool output clamp `maybe_truncate` (`run.py:17-26`, `edit_tool_cli.py:42`) and token usage accounting (`base_agent.py:281-288`).
- Blind retry into context errors: `retry_utils.py:34-50` catches **all** exceptions (`:37`), sleeps a random 3-30s (`:44,:50`), retries up to `max_retries` -- default 10 (`legacy_config.py:138`, every provider in `trae_config.json.example`). Applied to provider calls at `openai_compatible_base.py:142-147` and per-provider clients. A deterministic context-length 400 is retried ~10 times (up to ~5 minutes of sleeps) before crashing the run (`base_agent.py:170-175` breaks the loop on exception). Confirmed.

Placement: codel's 1 has a crude "ask the user" guard; trae has tool-output clamps and usage accounting but zero context-window management and a retry layer that actively worsens the overflow death-spiral. **Token-economy = 2** ("present but misleading or broken").

## Scores (lane)

| lane | score | note |
|---|---|---|
| architecture | 5 | clean agent/tools/utils layering, max file 722 LOC, no god-file; but naive loop model (string completion, vestigial state enum) |
| verification | 4 | see verdict 1 |
| safety-enforcement | 3 | see verdict 2 |
| token-economy | 2 | see verdict 3 |
| orchestration | 2 | max_steps cap only; no resume/subagents/queue/loop-detection; exception terminates run (`base_agent.py:170-175`) |
| interop | 4 | MCP client real and CI-tested (`tests/utils/test_mcp_client.py`), trajectory JSON (`docs/TRAJECTORY_RECORDING.md`); no ACP/SDK/IDE; server/ is a stub |
| operability | 4 | interactive mode + trajectory recorder + config examples; no sessions/resume/rewind |
| originality | 3 | CKG sqlite code-graph tool (`ckg_database.py`, 722 LOC) and lake_view are real in code; core loop is open-interpreter/OpenHands-lineage boilerplate (`run.py:17-26` verbatim pattern) |
| durability | 5 | ByteDance backing, CI gated to `bytedance/trae-agent` (`unit-test.yml:13`); no SECURITY.md; shallow clone noted |
| docs-dx | 4 | rich README command surface, 4 docs/ files incl. config migration; no SECURITY.md |

**Weighted total: 36.0 -- band D.**

## Calibration notes

Provisional 44.0 is not reachable on evidence. Even accepting Phase 1's other lanes at my generous readings, verification 6 -> 4 costs 3 points and safety 4 -> 3 costs 1 (=> ~40 at best on their base); my independent full re-derivation lands at 36.0. The 5-vs-6 verification hinge is resolved decisively against 6: no loop coverage, a third of the test corpus permanently skipped via `Makefile:42`, and patch-target rot in the skipped suites proving those tests have not run anywhere for some time. **The C floor (45) is not crossed; it is not within 5 points of being crossed.** No fork/archived calibration rules apply.
