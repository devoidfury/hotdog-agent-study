# Boundary re-review: gptme (independent pass)

Provisional was 76.0 (B), 2.0 under the A floor. Protocol + anchors + ERRATA read (incl. verification-8 ceiling and architecture god-file docking). Anchor statement: **closer to cline (76.5, B)** - broad feature surface, big tested core, strong interop, but architecture docked by large fused files and duplicated host paths; nowhere near pi's loop/state separation or codex's crate discipline.

## Swing 1: evals-in-CI - blocking or not? **NOT blocking. Verification stays 8.**

`eval-ci.yml` runs 5 real model evals (hello, prime100, fix-bug, hello-patch, init-git via Haiku) on every non-draft PR touching core paths, but it has three independent off-switches, any one of which voids gate semantics:

1. `.github/workflows/eval-ci.yml:28` - job-level `continue-on-error: true`. This is decisive on mechanics: a continue-on-error job **always reports green to the checks API**, so even a server-side branch-protection required-status-check on it would be permanently satisfiable. The repo also self-documents this: `:27` "# Non-blocking in Phase 1 - informational only", and every result branch of the PR comment posts "Informational only - does not block merge" (`:136, :149, :173, :200`).
2. Fork PRs skip evals entirely - no `ANTHROPIC_API_KEY`, step `check_key` (~`:41-58`), and the comment says so.
3. `:165` `fast_fail = passed_count == 0 and ... avg_duration < 5.0` - an all-fail run is reclassified as "skipped (API unavailable)" rather than a gate failure.

The ERRATA's 9-path wants evals-in-CI as defense, not telemetry. A quality regression can merge with 0/5 evals passing and every check green. That is `quality-gates-unwired` (concept already used for cursor-agent, groq-code-cli), not a gate. Meanwhile the ordinary suite IS blocking (`test.yml:113,187` `make test` on push/PR; matrix `continue-on-error` only for `experimental`, `test.yml:276`), and tests assert properties (e.g. `tests/test_guardrail_hook.py:24,65-90` - guardrail-over-allowlist regression for gptme#3598). **8 with the observed ceiling; 9 no.**

## Swing 2: fail-open autonomous-confirm - below the 6-rung? **No. Safety stays 6; hole filed, no demotion.**

The path is real: `get_confirmation()` auto-confirms when no `TOOL_CONFIRM` hook is registered (`gptme/hooks/confirm.py:228-231`) and after a declined chain (`:271-275`), with `default_confirm=True` as the signature default (`:190`). But context makes it a documented mode contract, not a wrong default:

- Only reached by explicit mode: `init_hooks` registers `cli_confirm` for interactive and `server_confirm` for server; "Non-interactive (autonomous): no confirmation hook (auto-confirm behavior)" - `gptme/hooks/__init__.py:125-127, 254-262`.
- `SECURITY.md:37` is honest about it ("Non-Interactive Mode: Use only in trusted, isolated environments"), pi-style honesty.
- Interactive hook never falls through: `cli_confirm.py:53-160` returns a `ConfirmationResult` on every path (confirm/skip/edit); a declining user gets `skip`.
- Hook *crashes* fail CLOSED: exception in any confirm hook returns `skip` ("On error, skip to be safe", `confirm.py:266-269`) - the opposite of nanocoder's jail-to-plain-sh fail-open (`bash-executor.ts:83`) that dropped nanocoder below 6.
- Third-party guardrails sit *above* the auto path in priority order and are tested able to block even allowlisted commands (`tests/test_guardrail_hook.py:90`), and run even in headless mode (`:143-158`).

The genuine hole: `gptme/cli/main.py:1082-1088` - non-TTY stdin **plus** argv prompts silently flips `interactive=False, no_confirm=True` with only a `logger.info`. A user invoking `gptme "do X"` in a CI/pipe context gets every tool auto-confirmed without ever passing `--no-confirm`. Filed as `autonomous-confirm-failopen` (concept exists from other subjects), med impact. Verdict: 6-rung holds ("tested approval, nothing underneath" - sandbox exists, firejail/bwrap/docker/wasmtime, but default `none`, `gptme/sandbox.py:98`), exactly the cline/pi/crush rung.

## Swing 3: architecture 8 vs 7. **7.**

- Dual step engines: `gptme/chat.py:896 step(...)` (CLI/TUI) and `gptme/server/session_step.py:741 step(...)` (server streaming loop). They share primitives (`executor.py:29-33`, `_chat_complete/_stream`, `ToolUse`) - this is not nanocoder-grade per-entry reimplementation - but the loop *semantics* are written twice. The 8->9 test per ERRATA is "loop/state model separated from every product surface"; gptme fails the stronger form and even the weaker form more than cline did.
- `gptme/server/api_v2.py` = 4,055 LOC holding 39 Flask routes *plus* business logic: webui deploy config (`:234`), OpenRouter audio transcription (`:201`), secret redact/restore (`:575, :582`), fork persistence (`:369-462`). Routes+wiring+logic fused = crush's `agent.go` docking shape, at 1.7x the size. Splitting has started (`api_v2_sessions/agents/common.py`) but api_v2.py itself is still the grab-bag.
- `gptme/tools/shell.py` = 3,701 LOC; `computer.py` 2,960. Under the ERRATA's 5k product-file docking bar, but the ERRATA's 5k is a ceiling marker, not amnesty at 4k when combined with the loop duplication above.
- Credit where due: hooks subsystem, tools registry, `logmanager`, `context/`, `tools/autocompact/` are genuinely clean boundaries; `chat.py` at 1,054 is a readable loop. That is why this is a 7 and not lower - crush (7) had the same fused-wiring complaint at smaller sizes and no hook-layer of this quality.

## Dimension calls (independent)

| dim | score | key evidence |
|---|---|---|
| architecture | 7 | dual step engines chat.py:896 / session_step.py:741; api_v2.py 4055 fused |
| verification | 8 | 407 test files / ~219k LOC; blocking `make test` (test.yml:113,187); evals in CI non-blocking (eval-ci.yml:28); no fuzzing found |
| safety-enforcement | 6 | approval in-loop + tested; sandbox default "none" (sandbox.py:98); honest SECURITY.md:37 |
| token-economy | 9 | decision tiers none/rule_based/summarize (decision.py:20) + MIN_SAVINGS_RATIO cache-invalidation guard (decision.py:18) + verified-shrink overflow retry (chat.py:865-872) + master-context byte-range recovery (util/master_context.py:20-26) |
| orchestration | 7 | subagents with wall-clock budgets (tools/subagent/api.py:1157) + steer/cancel control hook; circuit_breaker.py for tool/API failures; resume via logs |
| interop | 8 | MCP client + dynamic load/search (tools/mcp.py:8-10); ACP package (packages/gptme-acp); server+webui+TUI+headless `--output-format json` (cli/main.py:632-636); rival-harness memory import - reads Claude Code memory as a layer (memory/roots.py, "cc" layer) |
| operability | 8 | --resume, backtrack, workspace checkpoints+snapshots with tests (test_checkpoint.py, test_workspace_snapshot.py), doctor, telemetry |
| originality | 8 | verified-shrink compaction retry, model attestation (model_attestation.py), anti_slop.py, context scout; byte-range master recovery (already a canonical concept elsewhere, so 8 not 9) |
| durability | 7 | HEAD 2026-09-27 active, 18 workflows, SECURITY.md; census contributors=1 but shallow clone - evidence-limited on history/bus-factor |
| docs-dx | 8 | 198 in-repo docs, AGENTS.md/DOMAIN.md/SECURITY.md, demos |

**Weighted: 76.0 -> band B. A floor NOT crossed.** Each of the three swing questions independently resolves against promotion: the eval gate is structurally incapable of failing (mechanics of `continue-on-error`, not just policy), the confirm fail-open doesn't demote below 6 (so no offsetting loss either), and the architecture docking is well-founded. Consistent with cline at 76.5 (B).
