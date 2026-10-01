# tura — T2 deep review

Rust workspace (`tura_workspace`, npm `tura-ai` v0.1.37), AGPL-3.0(-or-later), remote
https://github.com/Tura-AI/tura. Shallow clone, census contributors=1/commits=1 (squashed
shallow; do not read as history). Head fresh (2026-09-26); no archived claim needed.

## Anchor question

Closer to **crush**: both are strong-CI, honest-docs harnesses whose safety sits below the
6-rung; tura scores above crush on token economy, journaling/resume, and operability, and
below on safety (crush has a real per-tool-call permission service; tura's approvals are
vestigial).

## Census sanity

cloc total code 335.8k (Rust 166.6k, TS 54.0k, JSON 42.4k). Census non_test 206,254 +
test_loc 128,104 = 334,358 reconciles with the total, but the split leans on inline
`mod tests` attribution (Rust `tests/` dirs alone are 51.2k; `#[test]`/`#[tokio::test]`
attributes total 1,810; TS `*.test.*` 17.5k). Totals fine; the test/non-test boundary is
soft. No correction asserted.

## Architecture

Codex-style crate discipline with explicit contract crates: gateway / router / runtime /
provider / tools / lifecycle / session_log + `router_contract`, `runtime_contract`,
`session_log_contract` (Cargo.toml:11-42). Process topology is CLI/line-protocol, not HTTP:
gateway hosts router; router spawns `tura_runtime` subprocesses driven over a stdin/stdout
JSON envelope line protocol (crates/runtime/src/worker.rs:1-14); child agents dispatch via
`tura_router run-agent` subprocess, never URL (crates/runtime/src/manas/child_dispatch.rs:1-5).
Session state is a lifecycle state machine (`SessionState`/`RuntimeState` transitions,
process.rs:97, 174-186) with durable projections (checkpoint per turn, process.rs:193-197).
Largest files are ~1.9k (context/build.rs 1932, command_run_streaming.rs 1800); no >5k god
file; per the pi errata the 8→9 gap is loop/state separated from every product surface, and
tura holds that; but `process.rs:120-750` fuses the turn loop with retry policy, media
fallback, and compaction triggers in one 1.7k-LOC file, and `mano`/`manas` naming is
cryptic. **8**.

## Core loop (line-level)

`process_manas_internal` loop at crates/runtime/src/manas/process.rs:120. Per turn:
turn cap `manas_max_turns()` (:121-137, env-overridable, default from runtime_contract),
runtime-prompt manual injection (:142), `execute_turn` (:144-158), provider failure ladder
with bounded retries and media-capability fallback that strips unsupported content types
and tail-injects a developer retry prompt (:200-260, provider_step/prompt_retry modules),
tool execution via `execute_tool_calls` (tool_flow/execute.rs:29). A no-tool-retry counter
(no_tool_policy.rs:1-6, default 20) and a startup gate (see safety) bind model behavior.
The headline mechanism: instead of many small tools, one `command_run` macro tool executes
a stepped command graph (command_type shell_command/apply_patch/task_status/planning/...);
steps within one step-id run concurrently for reads, mutating commands are barriers with
shared/exclusive locks (command_run/policy.toml), and later steps bind earlier step outputs
(command_run/output_binding.rs). Execution is *speculative against the provider stream*:
`command_run_streaming.rs` runs commands from streamed tool-call arguments before the stream
finishes, emits `streaming_partial` updates (provider_flow/streamed_command_run.rs:234),
supports live results (`emit_cli_live_command_run_started`, command_run_streaming.rs:15),
and can terminate a turn early with `early_finish_reason` when an apply_patch in the graph
fails (:927-939, test :1097). That is the mechanism behind the token claims (README's
benchmark numbers are marketing; the mechanism itself is real in code).

## Compaction / token economy (line-level)

Two tiers, both deterministic — no summarizer LLM turn:
1. Agent-structured handoff: `task_status` writes `compact_context` into a command_run
   turn; the loop extracts it (turn_loop/tool_step.rs:14-91), applies
   `compact_session_context_with_agent_message_and_capabilities` (process.rs:357-424),
   persists a `context_compaction` record containing rebuilt timeline + workspace snapshot
   + environment context + timestamp (context/compaction.rs:101-150), resets session
   capabilities to baseline, and re-appends only the active Runtime Prompt manuals
   (compaction.rs:137-145). Inherited compact summaries are capped at 2 (compaction.rs:18).
2. Projected auto-compact: `auto_compact_summary_after_new_context` (process.rs:1026-1047)
   estimates added tokens from sanitized tool-result bytes/4 and fires when projected
   input exceeds the active limit; tested (process.rs:1345-1375). Trigger style is pi-like
   (projection, not after-the-fact), estimate is coarser (bytes/4, char_budget.rs:4-10).

Cache discipline: per-tool-result `context_cache` with stable cache ids
(context/tool_results.rs:134,385), immutable provider-shaped `context_messages` per log
entry so history rebuilds append-only (context/build.rs:440-490), stable per-session
`prompt_cache_key` gated by provider support with four property tests
(provider_flow/request_options.rs:163-178, 525-596), Anthropic `cache_control: ephemeral`
breakpoints mirroring the claude shape (provider/src/llm/claude_code.rs:410-460), and
cross-vendor usage extraction incl. cache read/write and reasoning tokens
(provider/src/metrics.rs:11-21). Tool outputs truncated middle-out at 10k chars
(char_budget.rs:3-4,36-69). Weakest point vs codex-9: no cache warming, no separate
no-LLM token-budget fresh-window tier distinct from the compaction tiers themselves,
bytes/4 estimate. **8**.

## Safety enforcement (line-level)

What binds:
- Command interceptor, ON by default, wired before every shell spawn
  (shell_executor/mod.rs:51,133 → command_safety.rs:74-101): denylist with de-obfuscation
  (wrapper stripping :30-34, nested `bash -c`/eval recursion, connector chains, base64-to-shell,
  shell-variable alias following), workspace-aware deletion checks
  (`is_dangerous_command_with_workspace` :89). Tests are property-style: 17 unit fns
  (command_safety.rs mod tests :1297+) plus e2e that *executes* commands and verifies the
  destructive side effect did not happen (crates/tools/tests/business/command_interceptor_e2e.rs:1-12).
- Kill switch: `TURA_COMMAND_INTERCEPTOR_DISABLED=1` disables it entirely
  (command_safety.rs:26-28) — trivially disarmable guardrail, and the model's commands run
  as the user.
- Workspace-bound sandbox is OPT-IN (`--sandbox` CLI, TURA_COMMAND_RUN_SANDBOX): blocks
  workdir outside workspace (command_run/handler.rs:793) and apply_patch outside workspace
  (commands/apply_patch/mod.rs:307). Off by default; docs are honest about this
  (docs/core/command-run.md:349-359) and the CLI even documents that the codex-compat
  `--dangerously-bypass-approvals-and-sandbox` flag "does not enable sandboxing"
  (gateway/src/tura_exec/cli.rs:51,168).
- Startup gate (its own policy idea, enforced in the loop): while `task_type` is unset,
  write-producing apply_patch steps are discarded from command_run arguments before dispatch
  (tool_flow/execute.rs:37,55-102). A sequencing gate, not a privilege boundary.

What does NOT bind:
- Approvals are vestigial. The runtime cannot raise permission requests at all:
  `permission_denial_for_tool` returns a blocked result for any gated tool
  (tool_flow/permission.rs:8-23), `request_command_run_sandbox_bypass` always Errs (:25-33);
  approval policy is read from env (TURA_APPROVAL_POLICY, :43-49) with default = no gating.
- No Rust-side emitter for `permission.asked`/`permission.replied` anywhere in the
  workspace (grep: zero hits in crates), yet the TUI renders `/approve <id> /deny <id>`
  prompts (apps/tui/src/tui/render.ts:133-138) and parses those events
  (apps/tui/src/gateway/events.ts:114-115). UI scaffolding with no backend = phantom control.
- The gateway agent API hardcodes `permission: PermissionRuleset { allow: ["*"], deny: [] }`
  (gateway/src/api/agent.rs:126-129); `disable_permission_restrictions` threads through
  session plumbing (contracts/session.rs:96, store.rs:887-941) with no enforcement site found.
- No OS-level sandbox anywhere (zero hits for seatbelt/bwrap/landlock/seccomp).

Net: real, tested denylist + opt-in path containment + a phantom approval UI. Below the
crush/pi 6-rung ("tested approval, nothing underneath") because the approval layer does not
exist in the backend; at/above nanocoder-5 in honesty and interceptor rigor, below it in
nothing having a real jail. **4**.

## Orchestration

command_run's step graph with read-shared/write-exclusive locks (policy.toml) is itself an
in-turn concurrency runtime, unique in corpus. Child subagents exist via router CLI
subprocess dispatch with depth cap — but the cap defaults to 0
(manas/tool_catalog.rs:82-88): subagents are off unless configured. Crash/recovery is the
strong plane: session-log protocol has runtime leases (`ActivateRuntimeLease`,
`RuntimeLeaseOutcome::LeaseConflict`, `StaleLease`) and replay
(`ReplayRuntimeRequest`, `RuntimeReplay`) (session_log_contract/src/protocol.rs:315,459-531);
router startup kills orphan runtime workers (router/src/daemon.rs:87-91) and runs
`recover_after_start` (:12); runtimes watch their router parent (worker.rs:27,70); idle
shutdown monitor (:111). No budgets beyond turn caps, no dedicated loop detector (only
no-tool retry cap), first-class queue absent. **7**.

## Interop

Strong headless contract: `tura exec --json` JSONL events, script-friendly stdout separation,
`--output-last-message`, codex-compatible flags (gateway/src/tura_exec/cli.rs:30-59). Broad
provider plane incl. rival-subscription transports: openai chat/responses, codex/chatgpt
OAuth, claude_code OAuth, google, bedrock, cohere, qwen, minimax, xai
(provider/src/llm/providers/). Three product surfaces (CLI/TUI/tauri-GUI + HTTP/WS gateway,
web/server.rs:300). But no MCP protocol client/server and no ACP — "capabilities" are a
homegrown command-package protocol (command.toml + schema.json + binary, e.g.
tool_catalog.rs:869-918, `--capability DIR`); no published SDK. pi occupies the 6-rung for
"excellent headless, no MCP/ACP by philosophy"; tura matches it. **6**.

## Operability

Durable SQLite session log owned by a single `tura_session_db` service, workspace-scoped,
with typed commands and parent links (ARCHITECTURE.md:11-30, matches code); resume via
`get_session`; git checkpoint commits per session with tested repo hygiene and nesting
exclusions (workspace_git.rs:11,125-239); session snapshot persistence 890 LOC; provider
call logs per request (ARCHITECTURE.md:48-56); process lock (gateway/src/process_lock.rs);
orphan reap + parent watchdog; CHANGELOG with 37 releases and per-release notes
(docs/changelog/); npm install with no postinstall lifecycle script (verified: package.json
scripts, only prepack/postpack). CLI compaction as an explicit session op and sessions
rename/refresh are documented (README + docs/core/context-management.md) and consistent
with the loop code. No user-visible rewind/tree navigation found. crush-8 tier. **8**.

## Originality

The corpus has nothing else like the `command_run` macro-graph executed speculatively
against the streaming tool-call arguments with early termination on patch failure — that is
an actual alternative architecture thesis to ReAct, and it is implemented, not just
marketed. `task_status` as a control plane (structured state gating apply_patch, manual
selection, and compaction handoff; docs/core/task-status.md + execute.rs:37) is likewise
distinct. Contract-crate process boundaries via line protocol are a distinctive, codex-
adjacent shape. Docked from 9: components reuse known ideas (denylist interceptor mirrors
codex/claude-code explicitly, command_safety.rs:4-8), and the benchmark superiority claim
is self-published with the ablation caveat conceded in README:155. **8**.

## Verification

~Half of Rust LOC is tests (tests dirs 51.2k + 1,810 test fns inline) and they prove
properties: mock-SSE-provider e2e driving the full loop asserting final session state,
tool results, and absence of stray tool calls (tests/business/coding_agent_mock_e2e.rs:24-80,
helpers/coding_agent_mock.rs:185+ write_codex_sse); interceptor e2e verifying destructive
side effects did not occur under a Docker harness (command_interceptor_e2e.rs:1-12); a
behavior-equivalence gate with baseline + failure-injection evidence artifacts run in CI
(.github/workflows/ci.yml:228-244, tests/equivalence/runtime_session/); per-crate clippy+test
matrix (ci.yml:95-169); backend full-chain E2E (:248-298); TUI/GUI test+e2e jobs (:305-439);
install-script tests on Windows/Linux/macOS matrix (:444-502); quality gate with
cargo-audit, cargo-deny, typos (:59-72); OS-worker matrix incl. windows/macos self-hosted
(os-worker-tests.yml:1-35); session-log contract tests incl. lease conflict/replay.
No in-CI model evals (benchmarks are an external repo), no fuzzing anywhere. Per the
errata, that caps at 8. **8**.

## Docs / DX

~5.5k LOC of in-repo docs that match code (docs/core/command-run.md sandbox section matches
the env flags; docs/core/task-status.md matches execute.rs gating), SKILL.md task-router for
agents, bilingual README, KNOWN_ISSUES.md listing benchmark evidence gaps against the
README's own claims, SECURITY.md with intake policy, tested installers, honest CLI help text
(noting the codex-compat bypass flag is a no-op). README marketing claims kept separate from
mechanism docs. **8**.

## Durability

Single visible contributor (shallow; undercount possible), no institutional backing
observed, but 37 releases, npm + GitHub Packages dual publish with dedicated recovery
workflows (npm-github-package-recovery.yml), 12 workflows, CODEOWNERS, SECURITY.md.
pi-7 minus community evidence; 6.

## Scores

| dimension | score | weight | best evidence |
|---|---|---|---|
| architecture | 8 | 15 | contract crates Cargo.toml:11-42; worker line protocol worker.rs:1-14; loop process.rs:120 |
| verification | 8 | 15 | equivalence gate in CI ci.yml:228-244; mock-SSE loop e2e coding_agent_mock_e2e.rs:24-80 |
| safety-enforcement | 4 | 10 | interceptor on-by-default shell_executor/mod.rs:51; kill switch command_safety.rs:26; phantom approvals permission.rs:8-23 + render.ts:136 vs zero Rust emitters; allow["*"] api/agent.rs:126 |
| token-economy | 8 | 10 | two-tier deterministic compaction process.rs:357-458; compact reconstruction compaction.rs:101-150; stable cache key request_options.rs:163-178 |
| orchestration | 7 | 10 | step locks command_run/policy.toml; lease/replay protocol.rs:315-531; orphan reap daemon.rs:87; child depth default 0 tool_catalog.rs:82 |
| interop | 6 | 10 | exec --json cli.rs:34; provider matrix llm/providers/; no MCP/ACP |
| operability | 8 | 10 | sqlite session-log service ARCHITECTURE.md:11-30 + lease API; git checkpoints workspace_git.rs:11 |
| originality | 8 | 10 | macro command graph handler.rs + command_run_streaming.rs:935; task_status gate execute.rs:37 |
| durability | 6 | 5 | 37 releases, recovery workflows; single contributor |
| docs-dx | 8 | 5 | docs/core matches code; README:155 ablation honesty |

**Weighted total: 72.0 → band B.** Strongest: token-economy (with architecture/verification
tied at 8; token-economy is the more distinctive 8). Weakest: safety-enforcement (4).

## Provenance / calibration

Original (census flag `original`; manifest tura-ai matches remote Tura-AI/tura). No fork
relationship detected with other corpus subjects. AGPL-3.0: findings are concepts-only, all
`effort_for_us` assume clean-room. Verification capped at 8 per errata (no evals-in-CI, no
fuzzing). No calibration demotions. Not near any band boundary (72 vs 65/78).
