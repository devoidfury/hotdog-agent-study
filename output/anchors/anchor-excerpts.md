# Anchor excerpts (frozen reference - Phase 1, 2026-09-29)

Do not edit; this is what every later reviewer reads before scoring. Levels not observed here interpolate between the two nearest rungs. Full reports: `anchors/<subject>.md`.

**Band summary: codex=88.5 (S), pi=78.5 (A), cline=76.5 (B, band ceiling), crush=69.0 (B), nanocoder=60.5 (C), codel=23.0 (D).**

## architecture (15)
- **9 codex / 9 pi**: codex = 127 single-purpose crates, protocol/transport/core/tools/sandboxing separated, loop at `core/src/session/turn.rs:163` with explicit StepContext/TurnContext (only residual god files: turn.rs 3167, client.rs 2937). pi = three-layer split (898-LOC provider-agnostic loop `packages/agent/src/agent-loop.ts:37` / provider layer / product layer), everything else an extension.
- **8 cline**: SDK-first core consumed by thin hosts (`sdk/packages/agents/src/agent-runtime.ts` 2825 LOC serves vscode/cli/hub); docked for dual-tree migration (legacy `apps/vscode/src/core` + sdk) and 2.7k-LOC runtime host files.
- **7 crush**: client/server + sqlc-state design (`internal/db/`, `internal/server`) but `agent.go` 2393 + `coordinator.go` 1877 fuse loop, policy, and wiring.
- **5 nanocoder**: loop inside React hooks (`source/hooks/chat-handler/conversation/conversation-loop.tsx` 1414 LOC) forcing parallel re-implementations per entry path (`acp-conversation.ts`, `message-handler.ts`); services layer itself is fine.
- **3 codel**: no loop at all - `Provider.NextTask -> DB row -> executor` oracle (`backend/providers/providers.go:23-31`).

## verification (15)
- **8 codex / 8 cline / 8 pi**: codex = 440k LOC `*_tests.rs` + 204 suite modules driving full approval flows against scripted mock SSE servers (`core/tests/suite/exec_policy.rs:31-37`), multi-platform nextest CI. cline = 255k LOC `*.test.ts` asserting loop semantics (`agent-runtime.test.ts:2355` approval-iff-policy). pi = 625 files named like a property spec (`agent-session-tree-navigation.test.ts`, `bash-close-hang-windows.test.ts`), CI build+check+test (`ci.yml:36-42`). Common ceiling: no in-CI model evals, no fuzzing - that's what separates 8 from 10.
- **7 crush / 7 nanocoder**: crush = 61.7k LOC tests, `-race` 3-OS matrix (`build.yml:29-30`), govulncheck. nanocoder = 161k LOC ava specs with coverage-drop CI gate (`pr-checks.yml:17-21`) and a CI job that installs real bubblewrap to run the jail spec (`:63-95`) - tests assert behavior (`bash-sandbox.spec.ts:15-40`) but TUI/loop paths thin.
- **0 codel**: zero test files, only workflow publishes a Docker image.

## safety-enforcement (10)
- **10 codex**: kernel-level enforcement all platforms (`sandboxing/src/seatbelt*.sbpl`, `bundled_bwrap.rs`, `linux_pid_namespace.rs`, `windows-sandbox-rs`) with denial/violation tests, execpolicy DSL (`execpolicy/src/rule.rs:111,149`), separate network-egress approval (`seatbelt_network_policy.sbpl`, `tools/network_approval.rs`).
- **6 cline / 6 pi / 6 crush**: the "tested approval, nothing underneath" rung. cline: approval binds in-loop and is tested (`agent-runtime.ts:2384-2397` + `agent-runtime.test.ts:2355`), plan-mode guard before approval (`command-guard-extension.ts:11-13`). pi: no default protection, blocking hook enforced (`agent-loop.ts:739-740`) + project-trust gate (`project-trust.ts:24-29`), exemplary honesty (`SECURITY.md:50` "intentionally does not have a sandbox"). crush: permission service + per-tool-call-id-scoped hook grants (`permission.go:15-29`).
- **5 nanocoder**: jail exists (seatbelt+bwrap, `bash-sandbox.ts`) but opt-in and fail-open to plain `sh` (`bash-executor.ts:83`), plus regex deny-list gating (`execute-bash.tsx:189`). Real machinery + wrong default = below the 6 rung, not equal to it.
- **2 codel**: policy is prompt text ("Always auto approve terminal commands", `agent.tmpl:13`), unauthenticated container-spawning API (`router.go:31`); isolation exists but reads misleadingly.

## token-economy (10)
- **9 codex**: three compaction tiers incl. no-LLM token-budget fresh window (`compact_token_budget.rs:19-40`), prompt-cache-key-gated reuse (`client.rs:354-390`), image budgets, pre/post-compact hooks.
- **8 pi**: projected-context trigger (`compaction.ts:289,919` measures what the model would actually receive), branch-summarization, active cache warming (`cache-warmer.ts`, `cache-stats.ts`).
- **7 cline**: two strategies behind one threshold (`compaction.ts:379`), foreign-history fold on resume (`:692-732`), budget projection + cost fields; no cache discipline.
- **6 crush / 6 nanocoder**: one standard auto-summarize each (`agent.go:1093` unknown-window guard; `utils/auto-compact` + usage calculator).
- **1 codel**: prompt >30000 chars -> "ask the user" (`ollama.go:101-104`). Zero is reserved for not noticing context limits at all.

## orchestration (10)
- **9 codex**: subagents (`codex_delegate.rs`, `multi_agent_tool.rs`), agent graph/message-board crates, `codex queue` first-class (`cli/src/main.rs:1885`), daemon recovery + fork/resume journals.
- **8 cline**: teams as tools (`team/multi-agent.ts` 1943 LOC), durable sqlite cron (`sqlite-cron-store.ts`), hub event journal (`hub-event-log.ts`).
- **7 crush / 7 pi**: crush = loop-detection breaker (`loop_detection.go:11-40`, only anchor with one), race-tested cancellation, sqlite resume. pi = durable package + RPC server plane, but subagents/queues deliberately left to extensions.
- **6 nanocoder**: backpressured event daemon (`daemon.ts:1-12`) + subagents, unproven crash semantics, no budgets.
- **4 codel**: restart-tolerant DB task queue only (`executor/queue.go`).

## interop (10)
- **9 codex**: MCP client AND server (`rmcp-client`, `codex-mcp`), versioned app-server protocol + TS/Python SDKs published by CI, headless `exec --json` (`cli/src/main.rs:3109`).
- **8 cline**: MCP client, published `@cline/*` SDK, four IDE/CLI surfaces (VS Code, JetBrains `ext-jb-test-integration.yml`, CLI, hub/cloud); no ACP.
- **7 crush / 7 nanocoder**: crush = MCP-with-gate-test (`coordinator_mcp_gate_test.go`) + LSP integration, no SDK/IDE surface. nanocoder = full ACP server as its own IDE plugin's transport (`plugins/vscode/src/acp-client.ts` + `source/acp/acp-agent.ts` 1119 LOC) + MCP 1150 LOC - strongest ACP commitment found, no published SDK.
- **6 pi**: excellent headless contract (json/rpc modes `docs/rpc-commands.md`, SDK docs) but no MCP and no ACP at all, by philosophy.
- **2 codel**: GraphQL+websocket to its own UI; two hardcoded providers.

## operability (10)
- **9 codex / 9 pi**: codex = rollout JSONL journals + resume/fork/archive/delete/queue + `debug prompt-input` (`rollout/src/lib.rs:52-109`, `cli/src/main.rs:1885`). pi = session tree with /fork /clone /tree (`docs/sessions.md:20-32`), diagnostics + crash-log + documented session format.
- **8 cline / 8 crush**: cline = git-stash checkpoints with GC (`checkpoint-hooks.ts:12-19`) + session versioning service. crush = recover middleware at every trust boundary (`server/recover.go:18`, `shell/run.go:63`), datadir lock, broad auth.
- **6 nanocoder**: session manager + checkpoint + settings wizards; no visible crash-recovery posture.
- **3 codel**: docker-compose + DB rows; errors vanish into `log.Printf`.

## originality (10)
- **9 codex**: execpolicy DSL, guardian reviewer pool (`guardian_review.rs:1-5`), v8 code-mode (`code-mode/src/grpc_session`), WorldState compaction adapter.
- **9 pi**: session-tree-with-branch-summaries, cache warming as a feature, extension-owned policy/providers/subagents, RPC-first.
- **7 crush / 7 nanocoder**: crush = loop detection + tool-call-scoped hook grants + herdr. nanocoder = backpressure daemon, memory proposal store (`proposal-store.ts`), ACP dogfooding.
- **5 codel**: agent-as-DB-queue with per-flow containers and mandatory human plan-ask (2024-era Devin cosplay) - distinctive shape, no mechanism.

## durability (5)
- **9 codex / 9 cline**: institutional backing + heavy CI/release infrastructure (codex: 30+ workflows, cargo-deny; cline: 350 contributors / 7449 commits, multi-channel releases).
- **8 crush**: funded Charm, CLA, release/nightly cadence.
- **7 pi**: strong solo vision + community, thin institutional backing.
- **4 nanocoder**: one-person collective, no SECURITY.md, young churn (shallow clone noted, but independent signals agree).
- **0 codel**: abandoned - remote `pushed_at 2024-04-29` verified; shallow clone had hidden it. Lesson: shallow rows need remote checks.

## docs-dx (5)
- **9 pi**: 30+ in-repo docs (how-pi-works, session-format, rpc, security, containerization) matching code.
- **8 cline**: full mintlify doc tree incl. feature refs (`docs/features/auto-compact.mdx`), docked for dual-tree confusion.
- **7 codex / 7 nanocoder**: codex = in-repo docs are 206 lines of external pointers (`docs/sandbox.md:1-3`) despite a fine product. nanocoder = structured docs + generated config schema (CI drift-gated), trilingual README.
- **6 crush**: config+hooks docs only, no SECURITY.md.
- **3 codel**: README contradicts behavior ("fully autonomous" vs prompt-mandated plan-ask, `agent.tmpl:11`).

## Census artifacts found by anchor reviews (fix before Phase 2)
- crush `test_loc: 0` -> actually 61,738 (278 `*_test.go`).
- cline `test_loc: 39k` -> actually ~255k (`*.test.ts` missed).
- nanocoder `test_loc: 1,057` -> ~161k specs counted as non-test.
- codel `suggested_tier: T1` -> T0 (dead >12mo, remote-verified); manifest null -> module `github.com/semanser/ai-coder`.
- codex LOC sane (440k `*_tests.rs` correctly counted as tests, unusual pattern coverage).

---

## ERRATA (appended 2026-09-29, post-boundary -- original text above stays frozen)

- **architecture 9, pi**: the phrase "everything else an extension" overstates. Boundary
  re-review found `packages/coding-agent/src/modes/interactive/interactive-mode.ts` = 6,852 LOC
  (`:415` class InteractiveMode) plus `agent-session.ts` = 4,023 LOC. Two independent scorers
  kept pi at 9 (band A stands), but when placing YOUR subject between rungs, treat a
  product/TUI-mode file >5k LOC as a docking factor on both sides -- including if that would
  move you from 9 to 8. The 8->9 gap is not "no big files", it is "loop/state model separated
  from every product surface"; pi itself only partially holds that at 9.
- **verification 8 ceiling** now observed on three subjects (codex, cline, pi): real test
  corps, no in-CI model evals, no fuzzing. Do not award 9+ without evals-in-CI or fuzzing
  evidence (file:line of the workflow that runs them).
