# smelt -- T2 deep review

Identity check: manifest `smelt-agent` (root Cargo.toml install target `cargo install ... smelt-agent`, README.md:64), crates `smelt-*`, remote github.com/leonardcser/smelt ("A fast, Lua-scriptable AI coding agent for the terminal", API `fork: false`). NOT the 2010s Mozilla smelt build system; no overlap in language, shape, or history. Provenance: original, single author (leonardcser), repo created 2026-02-25, pushed 2026-09-28 (remote-verified), MIT.

Shape: Rust workspace, 15 crates (Cargo.toml:2). Headless agent engine (`smelt-engine`) + provider layer (`smelt-provider`) + product core (`smelt-core`: sessions, permissions, tools, Lua API, MCP, LSP) + terminal product (`smelt-tui`: own grid renderer, not ratatui; embedded Vim editor in `smelt-edit`). 18.6k LOC of bundled Lua under `runtime/lua/` implements compaction, modes, tools, and commands as plugins.

## Census sanity

Census test_loc=15,571 is wrong the usual way: there are ~5,622 `#[test]`/`#[tokio::test]` attributes; inline `#[cfg(test)]` modules ≈106k LOC across 268 files, file-local test modules (`*/tests.rs`, harness_tests) ≈57k, plus `tests/` 2.7k => real test LOC ≈166k. Census non_test_loc=397,048 overcounts: total cloc code across ALL languages is 421k; Rust is 372.7k code including tests, so non-test Rust ≈206k (plus 18.6k Lua, ~12k md). Tier stays T2 either way. Shallow clone (`.git/shallow`, 1 commit, head 2026-09-26); remote is active and NOT archived, so no activity demotion applies.

## Core loop (mandatory read)

`crates/engine/src/agent.rs:1556` `run()`: single async loop per turn. Notable, all in-loop:
- Tool defs re-sorted by name every request "so the request prefix stays byte-identical across turns. Anything that reorders tools busts the cache" (`agent.rs:1581-1584`); plugin `override_core` shadowing (`:1593-1607`).
- Pre-request host hook (`prepare_request_with_host`, `:1359`) with restart/continue/abort/cancelled outcomes -- this is where plugin-side auto-compaction intercepts.
- Provider context-window errors do NOT abort: engine calls `HostCall::RecoverFromContextLimit` (`agent.rs:1690-1770`, `engine/src/host.rs:102`), accepts a shortened replacement history, repairs orphan tool calls (`warn_if_replacement_has_orphans`, `:824`), and transparently retries the turn.
- Steering: user injections mid-LLM-call discard the response and re-loop (`:1800-1808`); `UiCommand::Steer/Unsteer` (`:394,:1473`).
- `ToolReplayGuard` (`:907-922`) aborts the turn if the provider replays a tool call from a prior round -- no double side effects.
- Every tool batch goes through `result_dedup::apply_in_place` then `trim::budget_tool_invocations` before history commit (`:1955-1957`).
- Cancel/quota paths commit partial assistant text (`commit_partial_assistant`, `:1284`) and emit retry_at_ms for quota resets (`:1668-1683`).
The loop is 4,211 LOC in one file but cohesive (helpers as methods); no god-file pathology inside the loop itself.

## Compaction (mandatory read)

Implemented as the bundled Lua plugin `runtime/lua/smelt/plugins/compact.lua` (740 LOC) against two engine contracts (`on_prepare_request`, `on_context_limit`, registered at `:638` and `:687`). Properties enforced and tested:
- Pre-request trigger uses an active-context estimate anchored to the last provider-reported token count plus a history delta, falling back to a full estimate past a delta cap (`crates/tui/src/app/host_dispatch.rs:448-478`) -- closest kin to pi's "measure what the model would actually receive", but anchored on provider ground truth.
- Group-boundary summarization keeps a live suffix; checkpoint summaries are recognized and not re-compacted (`is_checkpoint_summary` :149, `has_compactable_group_prefix` :405; harness test `crates/tui/src/app/harness_tests/compaction.rs:507`).
- Circuit breaker: MAX_CONSECUTIVE_FAILURES disables auto-compact after repeated terminal provider failures, `/compact` stays as manual override (`compact.lua:70-97`).
- Unchanged-checkpoint-estimate guard lets exactly one retry through to the provider instead of re-compacting in a loop (`:581-586,:646-652`).
- Summarization itself handles its own context-window overflow (`:284`) and mid-flight cancellation via work guards (`:593,:700`).
- No-LLM tool-output budget separate from compaction: `crates/engine/src/trim.rs:11-44` (per-tool 2k lines/10k tokens, per-turn aggregate 40k tokens, grapheme-safe middle truncation, tested `:131-257`).

## Permissions (mandatory read)

`crates/core/src/permissions/`: rules = per-mode allow/ask/deny tool rulesets + effect classes (read/write/network/process/config/user/other, `rules.rs:30-39`) + subpattern buckets with custom parsers (`rules.rs:9`). Shell commands are not regex-gated: `bash.rs` (2,483 LOC) is a shell-grammar walker (eval_program/list/and_or/pipeline/if/loop, heredocs, embedded commands, variable expansion, `:321-660`) that infers `ShellRisk` and path effects, with unknown constructs escalated to `ShellRisk::Unknown` (`:310`). Decisions bind in the engine loop (`Decision::Allow/Deny/Ask/Error`, `agent.rs:2142-2196`) and concurrent-dispatch path (`:2345-2400`); headless denies anything Ask (`docs/docs/reference/permissions.md:285-288`). Policy snapshots are immutable per turn while runtime approvals stay live (`mod.rs:318-345`); approvals scope to session/workspace/repository with persistence (`approvals.rs:14-40`); project trust gate keyed by content hash (`crates/core/src/trust.rs:45,91`). Tests: `permissions/tests.rs` 4,985 LOC, 337 test fns; fuzzed by `permissions_rules` with a workspace-downgrade oracle (`fuzz/README.md:24`).
NO kernel sandbox anywhere (zero hits for seatbelt/bwrap/landlock/seccomp); the docs say so plainly: "Anything else is defense in depth, not a sandbox" (`docs/docs/reference/permissions.md:292-299`, `SECURITY.md:26-38`). Secrets are redacted at ingress from user input and tool results, never from assistant output (`crates/engine/src/redact.rs:1-13`, applied `agent.rs:500-504`).

## Verification

The one subject in this study where the anchors' 8-ceiling ("no fuzzing") is broken. 17 fuzz targets with named oracles including TUI-event-loop scenario fuzz (`smelt_loop`), Lua FFI ledger (`lua_loop`), Anthropic/OpenAI cache-prefix byte-invariance (`cache_invariance`, `openai_cache_invariance`), store state machine (`store_state`), engine lifecycle (`engine_events`) (`fuzz/README.md:3-27`). Deterministic replay is a product stance: fixed clock + stubbed I/O (`crates/engine/src/clock.rs`, README "Why" section). CI: coverage floor `--fail-under-lines 80` (ci.yml:88), clippy -D warnings, fmt, generated-docs drift gate (ci.yml:94-103), MSRV matrix per published crate (ci.yml:120-186), Windows install smoke, a real-libFuzzer-subprocess tooling test (ci.yml:232-267), and `cargo xtask fuzz verify` replaying committed regression seeds (`fuzz/seeds/*/regression/`, e.g. 6 compaction seeds) on every PR (ci.yml:269-294). Subprocess integration tests drive the real binary against wiremock providers asserting on the JSONL event stream (`tests/scenarios.rs:1-8`). Storybook visual snapshots for transcript/dialog UI (`TESTING.md:47-59`). No in-CI model evals -> not 10.

## Token economy beyond compaction

Tool-name sort as canonical cache helper (`crates/provider/src/cache.rs:28-32`), Anthropic <=4 `cache_control` markers with 5m/1h TTL choice (`provider/src/anthropic.rs:15-29,301-327`), session-scoped `prompt_cache_key` for OpenAI-family (`cache.rs:14-17`), cache-safe result dedup pointers placed only on the new invocation, never on cached history bytes (`result_dedup.rs:1-9`), fuzz targets proving prefix invariance. Cost visibility: pricing per provider with overrideable per-model costs (`provider/src/pricing.rs:17-84`), request audit with export (`src/main.rs:495`), usage command + statusline. No cache warming, no branch summarization.

## Orchestration

No subagents, no queues, no loop-detection breaker. Present: background process supervision as first-class tools (`runtime/lua/smelt/tools/bash.lua, ps, read_process_output.lua, stop_process.lua`), goal state machine with goal-gated auto-continue and quota backoff 60s->300s (`runtime/lua/smelt/goal.lua:7-12`, `auto_continue.lua:4-6`), work guards invalidating stale async completions (`compact.lua:593` usage), crash-safe store with startup recovery receipts (`crates/store/src/session_commit.rs` StartupRecoveryReceipt), turn-level retry phase (`core/src/working.rs:20-24`).

## Interop

MCP client only: rmcp-based, stdio+HTTP transports, tools gated through the same permission engine via `ToolOrigin::Mcp` (`crates/core/src/mcp/mod.rs:22-45`, `dispatcher.rs:2,12-13`); no MCP server. No ACP, no SDK, no IDE surfaces. Headless contract: `--headless` JSONL event stream, documented (`docs/docs/advanced/headless.md`) and subprocess-tested. Unusual provider interop: ChatGPT Codex, GitHub Copilot, Kimi Code subscription auth flows (`src/main.rs:122`, `provider/src/{codex,copilot,kimi_code}.rs`). LSP integration as bundled plugin tools (`crates/core/src/lsp/mod.rs` 2,880 LOC).

## Operability

Session store is a content-addressed SQLite lineage DB with `session doctor|backup|gc|vacuum` CLI (`src/main.rs:837-936`), JSONL export of history and request audit (`:492-495`), a local web request-inspector (`smelt inspect`, `:129`), `smelt status` for running processes, self-upgrade, rewind with rewindable-snapshot restore (`core/src/session.rs:829-935`), resume via session catalog (headless.md confirms `/resume` interactive). No session tree / fork-clone. Debug + perf panels as plugins.

## Anchor question

Closest anchor: pi -- both are solo-vision agents with a provider-agnostic loop separated from the product surface, policy deliberately delegated to an extension layer (smelt pushes it further: even compaction is Lua), headless-contract-first, and exemplary honesty about missing protections; smelt sits below pi because its TUI layer carries god files and it lacks pi's session-tree operability, cache warming, and extension breadth maturity, while beating pi on CI fuzzing and permission-policy depth.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | engine has zero deps on core/tui (`crates/engine/Cargo.toml:10-32`); loop `engine/src/agent.rs:1556`; docked per errata: `crates/tui/src/app/transcript.rs` 11,784, `content/transcript_buf.rs` 9,335, `layout_ir.rs` 6,189 |
| verification | 9 | fuzz regression replay in CI (`.github/workflows/ci.yml:269-294`), 17 oracled targets (`fuzz/README.md:3-27`), 80% coverage floor (`ci.yml:88`), wiremock subprocess suite (`tests/scenarios.rs:1-8`); no model evals caps below 10 |
| safety-enforcement | 7 | grammar-walked shell policy (`core/src/permissions/bash.rs:321-660`), binding in-loop decisions (`agent.rs:2142-2160`), 337 permission tests (`permissions/tests.rs`), fuzz oracle, trust gate (`trust.rs:45`); honest no-sandbox posture (`SECURITY.md:32`) keeps it under 8 -- nothing underneath approvals |
| token-economy | 8 | provider-anchored context estimate (`host_dispatch.rs:448-478`), group-boundary compaction + circuit breaker (`compact.lua:94,384`), tool-output budget (`trim.rs:11-44`), cache-stable sort (`cache.rs:28-32`) + fuzzed prefix invariance |
| orchestration | 5 | process supervision + goal/auto-continue with quota backoff (`auto_continue.lua:4-6`); no subagents, queues, or loop detection |
| interop | 7 | permissioned MCP client (`mcp/dispatcher.rs:2,12`), headless JSONL (documented + tested), LSP tools, tri-vendor subscription auth; no server/ACP/SDK/IDE |
| operability | 8 | doctor/backup/gc/vacuum (`main.rs:837-936`), inspect web UI (`main.rs:129`), rewind (`session.rs:829-935`); no tree/fork surfaces |
| originality | 8 | Neovim-grade Lua-scriptable agent core incl. custom modes (`runtime/lua/`, 90-page generated API), deterministic TUI-loop + cache-invariance fuzzing, plugin-owned compaction, redact-at-ingress |
| durability | 4 | bus factor 1, 53 stars/9 forks (remote-verified), 7 months young; offset by SECURITY.md, release+publish workflows, crates.io policy CI (`ci.yml:63-86`) |
| docs-dx | 8 | generated Lua API reference drift-gated in CI (`ci.yml:94-103`), 90 api pages, TESTING.md layer guide, no-auth-start wizard |

Weighted total: 12+13.5+7+8+5+7+8+8+2+4 = 74.5 -> band B.
Strongest: verification. Weakest: orchestration.

## Calibration notes

- Census corrections: test_loc 15,571 -> ~166k; non_test_loc 397k -> ~206k non-test Rust (census figure ≈ all-code incl. tests). Tier unchanged.
- No calibration rules triggered: original (not fork), active (remote-verified), not boundary (74.5; 3.5 clear of the 78 cut, would need +3.5 to reach A).
- Distribution check: does not join the 80+ crowd; strong B, above crush (69.0) on verification/token-economy/originality, below pi (78.5) on architecture/orchestration/operability/durability.
