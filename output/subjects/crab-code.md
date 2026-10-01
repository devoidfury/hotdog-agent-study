# crab-code — T2 deep review

Subject: /data/samples/agents/crab-code (Rust, 25 crates + xtask, MIT)
Census row: non_test_loc 129,372 / test_loc 2,905 / contributors 1 / commits 1 / shallow true / head 2026-09-22 / T2.

## Anchor question

**Which anchor is this closer to, and why?** Closer to crush (69.0, B) than to cline (76.5): a
single-author, strongly-tested core with disciplined CI where breadth (kernel sandbox, tiered
compaction, ACP+MCP, headless contract) exceeds crush's, placing crab in upper-B rather than
mid-B -- short of codex-shape depth because no test executes the loop against a scripted
provider and the safety backends are never proven by live confinement-violation tests.

## Census sanity-check (corrections)

- **test_loc wrong (again).** Census says 2,905; actual: 60,876 lines inside `#[cfg(test)]`
  modules across 427 files (script-counted) + ~3,530 in `crates/*/tests/` = **~64k test LOC**,
  undercounted ~22x. Census counted only the tests/ dirs. Same failure mode as crush/cline/
  nanocoder anchors: Rust inline `#[cfg(test)]` modules missed. True non-test LOC ~97k; total
  .rs ~161k (README's "161k / 4940 tests" matches my counts: 4,971 `#[test]`/`#[tokio::test]`).
- **commits: 1 is a shallow-clone artifact.** `.git/shallow` present. Remote-verified: repo
  created 2026-04-02, pushed 2026-09-29, **579 commits** (578 lingcoder + 1 dependabot),
  contributors = 1 human confirmed remotely. Active, not dead; do not apply the archived cap.
- non_test_loc 129,372 vs my ~97.6k (158,441 non-tests-dir .rs minus 60,876 inline test lines).
  Census LOC inflated on the non-test side by the same glob miss. Tier stays T2 on total LOC.

## Provenance

- Not a fork (GitHub `fork:false`, original MIT). Census "original" stands.
- **But "built from scratch" (README.md:19) is contradicted by the code itself**: systematic
  per-module mapping comments to Claude Code internals -- "Maps to Claude Code's
  `microCompact.ts`" (crates/session/src/micro_compact.rs:8), "`bashClassifier.ts` +
  `shellRuleMatching.ts`" (crates/tools/src/builtin/bash_classifier.rs:7), "Maps to Claude
  Code's `SkillTool`" (crates/tools/src/builtin/skill.rs:7), "Mirrors Claude Code's
  `applyGrouping` + `CollapsedReadSearchContent`" (crates/tui/src/history/grouping.rs:10) --
  plus a whole-table structural mapping to a "TS reference" with internal dir names
  (`services/compact/`, `memdir/`, `cli.tsx`) at docs/architecture.md:68-73. Those internal
  module/component names are only visible from source-level access to proprietary Claude Code
  (this corpus separately holds a leaked snapshot: free-code). Confidence med that this is a
  source-derived port rather than behavior observation + good reverse engineering; logged as a
  `license-risk` finding, and any portables from declared-mapped modules are concept-only.
- One *cleanly* attributed borrow: `crates/sandbox/src/backend/seatbelt_base_policy.sbpl:3-5`
  credits codex's Apache-2.0 `seatbelt_base_policy.sbpl` (itself from Chromium). Apache-2.0 +
  attribution = fine.

## Mandatory line-level reads

### Core loop -- crates/engine/src/loop.rs (2,016 LOC)

- `query_loop` at loop.rs:190; ~9 recovery counters consolidated in an explicit `LoopState`
  struct (loop.rs:46-59) with a documented rationale ("makes the points where each counter is
  reset explicit and auditable").
- Recovery machinery is the standout: PTL (prompt-too-long) -> image-block strip first, then
  forced compaction, capped at 3 retries (loop.rs:296-318); overloaded -> fallback-model replay
  with `StreamAborted` so the TUI discards partial output (loop.rs:321-356); max-tokens
  truncation -> escalate cap to 64k without touching the conversation, then verbatim-continuation
  injection, guarded so a truncated message *with* tool_use never gets an orphan (loop.rs:390-440).
- `build_tool_result_message` (loop.rs:600-655) enforces the provider invariant "one
  tool_result per tool_use" with backfill errors -- inline-streamed, batch, and malformed-JSON
  results merged by id.
- Stream hygiene: a stream ending without `MessageStop` is discarded as retryable and inline
  tools are `abort_all()`ed so doomed attempts can't execute tools or emit stale results
  (loop.rs:895-903, 748-753, 826-831).
- Plan-mode gating closes the same-batch bypass: `apply_plan_mode_transitions` (loop.rs:570-590)
  gates writes appearing *alongside* EnterPlanMode in the same assistant message.
- Nested AGENTS.md injected per-turn for files touched by the batch, deduped via
  `loaded_nested_paths` (loop.rs:520-553).
- Residual: one 2k-line loop file; counters are grouped but the loop body is still long. Fine.

### Compaction / context -- crates/session/

- `ContextManager` ordered thresholds warn 70 / upgrade 75 / compact 80 (context.rs:9-26);
  occupancy from `effective_token_estimate` (conversation.rs:160) which **anchors on the real
  `input_tokens` from the last API response** plus char-estimate only for messages appended
  since -- explicitly called out for CJK where chars/4 underestimates 3-4x. Post-compaction the
  stale anchor is dropped (compaction.rs:296-301). This is the correct design and rare.
- **Novel step before compaction**: at 75% try swapping to a larger-context variant of the same
  model (`try_upgrade_context`, api/lib.rs:135; loop.rs:991-1050 emits `ContextUpgraded`), only
  falling through to compaction when no upgrade exists.
- 7-strategy usage-percentage ladder (`CompactionStrategy::for_usage`, compaction.rs:208-216):
  Snip 70-80 (no LLM) -> Microcompact 80-85 (LLM per-result summaries >500 tok) -> Summarize
  85-90 -> Hybrid 90-95 (keep 3) -> emergency Truncate, plus additive SessionMemory and
  SlidingWindow modes. Cheap no-LLM tiers first (snip_compact.rs:1-15, micro_compact.rs:1-16).
- Auto-compact circuit breaker: 3 consecutive failures disable further auto-compaction for the
  session (auto_compact.rs:18, 84-86); output reserve 20k + buffer 13k computed against window
  (auto_compact.rs:12-15, 92-97).
- Compaction cut repair: `repair_tool_pairing` (compaction.rs:329-362) strips orphaned
  tool_use/tool_result pairs a mid-conversation cut can create -- with tests asserting validity.
- Cost visibility: per-model pricing, cache-token accounting, persisted summary
  (session/src/cost.rs:192-298). Oversized tool outputs spill to temp files with an in-band
  2k preview + Read pointer (tools/src/executor.rs:15-45).
- Prompt-cache discipline is the weak half: two static breakpoints (System, Tools)
  (loop.rs:236-241), stats tracking only (api/src/cache.rs:24-37). No message-level breakpoints,
  no cache warming (pi), no cache-key-gated compaction (codex).

### Permission / sandbox

- `check_permission` (tools/src/permission.rs:41) implements a 7-mode matrix (Default,
  AcceptEdits, TrustProject, DontAsk, Dangerously, Plan, Auto-classifier) with an explicit,
  commented **SECURITY BOUNDARY: deny evaluates before allow, any source, any mode**
  (permission.rs:76-84) and a dedicated cross-layer regression spec
  (crates/config/tests/permission_deny_wins_spec.rs:1-100: same-source, user-allow vs
  project-deny, etc.).
- Per-segment deny for chained bash: `echo ok && rm -rf x` is split (quote-aware) and each
  segment re-matched (permission.rs:91-115, split_shell_segments:277, tests :1157-1164).
- `DontAsk` is a final ask->deny transform pass so no early-return path leaks a prompt
  (permission.rs:52-64).
- Tool provenance is first-class: `ToolSource::{BuiltIn, McpExternal{server}, AgentSpawn}` with
  different rules per mode (core/src/tool.rs:23-33); MCP tools untrusted by default.
- Sandboxing is **transform-style**: callers hand `(program, args, cwd)` + policy and get back a
  confined `tokio::process::Command` (sandbox/src/lib.rs:1-16). Linux Landlock installed via
  `pre_exec` in the child only, ABI V5 BestEffort, **full-disk read / cwd-scoped write,
  fail-closed spawn error if the kernel can't enforce** (landlock.rs:57-67, 130-142). macOS
  seatbelt SBPL deny-by-default, hardcoded `/usr/bin/sandbox-exec` PATH-hijack defense, network
  denied unless policy allows, `.git`/`.crab` carved read-only out of writable roots
  (seatbelt.rs:1-10, 57-66, 96-120). Windows: honest fail-open with a warning, and denial-hint
  attribution suppressed there so failures aren't blamed on a sandbox that didn't run
  (lib.rs:6-8, bash.rs:186-210).
- Sandbox policy derives from permission mode: Dangerously = None, Plan = read-only, everything
  else = workspace_write (policy.rs:164-177); sandbox denial detection + model-facing hint
  (denial.rs:26). Denial-loop tracking: warn after 3 consecutive / 20 total denials
  (core/src/permission/denial_tracker.rs:12-16).
- **Gap vs the 8 rung**: no live confinement-violation tests -- sandbox tests assert policy
  derivation and shape (39 tests), nothing spawns a confined process and asserts the write was
  refused (contrast: nanocoder's CI installs bubblewrap to run its jail spec). No Linux network
  confinement (explicitly out of scope, landlock.rs:11-12); no separate network-approval flow
  (codex). Landlock default grants full-disk read.
- Team approvals coalesce: teammates route through the session handler and
  `PermissionSyncManager::resolve` makes concurrent identical asks one prompt, broadcast answer
  (coordinator/permission_sync.rs:1-11; teams/permission.rs:1-12). No auto-approve side door.

## Other surfaces

- **Tests (do they prove properties?)** Mostly yes: deny-wins cross-layer specs, config merge
  specs, compaction pairing-repair, upgrade-vs-compact behavior async tests (loop.rs tests),
  quote-aware segment splitting, remote-protocol E2E against a real WS server on a loopback port
  (remote/tests/e2e.rs:1-30), TUI cell-level snapshot baselines (tui/tests/snapshots.rs). But:
  `LlmBackend` is a **closed enum** (api/src/lib.rs:35-42) with no injection seam, so `query_loop`
  is never exercised against a scripted provider; the M7b "E2E" builds a client pointing at
  `http://localhost:0` (integration_m7b.rs:37-41) and can't test loop semantics end-to-end.
  No fuzzing, no proptest, no evals (dependency grep clean). CI: fmt+clippy -D warnings, cargo
  check of two feature extremes, nextest on 3 OSes, cargo-deny (ci.yml:22-79). Verification sits
  a half-rung below the 8 ceiling: big, mostly property-shaped corps, zero loop-level e2e.
- **Orchestration**: in-process leader/worker teams (teams/runner.rs 1,067 LOC), mailbox bus
  with monotonic correlation ids (team/src/bus.rs:10-23), shared TaskList tools, cron jobs with
  fired-prompt injection queue (tools/src/builtin/cron.rs:16-45), git worktree + tmux launch
  flags (cli/src/args.rs:206-212), team permission sync. No durable team journal / crash-resume,
  no budgets beyond `max_budget_usd` config field, queue not first-class. 7 rung (crush/pi).
- **Interop**: MCP client (incl. OAuth flow) **and** server mode (`crab mcp serve` path via
  ServeArgs); ACP server on Zed's official SDK (acp/src/lib.rs:1-13, cli/acp_mode.rs); headless
  `text|json|stream-json` + `--json-schema` (args.rs:58-71); **imports the rival harness's
  ecosystem wholesale**: Claude Code plugins (plugin/src/lib.rs:1, cc_installed.rs), `.claude/
  agents/` (commands/agents.rs:45), `~/.claude/CLAUDE.md` memory (memory/src/agents_md.rs:74),
  hooks protocol, `.claude/settings.json` mcpServers format (mcp/src/discovery.rs:8); IDE plugin
  ambient context via MCP client (ide/src/lib.rs:1-28); own remote-control WS protocol with
  JsonSchema codegen for TS/Swift/Kotlin (remote/src/lib.rs:1-21) + e2e tests. Matches cline's 8
  rung on breadth; no published SDK, so not 9.
- **Skills**: registry + discovery, `Skill` tool for model-invoked lazy loading
  (tools/src/builtin/skill.rs), MCP prompts auto-converted to namespaced skills
  (`mcp__<server>__<prompt>`) at connect (agents/src/mcp_skills.rs:1-40) -- a genuinely nice
  interop bridge.
- **Memory**: AGENTS.md/CLAUDE.md layered loading + nested per-file injection in the loop;
  periodic memory extraction proposals from transcripts (session/src/memory_extract.rs:1-13);
  memory store with ranker/relevance/age/security modules; `autoDream` background
  memory-consolidation behind a default-off feature with three cheap gates and fd-lock
  (auto_dream.rs:1-20) -- the forked-agent runner is honestly marked deferred. Proactive
  suggestions are declared stubs (proactive/mod.rs:1-12). Vapor is labeled as vapor -- good
  honesty, but it doesn't earn originality credit until it runs.
- **Operability**: sessions persist messages incrementally incl. granted tools
  (session/src/history.rs:10-21), resume/continue/`--fork-session`, `/rewind` file history via
  path-hash versioned snapshots, LRU 100/session (file_history.rs:1-25), cost summary files,
  effort levels mapped to thinking budgets (engine/effort.rs:16-30), color-eyre + tracing-appender.
  No rollout-journal replay tooling or debug-dump command (codex `debug prompt-input`). 7.
- **Durability**: 1 human maintainer (bus factor 1), ~6 months old, 76 stars / 14 forks,
  active daily pushes, dependabot + cargo-deny + githooks. No SECURITY.md. nanocoder-like with
  better CI hygiene: 4.
- **Docs/DX**: docs/architecture.md is a 3,984-line current-state reference with per-crate
  designs, dependency rules matching actual imports (e.g. ide/src/lib.rs:26-28 restating the
  same-layer prohibition), config-design.md 511-line merge spec cited by tests
  (permission_deny_wins_spec.rs:5); bilingual README, honest feature-status notes. No SECURITY.md,
  no per-provider how-tos. 7.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | 25-crate strict 4-layer contract (docs/architecture.md:14-21, :337-341) enforced in code (ide/src/lib.rs:26-28); loop separated from every product surface in its own engine crate (engine/src/loop.rs:190); LoopState consolidation (loop.rs:41-59); max file 3,086 (tui/instance.rs), no god files; docked from 9: three parallel runtime layers (agents/runtime.rs 1,562 + agents/session/runtime.rs 1,028 + teams/runner.rs 1,067) |
| verification | 7 | ~4,971 tests / ~64k LOC; security-invariant specs (config/tests/permission_deny_wins_spec.rs); remote WS e2e (remote/tests/e2e.rs); TUI snapshots (tui/tests/snapshots.rs); 3-OS nextest + deny CI (ci.yml:52-79); docked: closed LlmBackend enum (api/src/lib.rs:35-42) means the loop is never tested against a scripted provider; no fuzz/evals |
| safety-enforcement | 7 | real kernel enforcement both unix platforms, fail-closed (landlock.rs:137-141); deny-wins + segment-deny + DontAsk leak-proof, all tested (tools/src/permission.rs:52-115); honest Windows fail-open; docked: no live confinement-violation tests, no Linux network confinement (landlock.rs:11-12), full-disk read default |
| token-economy | 8 | real-usage-anchored occupancy (conversation.rs:153-163); upgrade-before-compact ladder (loop.rs:991-1050); 7-tier strategy ladder (compaction.rs:208-216); circuit breaker (auto_compact.rs:84-86); output spill files (executor.rs:30-45); docked: two static cache breakpoints only (loop.rs:236-241) |
| orchestration | 7 | teams + coalesced approvals (permission_sync.rs:1-11), cron fired-prompt queue (cron.rs:36-45), worktree/tmux (args.rs:206-212); no durable team journal, budgets unproven |
| interop | 8 | MCP client+server, ACP on official SDK (acp/src/lib.rs), stream-json headless (args.rs:58-71), wholesale Claude Code ecosystem import (plugin/cc_installed.rs, agents_md.rs:74, hooks executor.rs:3), WS protocol with schema codegen (remote/src/lib.rs:17-19); no published SDK |
| operability | 7 | resume/continue/fork, /rewind snapshots (file_history.rs:1-11), grants persisted across resume (history.rs:17-19), cost files; no journal replay/diagnostics command |
| originality | 7 | context-upgrade ladder, permission coalescing, MCP-prompt-to-skill bridge, autoDream design; but a large share of surface is declared Claude-Code parity (micro_compact.rs:8 etc.) |
| durability | 4 | solo author (remote: 578/579 commits), 6 months, 76 stars, no SECURITY.md; active + dependabot/deny hygiene above nanocoder |
| docs-dx | 7 | 4.5k lines of current-state docs matched by tests; honest stub labeling; bilingual README |

**Weighted total: 72.0 -- band B (upper). Strongest dimension: token-economy (with architecture
and interop at the same 8; token-economy named for the upgrade-ladder + real-token anchoring).
Weakest: durability (4).**

## Calibration notes

- Census corrections (above): test_loc ~64k not 2,905; commits remote-verified 579 (shallow
  artifact); contributors 1 confirmed; repo active (pushed 2026-09-29). No archived cap applies.
- Not a fork; rule (a) N/A. No demotion. Boundary risk: 72.0 is >2 pts from any band edge.
- license-risk finding (ported-proprietary-source, med confidence): portables from declared-mapped
  modules (micro-compact, bash classifier, skill tool, history grouping) are concept-only;
  crab-original mechanisms (upgrade ladder, permission sync, mcp-prompt bridge, spill files)
  carry no such restriction.
- Distinct from claw-code / claw-code-agent (both TS-origin); crab-code is its own codebase.
- Did not exhaustively read all 4,971 tests; sampled across 8 crates incl. the spec files and
  sandbox/permission/engine/tui suites. Snapshot baselines exist under crates/tui/tests/snaps.
