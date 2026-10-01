# Anchor review: codex (openai/codex, Rust) — S

Tier read: T2/T3-scale (census 1.58M non-test / 455k test LOC; snapshot is 127-crate `codex-rs` workspace + TS/Python SDKs). Anchor for the S band. Shallow clone (1 commit): activity evidence is the 30+ CI workflows in-tree, not git dates.

**Closest anchor: n/a — is anchor.**

## Dimensions

### architecture — 9
- 127 single-purpose crates (`codex-rs/` dir listing); protocol (`app-server-protocol`), transport, core, tools, sandboxing cleanly separated.
- Turn loop at `core/src/session/turn.rs:163` (`run_turn`) with explicit state objects `StepContext`/`TurnContext` (imports `core/src/session/turn.rs:37-39`) and task model in `core/src/tasks/mod.rs` (1014 LOC, `regular.rs`/`review.rs`/`user_shell.rs` variants).
- Deduction: residual big files — `session/turn.rs` 3167 LOC, `client.rs` 2937 LOC; protocol surface has heavy generated-code bloat inflating the tree.

### verification — 8
- 439,900 LOC in `*_tests.rs` unit-test files (verified by find+cat count); 204 integration test modules in `core/tests/suite/`.
- Properties, not existence: `core/tests/suite/exec_policy.rs:31-37` drives full approval flows against a scripted mock SSE server (`start_mock_server`, `mount_sse_once`, `ev_function_call`) — asserts on protocol events, not snapshots of existence.
- CI: `.github/workflows/rust-ci-full.yml` runs nextest on multiple platforms (5 nextest refs); `cargo-deny.yml` for supply chain.
- Deduction from 10: no model evals and no fuzz targets anywhere in-tree (`find -type d -iname eval/fuzz` empty).

### safety-enforcement — 10
- OS-native enforcement on all three platforms: macOS seatbelt generated policies + `.sbpl` files incl. separate network policy (`sandboxing/src/seatbelt_network_policy.sbpl`, `seatbelt.rs:115-128` Restricted allowlists); Linux bundled bubblewrap + landlock + PID namespaces (`linux-sandbox/src/bundled_bwrap.rs`, `sandboxing/src/landlock.rs`, `linux_pid_namespace.rs`); Windows dedicated `windows-sandbox-rs` + read grants (`core/src/windows_sandbox_read_grants.rs`).
- Policy DSL for commands: prefix + network rules (`execpolicy/src/policy.rs:28`, `execpolicy/src/rule.rs:111,149` `PrefixRule`, `NetworkRule`) with parse/decision tests.
- Enforcement is in the tool path, not advisory: `core/src/tools/orchestrator.rs` + `core/src/tools/sandboxing.rs` (with `sandboxing_tests.rs`), `core/src/tools/network_approval.rs` for egress prompts, denial classification `sandboxing/src/denial.rs` and violation tests `violation_tests.rs`.
- Guardian reviewer pool reviews risky agent startup/context (`core/src/guardian_review.rs:1-5`).

### token-economy — 9
- Tiered compaction: model summarization (`core/src/compact.rs:56` SUMMARIZATION_PROMPT), remote compaction v2 with image budget tests (`compact_remote_v2_images.rs`, `compact_remote_v2_image_budget_tests.rs`), and token-budget compaction that skips summarization and installs a fresh context window (`compact_token_budget.rs:19-40`).
- Auto-compact wired to token status inside the loop (`session/turn.rs:555-612` `context_window_token_status`, three trigger sites :344/:612/:761) plus pre/post-compact hooks (`compact.rs:11-13`).
- Prompt-cache discipline: reuse gated on `previous_response_id` + `prompt_cache_key` equality (`core/src/client.rs:354-390`).
- Cost visibility: token usage surfaced in TUI (`tui/src/app.rs`, `tui/src/analytics/activity_chart.rs`). Deduction: no price/cost ledger in core.

### orchestration — 9
- Subagents: `core/src/codex_delegate.rs`, `core/src/tools/multi_agent_tool.rs`, plus dedicated `agent-graph-store`, `agent-roles`, `agent-message-board-client` crates.
- `codex queue` is a first-class command (`cli/src/main.rs:1885`); fork/resume/archive/delete (`fork_thread.rs`, `compact_resume_fork.rs`); daemon crash recovery (`core/src/session/daemon_recovery.rs`).
- Deduction: per-subagent token budgets not evident.

### interop — 9
- MCP on both sides: client (`codex-rs/rmcp-client`) and server (`codex-rs/codex-mcp`), MCP trace propagation test (`core/tests/mcp_trace_propagation.rs`).
- Machine contract: versioned `app-server-protocol` + `app-server-transport` + `app-server-daemon` — IDE surfaces (VS Code) drive this, not the TUI.
- SDKs: `sdk/python`, `sdk/typescript`, released via `.github/workflows/python-sdk-release.yml`, `sdk.yml`. Headless: `codex exec --json` (`cli/src/main.rs:3109` test asserting the flag).
- Deduction: no ACP.

### operability — 9
- Session persistence as parseable rollout journal (`rollout/src/lib.rs:52-109` decode/parse/materialize), resume picker + `--last` (`cli/src/main.rs:364,1446,2556`), `codex debug prompt-input` diagnostic (`cli/src/main.rs:1885`).
- Profiles, config constraints tested via `Constrained` config type (`core/tests/suite/exec_policy.rs:7`).
- Deduction: rewind-to-checkpoint of file state not present (diff tracker `turn_diff_tracker.rs` is per-turn, not revert).

### originality — 9
- execpolicy command-prefix/network DSL as a standalone crate + docs (`execpolicy/`, `docs/execpolicy.md`).
- Guardian reviewer pool (`guardian_review.rs`), code-mode: tools executed as code in a v8 session (`code-mode/src/grpc_session`, `.github/workflows/rusty-v8-release.yml`), `WorldState` context adapter in compaction (`compact.rs:9`).
- Verified in code, not marketing; but individually many ideas exist in embryonic form elsewhere (subagents, hooks).

### durability — 9
- Institutional backing: CLA workflow (`cla.yml`), `cargo-deny`, open-source fund doc (`docs/open-source-fund.md`), Apache-2.0. 30+ workflows = real maintenance surface.
- Shallow clone caps contributor evidence at 1 — measurement artifact, not bus factor.

### docs-dx — 7
- In-repo reference docs are stubs: entire `docs/*.md` is 206 lines; `docs/sandbox.md:1-3` is just a pointer to developers.openai.com. `docs/config.md`/`getting-started.md` are empty placeholders (wc: 0 total).
- Install/CLI help is dense and accurate (`cli/src/main.rs:1885` flag scoping errors are precise) but the reference lives off-repo.

## Verdict
**Strongest: safety-enforcement (10).** Weakest: docs-dx (7).
Weighted total 88.5 → **S** (safety 10, verification 8 both ≥5, so the gate passes). Borderline: if verification were read as 7 for the absent evals/fuzzing, total = 87.0 = A. Recorded deliberately: S here rests on OS-level sandbox + 440k LOC of property tests, not on vibes.
