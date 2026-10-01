# atomic-agent - T2 deep review (2026-09-30)

Subject: AtomicBot-ai/atomic-agent, TypeScript, v0.6.5, MIT (LICENSE file + manifest, byte-consistent).
Census row: T2, provenance `original`, head 2026-09-24 (6 days fresh), shallow clone (`.git/shallow` present).

## Anchor question

Closest anchor: **pi** - both pair one separable turn loop behind thin frontends with prompt-cache discipline, a huge property-spec-shaped test corps, gated side-effects with honest docs, and a strong solo vision; atomic sits below pi's rung on durability/operability/interop breadth while its shipped-default enforcement posture is *stronger* than pi's 6-rung "no default protection" (pi 78.5 vs atomic 74.25).

## Census verification (before believing - census was wrong again)

- `test_loc: 0` - **FALSE.** 931 test files in src/ (833 `*.test.ts` + 98 `*.test.tsx`) = **217,290 LOC**, colocated vitest pattern the census glob missed (TS blindness instance; crush/cline/nanocoder/gemini-cli precedent).
- `non_test_loc: 399,314` - overstated ~1.7x. Hand-written src non-test = **238,046 LOC** (`find src -name '*.ts|*.tsx' ! -name '*.test.*' | xargs wc -l`). No generated/vendored blobs found inside src (grep for DO-NOT-EDIT/@generated: zero hits); the surplus is eval*/scripts/docs/lockfile territory. Generated/vendored accounting rule: nothing to exclude, nothing to credit.
- Test screens (dispatch doctrine): **runner-blind** - root vitest include (`vitest.config.ts:5-15`) covers src + eval-memory hermetic tests; eval-agents' 4 `*.test.ts` run only under their own config (`eval-agents/vitest.config.ts`, not in CI) - minor, noted in findings. **Import-graph** - 199/200 sampled test files relative-import production code: real tests, not a binharic-style replica harness.
- `contributors: 1` = shallow-clone artifact; HEAD author single-name, PR #503 at head implies real PR flow. Head fresh; no activity claims needed (rule (b) n/a).

## Line-level reads (T2 mandate)

### Core loop - src/agent/agent-loop.ts (3,042 LOC) + step-executor.ts (3,756)

`runTurn`/`runTurnInner` at agent-loop.ts:1106/1131. One loop, three frontends: CLI `run`, TUI, sidecar all enter `runtime.runTurn` (AGENTS.md invariant 6; sidecar/message-router.ts, tui/chat-orchestrator.ts contain no parallel loop). Per-turn setup reads live posture via getters (`isPlanMode`, `isFusionMode`, `approvalPosture` - agent-loop.ts:131-160), clears the fan-out grant at turn START not turn end (:1236-1241 - an aborted turn cannot leave authority behind), resolves the tool role per turn (:1243-1247), and arms the claim-evidence notice once per turn (:1248-1251, wired into steps at :1688). Recovery machinery is decomposed into own modules with own tests: truncation-recovery, parse-failure-recovery, size-rejection-recovery, empty-completion-recovery, request-deadline, steer-notice. The mega-files hold sequencing, not mechanism - which is what separates it from crush's fused `agent.go`. Mid-turn steering is a tested contract including the no-silent-loss property (`undelivered` on both normal and cancelled paths, agent-loop-steering.test.ts header).

### Context management - no LLM compaction, by design

Strategy = budget-packing + externalized state (AGENTS.md invariant 1 "Project != Prompt"), not summarization:
- Deterministic overcounting estimator, intentionally +10-15% (token-budget.ts:23-38); section budgets from `agent.tokenBudget` (token-budget.ts:49-66); 512-token physical headroom (token-budget.ts:124-131).
- **KV-cache ordering is the architecture**: stable prefix must stay byte-identical within a session (stable-prefix.ts:84,108,156,221,232,293 "KV-cache safe"); everything a step can change sits AFTER `### conversation` in the variable tail (AGENTS.md invariant 2); per-request GBNF grammar narrowing explicitly kept outside the cached prefix (invariant 4). Cache hits measured server-side (`cacheHitTokens` via `resolvePromptUsage`, llama-server-client.ts:1420,1467) and surfaced per step in traces (cli/trace-formatter.ts:71).
- `packConversation` keeps a hysteresis low-water between overflows (`agent.conversationLowWater` = 0.65, config-schema.ts:2431,2746).
- **Learned context window**: a provider 400 naming the real window repacks the next prompt to it once, or `x0.8`-of-prompt estimate when no number is named, then corrects the model catalog via `onContextWindowObserved` (size-rejection-recovery.ts:11-31,57-90; agent-loop.ts:196,2157,2380). Measured-not-assumed.
- Tool-result compression with head/tail overflow semantics (compressor/result-compressor.ts:19-30).
- No summarization fold exists, so no cache-monotonic *fold*; the whole prompt layout is cache-monotonic *packing*. For a local llama.cpp KV cache this is coherent; the absence costs it pi/codex-style tiering credit but nothing for correctness.

### Permission / sandbox - src/approval, src/tools/os/shell-command-guard, src/sandbox

- Shipped default: `approvalLevel: 1` - every gated action asks (config-schema.ts:2740). Five-rung ladder; `fs_read_outside` pinned at 5 (approval-level.ts:57-64); hardline shell-guard rules fire BEFORE the gate and block at every level (approval-level.ts:4-6).
- Grant-scope correctness (the corpus's cleanest): the caller names the scope, the gate reads category/shape from its OWN pending request - "a host cannot grant a category the prompt was not about" (approval-gate.ts:47-53, :340). Shape grant unit is the normalized argv[0] the guard keyed on, **withheld for opaque interpreters** (`bash -c ...`) where the binary hides what runs (shell.ts:196-205). `trust_config` writes are never session-grantable (run-agent.ts:124-126). Novel `redirectablePath`: operator can approve-with-retarget (approval-gate.ts:38-44; fs-write-retarget tool path).
- Guard = layer-ordered rules (hardline > policy > dangerous > trusted > safe-allow), any rule throwing yields `approval_required` fail-closed (guard-engine.ts:43-59); normalization is NFKC + ANSI/NUL/control-strip (normalise.ts:14-40); subshell mode re-tokenizes the full line so compound commands still hit rules (shell.ts:153-165). Hardline layer is regex (rm-root, mkfs, dd-of-dev, fork bomb... rules-hardline.ts) but it only gates catastrophic blocks - approval-by-default carries the load in the shipped posture, so regex-denylist fragility is largely moot at level 1.
- FS containment on realpath with symlink-edge handling (fs-approval-scope.ts:16,133-142) - clean counterexample to mini-kode sibling-prefix grants. Skill scripts must be declared in frontmatter `requires_scripts` and contained under the skill's scripts/ via `relative()` escape-reject (skill-script-runner.ts:41-62); at default level even those prompt - no clone-and-run hole.
- SSRF guard resolves DNS and blocks private/reserved ranges before fetch (web-fetch-ssrf-guard.ts:1-35). No `eval`/`new Function` anywhere in src (grep clean). No OS sandbox (no seatbelt/bwrap/landlock hits) - processes run as the user; per the non-kernel ladder this caps the lane at 7.
- Fail behavior: closed stdin resolves answers to non-`y` => deny; pending-forever on dead hosts is fail-closed (nothing runs); abort of a pending request denies (approval-gate.ts:275-293); `denyAll(sessionId)` exists (:379-395). CLI `--no-approval` is explicit opt-in to level 5, never default (run-agent.ts:48).

## Verification

- 931 files / 217,290 LOC property-shaped tests (agent-loop.test.ts 77 its; steering/claims/recovery/invariant specs; faux provider `llm/provider/fake-provider.fixture.ts` + mock CompletionResult harnesses in 33 files - faux-provider-testing pattern).
- CI BLOCKS on PR+push (test.yml:16-24): typecheck (:106) then full vitest (:128); a two-shape install matrix whose job asserts its own difference in-log (canvas present/absent meta-check, test.yml:88-104) - an anti-vacuous test-of-the-tests touch (molt resonance).
- Two test files excluded to keep main green (test.yml:110-130): both genuinely red, documented, tracked (issue #203), 2/931. Per the amazon-q/trae-agent precedent this is documented-and-narrow, filed as a finding, no lane dock.
- Massive eval assets OUTSIDE CI by design: GAIA-style agent evals, LoCoMo, LongMemEval, memory campaign E1-E12, competitor adapters - `eval/vitest.config.ts:4-8` states they must never run under `npm test`; only workflows are test.yml + release.yml. **evals-in-ci gap x4** (deepagents/forge/jazz family). No fuzzing. => verification 8, ERRATA ceiling, no verification-9 claim made.

## Other lanes - key evidence

- Orchestration: fusion fan-out workers with an operator-approved scoped grant (`approval/fanout-scope.ts`; shell.ts:181-190 - scoped command runs are cwd-bounded, "the directory is the whole of the promise"); the orchestrator-may-not-mutate gate with two documented historical holes and their closes (fusion-orchestrator-mode.ts:5-40); review-stall forces a read-only planner to delegate or reply (agent-loop.ts:1222-1233). Scheduler + sqlite task store with backoff (scheduler/scheduler.ts, tasks/task-backoff.ts). No crash-recovery journals, no queue surface.
- Interop: MCP client incl. per-server GBNF grammar synthesis for MCP tools (mcp/mcp-grammar-builder.ts); state import from SIX rival harnesses - Claude Code, Codex, Pi, Oh-My-Pi, Hermes, OpenClaw - skills/memory/MCP/sessions/cron, dry-run first (src/import/*/); ClawHub skill hub (skills/clawhub/); sidecar NDJSON contract (sidecar/stdio-protocol.ts); user-authenticated `claude`/`codex` CLI as LLM providers (llm/provider/subscription-cli/claude-cli-adapter.ts:138-146,235-247 - honest rival-subscription variant: the operator's own logged-in official CLI, no protocol impersonation). No ACP, no published SDK, no IDE plugin (the Tauri sidecar IS an embed contract).
- Operability: sqlite session store (session/session-store.ts:1-2), session replay + inference replay (src/replay/), trace formatter/command, debug REPL, 66-version config migration chain (config-schema.ts:2292ff), update/uninstall commands, structured logging + analytics with cost metering (analytics/turn-usage-meter.ts, llm/provider/usage-cost.ts). Crash posture modest: scheduler swallows tick errors (scheduler.ts:63); no rewind/checkpoint journal (per-write restore copies exist for files: fs-restore.ts).
- Originality (all code-verified, not marketing): claim-evidence gate (below), bounded-reasoning GBNF anti-degeneration guard (F49: `think-body ::= think-char{0,N}` - "a model that would think for 22 minutes is made to close the block", AGENTS.md invariant 4), dual reasoning-ownership model profiles, grammar-narrowing-without-cache-invalidation, retargetable approvals, fanout-scope grants, six-way rival import matrix, memory fabric (reflection/consolidator/lessons/procedures/bitemporal profile - memory/* with per-phase eval campaigns E1-E12).
- Durability: solo author, v0.6.x young, no SECURITY.md (ls: only project md files), shallow clone honors (contributor/bus-factor evidence-limited per census caveat); offset by real release infra: signed (DigiCert KeyLocker) + notarized multi-OS SEA binaries (release.yml:110-237) and a genuine PR-gated CI at PR #503.
- Docs-DX: AGENTS.md is a 2,625-line invariants guide that cites the tests pinning each invariant (the study's best code-docs coupling since pi); README 796 lines; TESTING/EVOLUTION/BUNDLING/PROMPT/SKILLS/MEMORY_* docs; `.env.example`; config help; error paths carry auth hints (claude-cli-adapter.ts:246-247 "Run claude ... and complete /login").

## Scores

| dimension | score | weight | basis |
|---|---|---|---|
| architecture | 7.5 | 15 | single loop, thin frontends, mechanism decomposition; agent-loop 3042 + step-executor 3756 + declarative config-schema 5855 (>5k errata file) cap at cline-minus |
| verification | 8 | 15 | 217k property LOC blocking CI + matrix; ERRATA ceiling: zero evals-in-CI, zero fuzzing |
| safety-enforcement | 7 | 10 | shipped-default ask-everything + level-invariant hardline + fail-closed + scope-correct grants + realpath + SSRF; no kernel layer |
| token-economy | 8.5 | 10 | cache-monotonic prompt layout, measured cache hits, learned window, hysteresis pack; no fold/warming/realized-$ validation |
| orchestration | 7 | 10 | fusion fan-out with scoped grants + review-stall + scheduler; no journals/queue |
| interop | 7 | 10 | MCP+grammar, 6-way import matrix, sidecar contract, subscription-CLI providers; no ACP/SDK/IDE |
| operability | 7 | 10 | sqlite sessions, replay, traces, migrations, cost metering; thin crash-recovery |
| originality | 8 | 10 | claim-evidence, GBNF reasoning caps, grammar-narrowing, retarget/fanout grants - all in code |
| durability | 5 | 5 | solo, young, no SECURITY.md; real CI+signing cadence; shallow-clone caveats |
| docs-dx | 8 | 5 | invariant guide citing test files; strong error copy |

**Weighted total: 74.25 -> band B.** Strongest dimension: **token-economy (8.5)**. Weakest: **durability (5)**.

Boundary check: 78 - 74.25 = 3.75; 74.25 - 65 = 9.25. No self-flag (outside inclusive-2 on all cuts). Sensitivity: granting arch 8 and orch 7.5 lands 76.5, still B; the B-ceiling plateau's shared DNA fits ("no blocking model-verification, durability dock") except atomic's enforcement DEFAULTS are on, which the doctrine says counts - it just isn't enough to reach A without verification-9.

Provenance: original (no fork/leak markers; grep for ported/fork/based-on clean; the rival-import directories are interop tooling, not lineage). Rules (a)/(b) n/a. Honesty-credit doctrine: n/a - real machinery, shipped on.

Calibration notes: no demotion/promotion. Census corrections above (test_loc 0->217,290; TS-glob blindness instance; non_test inflated ~1.7x). CI-red-test exclusions ruled trivia per amazon-q precedent (loud, narrow, issue-tracked) - finding filed, no dock.
