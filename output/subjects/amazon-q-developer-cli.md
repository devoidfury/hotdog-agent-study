# amazon-q-developer-cli — T2 deep review

Tier: T2. Lane: coding-agent CLI (Rust, `q`). Version 1.19.7 (Cargo.toml:9).
License: MIT OR Apache-2.0 manifest (Cargo.toml:10) + Apache-2.0 file — permissive, no porting restrictions.

## Anchor question

Closest anchor: **crush** — same funded-org terminal agent at the "tested approval, nothing underneath" safety rung, single-strategy compaction, sqlite session state and mid-band orchestration; crush's 69.0 sits ~4 pts above my 65.0, which is the right ordering (crush has loop-detection, LSP, and a cleaner single tree; Q has a stronger checkpoint/hook layer and a cleaner second-stack loop).

## Census sanity (corrections)

- Shallow clone (`.git/shallow`, 1 commit, HEAD 2026-04-23). **Remote-verified: `pushed_at 2026-08-24`, `archived: false`** (github.com/aws/amazon-q-developer-cli API). Not dead; rule (b) does not apply. `contributors: 1, commits: 1` are shallow artifacts (440 forks / 1337 open issues on remote).
- `test_loc: 1015` wrong: Rust tests are inline — 129 files with `#[cfg(test)]`, 512 test fns, ≈14,947 inline test LOC + 1,259 LOC in `crates/*/tests/`. Real test LOC ≈ 16.2k (same glob-miss family as the crush/cline anchor artifacts).
- `non_test_loc: 197695` conflates generated code: the five `amzn-*` smithy-style SDK clients are ≈203k of the 280.3k total `.rs` LOC. Hand-written core ≈ 77k (agent 13.1k + chat-cli 53.3k + chat-cli-ui 1.3k + semantic-search-client 9.1k). By LOC alone this is a T1-shaped hand-written subject; T2 executed as dispatched.
- Provenance: original (aws org). GitHub API shows `template_repository: amazon-archives/__template_Custom` — repo-creation boilerplate only, no code delta; no fork.

## Product shape (matters for every dimension)

Two parallel agent stacks in one repo:
1. **Legacy shipped product** `crates/chat-cli` (53k LOC): the `q chat` TUI, its own conversation loop, tools, hooks shim, checkpoints.
2. **New `agent` crate** (13k LOC): a clean agent-framework library with its own protocol, loop, task executor, MCP manager, and a standalone runner (`agent/src/cli/run.rs`). Not yet consumed by the chat TUI (only `chat-cli/src/cli/agent/legacy/` wrappers). This is a mid-migration dual-tree.

## Core loop (mandatory read)

`crates/agent/src/agent/agent_loop/mod.rs` (732 LOC) — a genuinely clean state machine: `LoopState::{Idle, SendingRequest, ConsumingResponse, PendingToolUseResults, UserTurnEnded, Errored}` (:89-108); tokio `select!` over request channel and response stream (:174-260); `StreamParseState` incrementally assembles tool-use JSON deltas and errors invalid tool JSON back to the model (:523-537); explicit `Cancel` request that drains the stream and produces `UserTurnMetadata` with cycles/duration/end-reason (:288-312, :333-380). The loop itself does not execute tools or gate approvals — that is the Agent session in `agent/src/agent/mod.rs` with a documented 5-step tool pipeline: parse → permissions → PreToolUse hooks → approvals → execute (`mod.rs:985-992`, `handle_tool_uses` :992+). Denials and hook blocks return tool-error content to the model, not just to the UI (:1037-1056). Docking: the loop's own unit test is commented out at `agent_loop/mod.rs:704-731`.

Legacy loop: `chat-cli/src/cli/chat/mod.rs` is a **4,699-LOC god file** fusing the ChatState machine, rendering, compaction driver, error UX and its own test module (:4032+).

## Compaction / token economy (mandatory read)

- Legacy: counting is **character-based** (`token_counter.rs:13-50` CharCount/TokenCount wrappers; `conversation.rs:1018-1052` size model), single Critical warning threshold at 600K chars (`conversation.rs:1075-1076`) which nags the user to run `/compact` (`chat/mod.rs:3652-3673`). Auto-compaction is **reactive**: only when the backend returns `ContextWindowOverflow` does the client enter `ChatState::CompactHistory` (:991-1010), with a settings opt-out (`ChatDisableAutoCompaction`) and a hard-failure recovery menu when even compaction overflows (:969-986). LLM-summarization prompt with custom-prompt injection and prior-summary fold (`agent/src/agent/compact.rs:130-193`). No token projection, **no prompt-cache discipline anywhere** (zero `cache_control` hits in chat-cli/agent src).
- New agent crate: `CompactStrategy` = equal-share truncation of text/JSON blocks over `max_message_length` (default 25,000) (`compact.rs:20-124`), triggered on overflow (`CompactingState.last_user_message` :24-30). Tested (`compact.rs:296-329`). Below every anchor's projected-context rung; crush's "one standard auto-summarize" at 6 is generous compared to char-threshold + error-triggered.

## Permissions / safety (mandatory read)

- Legacy bash gate (`chat-cli/src/cli/chat/tools/execute/mod.rs:51-148`): default-require-approval, with a decent conservative heuristic layer — shlex split; multiline always asks; `DANGEROUS_PATTERNS` scan (`<(`, `$(`, backtick, `IFS` etc., :62); per-pipe-segment check against `READONLY_COMMANDS` (:40-42, :133-140); special-cased `find -exec/-delete/-fprint` (:97-109) and `grep -P` RCE (:112-119); user-configured **anchored regex allowlist** (:124-131). Behaviorally tested: `test_requires_acceptance_*` (:303, :377, :415), `test_eval_perm_*` incl. denied-command regex and deny-by-default (:462-701), CloudTrail tests (:703, :730).
- **Nothing underneath**: zero `sandbox|bubblewrap|seatbelt` references in any hand-written crate. Enforcement is approval + config only.
- New framework hole: `evaluate_tool_permission` returns `Allow` **unconditionally** for `ExecuteCmd`, `Mkdir`, `Grep`, `Introspect`, `SpawnSubagent` (`agent/src/agent/permissions.rs:62-69`); `allowed_tools` gates tool *exposure* (mod.rs:1498), not per-call execution; no `requires_acceptance` exists in `agent/src/agent/tools/execute_cmd.rs`. The framework has the approval channel (`AgentRequest::SendApprovalResult`, mod.rs:598-723) but nothing routes shell through it.
- Real bug: `Ls` and `ImageRead` are evaluated against **`fs_write`** allowed/denied paths while the comment says "Reuse the same settings for fs read" (`permissions.rs:47-61`) — read tools gated by write policy (fail-direction ambiguous, at minimum misleading config semantics).
- Hooks: Claude-Code-style semantics — JSON event on stdin, exit 0 pass / exit 2 blocks PreToolUse with STDERR returned to the LLM (`docs/hooks.md:27-29`), glob matchers incl. `@mcpserver/tool` forms (`docs/hooks.md:33-42`); hook config with timeout, 10KB output cap, and a **cache TTL** (default off) for hook outputs (`chat-cli/src/cli/agent/hook.rs:15-76`); triggers `AgentSpawn/UserPromptSubmit/PreToolUse/PostToolUse/Stop`. No trust-scoping of where hook definitions come from (project files define executable hooks).

## Tests, CI, orchestration, interop

- Tests ≈512 fns; the standout is faux-provider harnesses in both stacks: `crates/agent/tests/mod.rs` drives whole sessions from scripted JSONL response streams with **scripted approval decisions** (`builtin_tools.jsonl`, `context_window_overflow.jsonl`) and asserts config-file assembly into prompts; chat-cli drives the full loop via `os.client.set_mock_output(...)` (mod.rs:4032+) including PreToolUse/PostToolUse hook-output assertions (:4550-4567). Real property-style assertions, not existence tests.
- CI (`.github/workflows/rust.yml`): clippy `-D warnings` ubuntu+macOS (:15-46), nightly `cargo-llvm-cov` test job (:48-77), cargo-deny (:130-135); Windows jobs commented out (:79-). Terminal-bench eval harness exists but is **workflow_dispatch-only** (`.github/workflows/terminal-bench.yaml:7-8`). Anti-pattern: the CI test command `cargo test --workspace ... --exclude fig_desktop-fuzz` excludes a crate absent from workspace members (`Cargo.toml:8-9`) — static mismatch suggests the gate may not run as written (med confidence; cannot execute to confirm).
- Orchestration: `delegate` tool spawns **background sub-agent processes** (a `q chat` per agent, one task per agent, JSON state files under workspace `subagents/` dir), monitored by a coroutine (`chat-cli/src/cli/chat/tools/delegate.rs:44-58, 337, 377, 418-463, 589`). TaskExecutor runs tools+hooks concurrently off the session task (`agent/src/agent/task_executor/mod.rs:40-56`). No budgets, no queue, no loop detection, no crash-resume for delegated subagents. Experiment flags gate Knowledge/Thinking/TangentMode/TodoList/Checkpoint/Delegate (`chat-cli/src/cli/experiment/experiment_manager.rs:44-114`).
- Interop: MCP client via rmcp 0.8 with child-process/SSE/streamable-http transports and **full OAuth for MCP servers** (740-LOC `chat-cli/src/mcp_client/oauth_util.rs`; `Cargo.toml:137` incl. `auth` feature); dynamic tool manager (2,205 LOC). Headless `--no-interactive` + `OutputFormat` (`chat-cli/src/cli/mod.rs:68, 574`). No MCP server mode, no ACP, no published SDK/IDE plugin in-tree.
- Operability: sqlite session persistence + `--resume` (`chat-cli/src/cli/mod.rs:469-503` tests), **shadow bare-git checkpoints** with per-turn tags, restore/diff/stats/cleanup (`chat-cli/src/cli/chat/checkpoint.rs:34-36, 149, 207, 259-334`, requires system git :384); tangent mode = checkpointed side-thread with restore (`docs/tangent-mode.md`); `/usage` context analysis, `q diagnostic`, logdump, settings + changelog + issue-creation commands (`cli/mod.rs:97-118`).
- Originality mechanisms verified in code: use_aws tool with CloudTrail attribution env plumbing (`tools/use_aws.rs` 508 LOC + tests execute/mod.rs:703-757); semantic-search knowledge store with text embeddings (`semantic-search-client`, 9.1k LOC, `src/lib.rs:26-27`), todo lists as resumable artifacts folded through compaction summaries (`compact.rs:146-147` requires the loaded todo-list ID); tangent mode; in-CLI experiment-manager feature gating.

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | clean new-agent loop (`agent/src/agent/agent_loop/mod.rs:89-108,174`) vs 4,699-LOC `chat-cli/src/cli/chat/mod.rs` god file + dual parallel stacks (`Cargo.toml:8-9`, `chat-cli/src/cli/agent/legacy/`) |
| verification | 7 | faux-provider harnesses `crates/agent/tests/mod.rs:12-60` + `chat/mod.rs:4032+,4550-4567`; dual-OS clippy/test/cov + cargo-deny (rust.yml:15,48,130); docked: loop's own test commented out (agent_loop/mod.rs:704-731), CI excludes missing crate (`Cargo.toml:8`), evals manual-only |
| safety-enforcement | 6 | tested default-deny bash gate (execute/mod.rs:51-148 + tests :303-701); exit-2 blocking hooks (docs/hooks.md:29); nothing underneath: no sandbox anywhere; new-framework ExecuteCmd unconditional Allow (permissions.rs:67) |
| token-economy | 5 | char-based 600K threshold (conversation.rs:1075) + reactive-on-overflow auto-compact (chat/mod.rs:991-1010) + equal-share truncation (compact.rs:52-124); no projection, no cache discipline |
| orchestration | 6 | delegate background subagent processes (delegate.rs:337-463), TaskExecutor concurrency (task_executor/mod.rs:40-56), sqlite resume; no budgets/queues/loop-detection/crash-resume |
| interop | 7 | MCP client w/ OAuth + 3 transports (oauth_util.rs 740 LOC; Cargo.toml:137), headless json output (cli/mod.rs:68,574); no MCP server/ACP/SDK |
| operability | 7 | shadow-git checkpoints restore/diff/cleanup (checkpoint.rs:149-334), --resume sessions, tangent, /usage, diagnostics, logdump; crash-recovery posture not visible |
| originality | 7 | CloudTrail-tracked cloud tool (execute/mod.rs:703-757), semantic knowledge store (semantic-search-client/src/lib.rs:26-27), tangent mode, todo-through-summary (compact.rs:146), experiment gating (experiment_manager.rs:44-114) |
| durability | 8 | AWS institution, remote-verified active (pushed 2026-08-24, not archived), 1.19.x cadence, cargo-deny, SECURITY.md; public CI/release surface thinner than codex/cline (5 workflows) |
| docs-dx | 7 | mdbook docs 2.1k lines incl. hooks/agent-format/tangent (docs/), CONTRIBUTING, `q diagnostic`, issue command; deep product docs external to repo |

Weighted total: (6·15 + 7·15 + 6·10 + 5·10 + 6·10 + 7·10 + 7·10 + 7·10 + 8·5 + 7·5)/10 = **65.0 → band B**, exactly on the B floor (boundary-risk note to dispatcher).

Strongest dimension: **durability (8)**. Weakest dimension: **token-economy (5)**.

## Calibration notes

- No rule (b) demotion: shallow HEAD ≠ dead; remote active.
- Hand-written LOC ≈77k argues T1-sized; tier per census/dispatcher kept T2, noted for synthesis.
- If the `fig_desktop-fuzz` CI mismatch is confirmed as a broken gate, verification drops to 6.5 → total 64.25 (C boundary). Left at 7 pending CI-status evidence; boundary note covers the risk.
- Institutional durability weighed honestly at 8 (not 9: thinner public CI/release infra than the 9-rung anchors; not lower: remote-verified active, funded team).
