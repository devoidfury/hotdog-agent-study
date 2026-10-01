# gptme — T2 deep review

Python, MIT, upstream https://github.com/gptme/gptme. Shallow clone (`.git/shallow`, HEAD `e14e967` 2026-09-27).
cloc: 381,299 total code; `tests/` 149,816 (406 Python files); `gptme/` package 121,383; webui TS ~54k. Census (202k non-test / 179k test) plausible if webui + `gptme/hooks/tests` specs are counted; census `contributors: 1, commits: 1` is a shallow-clone artifact. Remote check: org-owned, not archived, `pushed_at 2026-09-29` — active; do not read anything from the single-commit HEAD.

## Anchor question

**Closer to pi than to any other anchor**: same design philosophy (one-person-vision CLI agent where hooks/extensions own policy, honest no-overclaim security posture, branch-based session model), but gptme carries cline-like surface breadth (webui, Tauri, MCP+ACP+A2A) and a heavier eval/CI apparatus, while missing pi's loop/product separation and cache warming.

## Core loop (read at line level)

`gptme/chat.py` (1,055 LOC): `chat()` :149 → `_run_chat_loop()` :324 → `_process_message_conversation()` :533 → `step()` :896.

- Clean two-level loop: outer per-user-turn, inner per-step tool continuation (:556 `while True`). Continuation decided by `_has_pending_tooluse` on the last assistant message; mid-turn compaction deliberately preserves the pre-compaction continuation decision because the resumed view no longer ends with the tool call (`chat.py:566-577` comment + `_run_post_tool_compaction` :679). This is a subtle correctness case handled explicitly.
- Hook spine fires at every boundary: SESSION_START/END, TURN_PRE/POST, STEP_PRE, GENERATION_POST, TOOL_CONFIRM, LOOP_CONTINUE (:215, :352, :537, :484, :1026). `LOOP_CONTINUE` implements auto-reply/loop control as a hook, with a `MAX_PROMPT_QUEUE_SIZE` cap (:489-497).
- Durable steering queue: file-based `prompt-queue.jsonl` (prompt_queue.py:25) drained at loop top (:332), with a `prompt-queue-closed` sentinel whose write ordering against the final drain is argued in comments at :230-250 and :407-431 — the race window with concurrent `subagent_steer()` is explicitly reasoned about and belt-and-suspenders handled in `finally` (:310-316).
- Error discipline: provider errors in interactive mode are reported and handed back without killing the session; non-interactive re-raises so exit codes carry the class (:501-513, cites issue #3668). `GPTME_MAX_STEPS` step cap (:540-551, :613-621).
- Overflow recovery `_reply_with_overflow_recovery` :794: on context-length error, compacts to a new view, retries once, but refuses the retry if the compacted *provider* input did not actually shrink (`provider_tokens_after >= provider_tokens_before` → re-raise, :865-870), and refuses if visible streaming output already happened (:833-836, output duplication). Compaction events journaled with before/after tokens both log-level and provider-level (:876-889).
- ContextVar hygiene is careful and documented: token accumulator rebinding for nested/subagent contexts (:93-107), `_url_safety` allow-host var copied per browser-thread (:chat.py:196-199, _url_safety.py:17-35).
- Server path (`gptme/server/session_step.py` 1,731 + `api_v2.py` 4,055) wraps the same `step()` but carries its own mirror logic (`_compact_after_tool_results` referenced from chat.py:683-687). Loop is shared; API surface is a god file.

## Compaction / context management (read at line level)

`gptme/tools/autocompact/` + `gptme/context/` + `gptme/util/context_budget.py`:

- **Trigger**: single resolved budget = `min(0.9×window, window − max_output − headroom)` with env/config overrides and a documented rationale for decoupling budget from window (context_budget.py:1-40). Trigger measured against `get_context_budget` (hook.py:136-140), not raw provider window.
- **Tiering** (decision.py:121-155): `none | rule_based | summarize`. Rule-based only when estimated savings ≥ 10% (`MIN_SAVINGS_RATIO` decision.py:18 — explicitly to avoid invalidating the prompt cache for marginal gains); otherwise LLM summarization on the same trigger; overflow retry is the third tier (chat.py:794).
- **Rule engine** (engine.py:1-40, 3-phase): age-based reasoning-tag stripping; truncate-largest-tool-results-first; extractive compression of old long assistant messages that skips tool-call pairs to keep them parseable (engine.py:206-214). Truncation stops as soon as strictly below target (engine.py:165-169).
- **Lossy view / lossless master**: compaction never rewrites the append-only `conversation.jsonl`; it creates a *view branch* (`create_view`/`switch_view`, hook.py:169-172), and every truncated/compressed message embeds a **byte-range reference into the master log** (`build_master_context_index`/`create_master_context_reference`, master_context.py:26,62; engine.py:66-72, 180-193) so the model can recover the exact original. `keep_head` protection pins task context so a downstream `reduce_log` cannot drop it, including the accepted-overshoot case (engine.py:304-334).
- **Reentrancy/cooldown**: autocompact cooldown keyed by (logdir, branch) so concurrent server sessions and sibling branches don't share a throttle; bounded map with documented pruning (hook.py:30-58). Mid-turn compaction fires after tool results land, not only at TURN_POST (chat.py:679-703).
- **Cache discipline**: `CACHE_INVALIDATED` hook fired on both compaction paths with token deltas (hook.py:180-186, 231-237); `cache_awareness.py` is a context-local cache-state bus: invalidation counters, turns/tokens-since-invalidation, and `is_cache_likely_cold()` TTL prediction (cache_awareness.py:49,160-186) letting plugins trim *before* a predicted cold request; Anthropic `cache_control` breakpoints applied in `llm/utils.py:205-276` for both Anthropic and OpenAI-shaped paths. No cache warming (pi's rung) and no codex-style cache-key gating.
- Also present: `gptme/context/` selector stack (rule/LLM/hybrid file selection, adaptive compressor, scout/task_analyzer) — used for context construction; a parallel second system whose tie to autocompact is looser (tests/test_context_selector.py exists).

## Permission / sandbox code (read at line level)

- **Confirmation is the default binding gate**: every mutating tool routes through `execute_with_confirmation` (e.g. shell.py:3355-3366) → `get_confirmation` dispatches TOOL_CONFIRM hooks in priority order (confirm.py:186-276). A failing confirm hook **skips execution** ("On error, skip to be safe", confirm.py:267-269) — fail-closed. MCP direct executor confirms once at the boundary and marks the ToolUse pre-confirmed so nested calls can't double-prompt or bypass (confirm.py:44-58, 220-222).
- **Denylist binds before confirmation**: `is_denylisted` (shell_validation.py:732) checked pre-confirm and re-checked on user-edited commands (shell.py:3327-3345); sensitive path prefixes force confirmation even for allowlisted commands (shell_validation.py:23-47); shellcheck integration can block (shell.py:3322-3326).
- **Guardrail hook** (guardrails.py:1-40): deterministic TOOL_CONFIRM blocker at priority 200 (above all confirms), targeting the confused-deputy problem of `--no-confirm` runs; three modes shadow/enforce/off — **default shadow**, i.e. logs only.
- **Injection screening** (injection_screening.py:1-77): screens shell/read/browser/gh/mcp outputs, HIGH/LOW severity patterns, modes off/warn/block — **default warn**, prepends [UNTRUSTED] markers rather than blocking.
- **Sandbox** (sandbox.py): firejail/bwrap for shell, Docker/Wasmtime for Python, with env-var allowlist that deliberately excludes provider API keys (sandbox.py:53-60), tmpfs home, workspace-only bind, `--unshare-pid --new-session --die-with-parent` (sandbox.py:378-390), ulimit-v memory ceiling hardened against `BASH_ENV` pre-exec escape (sandbox.py:214-260) with pre-flight verifiability (`verify_memory_limit` :262-286). **Default backend is "none"** (sandbox.py:98,117) — opt-in, Linux-only for shell backends. Docstring states in-scope/out-of-scope honestly (:26-31). 118 tests in `tests/test_sandbox.py`.
- **Hook trust scoping**: project-configured shell hooks and context-cmd gated behind a TOFU trust DB keyed by a hash of the command set (config/trust.py:9,179; core.py:80-83).
- **Computer-use gate**: read/write/sensitive risk tiers, sensitive actions blocked in non-interactive sessions unless explicitly enabled (opt-in) (_computer_gate.py:1-35); URL host allowlist with deliberate `*` rejection and empty-string-is-blocks-everything semantics (_url_safety.py:38-57, HEAD commit #3954) — tests: `tests/test_url_safety.py`, `test_guardrail_hook.py` (15 tests prove a priority-200 hook blocks an otherwise-runnable `curl evil.com` end-to-end), `test_shell_allowlist_autoconfirm.py`, `test_computer_gate.py`.
- **What does not bind by default**: nothing kernel-enforced; in `--no-confirm` mode with no TOOL_CONFIRM hook registered, everything auto-confirms (confirm.py:228-236) unless you opt into `GPTME_GUARDRAILS=enforce` + `GPTME_SANDBOX=...`. SECURITY.md and sandbox docstrings say this honestly.

## Verification

407 test files, ~150k LOC in `tests/` + 5.4k in `gptme/hooks/tests` + webui Jest/Playwright. Tests assert properties, not existence: e.g. `test_auto_compact.py` (77 tests) pins savings-estimate phase gating (:553-670), pinned-message preservation (:162), suffix-accumulation on repeat compaction (:175-260); guardrail tests drive real `ToolUse.execute()`. Built-in offline `mock` provider (`llm/llm_mock.py:1-14`) is a product feature for no-network loop testing. **CI**: 19 workflows; `eval-ci.yml` runs 5 real evals (hello, prime100, fix-bug, hello-patch, init-git) with claude-haiku on every non-draft PR touching core paths, posts a PR comment (:85-96) — but `continue-on-error: true` (eval-ci.yml:28) makes it informational ("Phase 1"), and fork PRs skip for lack of API key. `self-heal.yml` analyzes failed test runs and posts fix proposals via `workflow_run` (self-heal.yml:1-27). `benchmark.yml`, `model-freshness.yml`, coverage→Codecov on three matrices. No fuzzing found (single hypothesis mention in tests/test_tools.py). Per the errata the evals-in-CI rung exists but as a non-blocking gate it does not clear 9.

## Orchestration / interop / operability

- Subagents: `gptme/tools/subagent/` (api 1,992 + concurrency + persistence + batch); global semaphore from env>config>CPU default (concurrency.py:21-57); subagent metadata persisted and **rehydrated across restarts** (`scan_rehydrate_subagents`, persistence.py:186); steering into running subagents via the prompt queue with sentinel protocol (chat.py:230-250). `k_best.py` K-candidate generate-and-score framework (:1-40). No per-run token budgets on subagents found. `cmd_agents`/agent service manages background fleet; `workspace_agents.py` process-scans the machine and *warns about concurrent rival agents* (claude, codex, aider, goose, opencode, amp) sharing the workspace (:1-16) — novel collision-avoidance.
- Interop: MCP client **and** server (mcp/client.py, server.py, cli/cmd_mcp_serve.py), ACP adapter/agent/client (gptme/acp/ + `packages/gptme-acp` npm), A2A JSON-RPC surface (server/a2a_api.py:1), versioned REST v2 with generated OpenAPI docs (server/api_v2*.py, openapi_docs.py), webui + Tauri desktop, headless `--output-format json` gated behind `--non-interactive` (cli/main.py:1356-1363). Imports rival histories (Cursor sessions, tools/chats.py:154-167); consumes rival skill formats (Claude Code/Codex `SKILL.md` exposed as `/<name>` commands with collision rules, lessons/skill_commands.py:1-15).
- Operability: LogManager views/branches (master never destroyed, compaction forks are automatic), `undo` (manager.py:756), conversation checkpoints + `/backtrack` (conv_checkpoints.py:1-16), git-backed workspace checkpoints with honest refusal of non-git/multi-root and explicit `--include-dirty` (checkpoint.py:9-16), fsync + directory-barrier durability with a carefully-chosen tolerated-errno set that refuses to forgive EACCES/EPERM (durability.py:13-27), per-logdir platform locks (manager.py:333-445), event log, model trace, `doctor` (1,538 LOC), crash-recovery prompt logic (_should_prompt_for_input, chat.py:707-740), auto-naming in bounded background threads (chat.py:66-105).

## Originality / durability / docs

Unique-in-corpus mechanisms (verified in code): master-context byte-range recovery from lossy views; predictive cold-cache TTL bus; k-best decision framework; **model-selection provenance attestation** (model_attestation.py:1-15: which model was requested, how aliases resolved, what evidence exists about the serving backend) and output attestation with canonical-JSON hashing (attestation.py); anti-slop writing gate calibrated on a 1,098-post corpus (anti_slop.py:1-12); shadow→enforce rollout pattern for guardrails; rival-agent process scanning; self-heal CI. Durability: org repo, active at remote, 4.4k stars / 443 forks (GitHub API 2026-09-29), MIT, SECURITY.md + security-advisories workflow, 19 workflows; no corporate funding — pi-tier institutional backing. Docs: 74 entries under docs/ incl. context-compression, hooks, ACP, security, computer-use-warning; `.env.example`, AGENTS.md, tutorial cmd — dense and code-matching.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7 | shared step() (chat.py:896) across CLI/TUI/server; hook spine; view state model; docked: api_v2.py 4,055, shell.py 3,701, computer.py 2,960, server-side loop mirroring (session_step.py) |
| verification | 8 | 407 property-style test files; offline mock provider (llm_mock.py:1-14); eval gate on PRs (eval-ci.yml:85-96) but non-blocking (:28); self-heal.yml:1-27; no fuzzing |
| safety-enforcement | 7 | confirm fail-closed (confirm.py:267-269); denylist pre-confirm (shell.py:3327); TOFU hook trust (trust.py:9,179); tested sandbox+env-allowlist (sandbox.py:53-60,378-390, test_sandbox.py) — but sandbox none-default (:117), guardrails shadow, screening warn: the deep layers are opt-in |
| token-economy | 8 | tiered trim→summarize→overflow-retry with monotonic-shrink guard (decision.py:121-155, chat.py:865); MIN_SAVINGS_RATIO cache-aware floor (decision.py:18); CACHE_INVALIDATED bus + TTL cold-cache prediction (hook.py:180, cache_awareness.py:160); no warming |
| orchestration | 7 | subagent persistence/rehydration (persistence.py:186), semaphore (:21-57), durable steering queue with sentinel race protocol (chat.py:230-250); no budgets |
| interop | 8 | MCP client+server, ACP, A2A (a2a_api.py:1), OpenAPI v2, Cursor import (chats.py:154), cross-runtime SKILL.md (skill_commands.py:3-10); no published core SDK |
| operability | 8 | views+undo (manager.py:756), conv/workspace checkpoints (conv_checkpoints.py:1-16, checkpoint.py:9-16), fsync barrier (durability.py:13-27), locks, doctor |
| originality | 8 | master-context byte-range recovery (master_context.py:26,62); model provenance attestation (model_attestation.py:1-15); k_best; cold-cache bus; rival-agent scan (workspace_agents.py:1-16) |
| durability | 7 | org-owned, remote-verified active (pushed_at 2026-09-29), 19 workflows, SECURITY.md; no funding; contributors=1 is a shallow artifact |
| docs-dx | 8 | 74 docs entries incl. context-compression.rst, computer-use-warning.rst; SECURITY.md honest; doctor/onboard/tutorial |

**Weighted total: 76.0 → band B** (upper). Strongest: token-economy (tied with verification/interop/operability/originality at 8 — token-economy is the deepest single read). Weakest: architecture (7, god files + server-loop mirroring).

Calibration: no fork rule applies (original per census; remote not-fork). No demotions. Boundary note: 76.0 is within 2 pts of the A boundary (78).

## Census sanity

- non_test 202k / test 179k: consistent with cloc (381k total code; tests/ 149.8k + hooks/tests 5.4k + webui specs). No vendored-blob inflation found.
- `contributors: 1, commits: 1, shallow: true` — artifact; upstream is a 2023-created, actively-pushed org repo (verified 2026-09-29). Do not treat as single-maintainer from this clone.
