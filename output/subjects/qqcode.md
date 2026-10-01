# qqcode — T1 review

- Subject: /data/samples/agents/qqcode (Python, package `qqcode`, code tree `vibe/`)
- Census row: non_test 31,458 / test 9,211; 1 contributor; head 2025-12-18; shallow=true; Apache-2.0; provenance `divergent-fork:mistral-vibe`; T1.
- Read-only static review; nothing executed. Shallow clone (`.git/shallow`, 1 visible commit be6a96c "chore: bump version to v1.2.0") — no activity claims made from HEAD.

## Anchor question (one sentence)

Closer to **nanocoder (60.5, C)** than to crush (69, B): same tested-but-thin verification mass and approval-only safety story, but qqcode's core is cleanly separated from product surfaces (no React-hook loop) and its ACP/IDE surface breadth exceeds nanocoder's — net it sits right at the nanocoder rung.

## Provenance / fork delta (mandatory, rule (a) deferred to synthesis)

Census "21 files hash-identical to mistral-vibe" **verified exactly**: of 145 files shared with the local `/data/samples/agents/mistral-vibe` sample tree, exactly 21 are byte-identical, 124 differ, and 494 upstream files have no qqcode counterpart (upstream restructured heavily post-fork; its current shape is for the T3 lane to characterize).

Divergent, not sync: 70 qqcode-only files including entire subsystems absent from the sampled upstream tree —
- plan mode: `vibe/core/tools/builtins/plan_mode.py`, `vibe/core/prompts/plan_mode.md`, gate at `vibe/core/agent.py:896-930`
- subscription OAuth: `vibe/core/oauth/claude.py` (PKCE), `vibe/core/oauth/qwen.py` (device flow), CLI login at `vibe/cli/login.py:21,68`
- Anthropic backend + adapters: `vibe/core/llm/backend/anthropic_sdk.py:280`, `vibe/core/llm/backend/generic.py:507` (OpenAI+Anthropic adapters behind one `APIAdapter` protocol, `generic.py:32`)
- update notifier: `vibe/cli/update_notifier/github_version_update_gateway.py:13`
- full ACP test suite: `tests/acp/` (11 files, `test_acp.py` 882 LOC)
- product surfaces: `vscode-extension/` (webview chat + file indexer), `distribution/zed/`, `action.yml` (GitHub Action)

Scoring on merits as a divergent fork; synthesis runs rule (a) against the mistral-vibe T3 result.

## Census sanity

- `test_loc 9211` undercount: `tests/**/*.py` = 10,977 LOC across 64 files (~1.7k missed, likely glob misses). `non_test_loc 31458` vs 31,763 measured `*.py` — within noise. No tier impact (well inside T1).
- 1 commit is a shallow-clone artifact; head_date 9.4 months is under the 10-month remote-check trigger, and remote was not consulted — durability evidence-limited, no dead/archived claim.

## Code read (core loop / compaction / permissions)

**Loop** — `vibe/core/agent.py:313-377`: while-loop gated on `finish_reason` + role, wrapped by a middleware pipeline (`run_before_turn`/`run_after_turn`, `vibe/core/middleware.py:158-192`) whose result types are CONTINUE/STOP/COMPACT/INJECT_MESSAGE. `middleware.py` ships TurnLimit, PriceLimit (`:72-85`), AutoCompact (`:88-107`), ContextWarning (`:110-137`). History-consistency repair is explicit and tested: `_fill_missing_tool_responses` (`agent.py:992-1032`) inserts cancellation placeholders; `_ensure_assistant_after_tools` (`:1034`). Streaming path batches chunks and reconstructs tool-call indices (`agent.py:430-510`). Cleanest module boundaries in its size class so far: max file `textual_ui/app.py` 1,480 LOC; core has no god file.

**Compaction** — one tier. Trigger uses *measured* context: `stats.context_tokens` is set from billed usage every turn (`agent.py:790-792, 858-860`), threshold = per-model context limit with 200k fallback (`config.py:435,516-521`). `compact()` (`agent.py:1071-1133`) summarizes in-conversation, preserves the last user message (`:1097-1101`), then re-counts post-compact via a real `count_tokens` probe — implemented as a `max_tokens=16` completion (`generic.py:800-827`), so probe cost is a request. Manual `/compact` + observer batching tested (`tests/test_agent_auto_compact.py:19`). No branch summarization, no pre/post-compact hooks.

**Permissions** — default ASK (`vibe/core/tools/base.py:66`); fail-closed to SKIP when no approval callback (`agent.py:955-960`); plan mode restricts to a read-only tool frozenset (`agent.py:69,896`) and applies a stricter bash denylist (`bash.py:211-237`). Bash allow/deny matching is prefix `startswith` over `&&/||/;/|`-split segments (`bash.py:258-300`): deny is fail-closed, but the ALWAYS path has a substitution hole (`date $(rm -rf /)` matches allowlist prefix `date`). No OS sandbox anywhere (`grep -ri sandbox|bwrap|seatbelt vibe/` → zero hits); README "Safety First" refers to approval only, which the code does honor. Approval semantics are genuinely tested (`tests/test_agent_tool_call.py:106,171,209,240`).

**Interop** — ACP server (`vibe/acp/acp_agent.py:81-534`): loadSession `:344`, setSessionMode `:348`, setSessionModel `:363`, per-session approval callbacks mapped onto ACP permission options `:221-336`. MCP stdio + HTTP proxy-tool classes (`vibe/core/tools/mcp.py:8,102-184,211-212`). VS Code extension (`vscode-extension/src/qqcodeBackend.ts`), Zed extension (`distribution/zed/extension.toml`), GitHub Action (`action.yml`), programmatic headless with stdin approval/plan callbacks (`programmatic.py:37-202`).

**Operability** — JSONL interaction journal with git metadata written per turn (`agent.py:341-343`; `interaction_logger.py:32-131`); `--resume`/`--continue` load sessions back (`cli/entrypoint.py:142,335-386`; `interaction_logger.py:349`); `/conversations` browse-and-continue (`cli/commands.py:49-53`). No checkpoint/rewind, no diagnostics surface. Config migration with tests (`tests/core/test_config_migration.py`).

## Verification

~11.0k LOC / 64 files. Tests assert loop semantics, not existence: approval-iff (`test_agent_tool_call.py:106`), NEVER-permission skip (`:209`), ALWAYS-flip (`:240`), interruption (`:382`), placeholder repair (`:421`); auto-compact trigger + observer (`test_agent_auto_compact.py:19`); 882-LOC ACP integration suite + per-tool ACP tests; backend tests ride real provider API-response fixtures through respx, with the fixture-maintenance policy stated in-file (`tests/backend/test_backend.py:1-10`, data for fireworks/mistral/zai). CI: pytest on push/PR (`ci.yml:86`), but the lint gate is neutered (`ci.yml:51` `|| true`) and both snapshot jobs are `continue-on-error` (`ci.yml:92,116`). No evals, no fuzzing — consistent with the corpus verification-8 ceiling being about more than these subjects have; qqcode is below that ceiling on mass and gate integrity.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7 | loop `agent.py:313-377` + middleware `middleware.py:158`; core/cli/acp split; max file 1,480; adapter protocol `generic.py:32` |
| verification | 6 | property tests `test_agent_tool_call.py:106-461`; ACP suite `tests/acp/`; but `ci.yml:51` `\|\| true`, `:92,116` non-blocking |
| safety-enforcement | 6 | default ASK `base.py:66`; fail-closed `agent.py:955-960`; plan gate `agent.py:896`; no sandbox; prefix-ALWAYS hole `bash.py:286` |
| token-economy | 6 | usage-measured trigger `agent.py:790,858`; per-model limits `config.py:516`; cache markers `generic.py:73-120` (untested); single tier |
| orchestration | 5 | turn/price budgets `middleware.py:54-85`; tested cancel `test_agent_tool_call.py:382`; multi-session ACP; no subagents/queues/journals |
| interop | 7 | ACP `acp_agent.py:81-385` + tests; MCP stdio+HTTP `mcp.py:211`; VS Code + Zed + GH Action + `programmatic.py:144`; no published SDK |
| operability | 6 | journal `interaction_logger.py:349`; resume `entrypoint.py:335-386`; migration tests; no rewind/diagnostics |
| originality | 6 | Claude+Qwen subscription OAuth `oauth/`; provider-vs-model config split `PROVIDER.md`; API-fixture test policy `test_backend.py:1-10` |
| durability | 4 | 1 contributor, shallow clone (evidence-limited), solo-maintainer fork; PyPI/nix/release workflows exist |
| docs-dx | 5 | `PROVIDER.md` matches code; onboarding wizard; install.sh; no docs/ tree, no SECURITY.md |

**Weighted total: 60.0 → band C** (7·1.5 + 6·1.5 + 6 + 6 + 5 + 7 + 6 + 6 + 4·0.5 + 5·0.5 = 60.0). Strongest: interop (7; four product surfaces off one ACP core, unusual at 31k LOC). Weakest: durability (4).

## Calibration notes

- Divergent fork verified (21/145 identical); no sync-fork cap applied; rule (a) reconciliation with mistral-vibe (T3) deferred to synthesis per brief.
- No archived/dead claim (shallow clone; head within 10-month window; remote not checked).
- Not within 2 pts of any band boundary (60.0; nearest trigger 65 is 5 pts away).
- Distribution sanity: consistent with nanocoder 60.5 neighbor placement.
