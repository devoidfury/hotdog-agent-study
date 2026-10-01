# nausicaa-harness - T2 deep review

- Repo: /data/samples/agents/nausicaa-harness (jackispm/nausicaa-harness, MIT, v0.2.0)
- Census: 108,061 non-test / 96,235 test LOC, 1 contributor, shallow=true, head 2026-09-20, tier T2.
- Measured: src = 112,855 LOC across 14 top-level modules; test = 104,276 LOC, 252 `*.test.ts`
  (1,592 `it(` blocks in test/unit alone). Census is sane. `.git/shallow` present with 1 commit:
  activity claims limited accordingly, but CHANGELOG shows dated releases 0.1.8 -> 0.2.0 between
  2026-09-11 and 2026-09-20, so the project is demonstrably alive; no remote check needed.

## Anchor question (one sentence)

Closer to **cline** than to any other anchor, because it is a TypeScript host with a strong
product-level test corpus and durable multi-agent orchestration but enforcement that is opt-in
rather than default; it lands below cline mainly on interop (no ACP/IDE/SDK) and durability.

## What this thing is

A "ledger-driven multi-lane" harness: every run is an append-only, event-sourced Ledger
(JSONL, sha256 content hashes, idempotency keys, CAS withdrawal); the agent loop (MainLoop),
context assembly (Fukai), tool admission/execution (Mowe), and cross-run messaging (A2A) are
separate modules behind explicit ports (`src/domain/ports.ts`). Lanes = Main (Nausicaa),
Teto (advisory observer), workers, team members. Pi's `pi-ai`/`pi-tui` are the provider/TUI
substrate; `THIRD_PARTY_NOTICES` pins adapted snippets from Pi, Prime Agent, and DeepSeek
Harness's sandbox-local (all MIT, commit-pinned).

## Line-level read #1: core loop (`src/runtime/main-loop.ts`, 2,360 LOC)

- `MainLoop.run` (`:481`) is a dependency-injected step loop over ledger refs, not raw messages:
  conversation is `conversationRefs` into a content-addressed store (`:500-560`), and only refs
  already shown to the model are compaction-eligible (`pressureEligibleConversationCount`, `:509`
  with the explanatory comment) - fresh input always gets one raw read.
- Per-step tool surface is filtered twice before the request: plan mode strips everything whose
  `effect` is not read/compute (`:643-648`), and catalog/skill-schema pairs are dropped atomically
  when budget forces eviction (`:746-770`) - the schema is never advertised for a catalog that
  was not rendered (`hasRenderedSkillCatalog`, `:759`).
- Token budget: forward reservation with priority - Main can pre-empt advisory (Teto/reflection)
  reservations and take a bounded admission slack so a valid Main step is not killed by
  in-flight auxiliary reservations (`:795-825`, `src/runtime/run-token-budget.ts:82-125`).
- Every request persists `model.requested` with requestHash, prefixHash, deadlineAt,
  estimatedInputTokens and an idempotency key (`:891-912`); the wall-clock deadline is started
  *before* the ledger append so event and request describe the same deadline (`:873-880`
  comment). Retries are wrapped at the provider boundary, snapshots outside it (`:410-421`).

## Line-level read #2: compaction / context (`src/fukai/`)

- Two tiers: per-turn `selectCompaction` (`main-loop.ts:668-684`) plus a
  `compactForPressure` gate evaluated against the built view's `estimatedInputTokens`; a pressure
  candidate is adopted only if it still leaves >=1 token of output headroom after reservation
  (`:827-870`). The pressure decision itself is deterministic, provider-free, and has a
  `minimumGainTokens` floor so useless compactions are skipped
  (`src/fukai/compaction-pressure.ts:53-75`, reasons incl. `insufficient-predicted-gain`).
- Degradation ladder: `FukaiBudgetError` -> retry without compaction
  (`main-loop.ts:742-749`) -> retry dropping skill catalog and its schema together (`:753-764`).
  "Compaction is an optimization over the bounded raw Inbox; discard only the optimization"
  (`src/fukai/context-provider.ts:133-136`).
- Compaction is ledger-durable: requested/completed/failed/fallback events with derived attempt
  ids and budget charging (`src/fukai/compaction-runtime.ts:28-50`, `:146-186` idempotent
  replay), watermark monotonicity enforced at commit (`src/fukai/core.ts:676,928`).
- Cache discipline: `prefixHash` hashes only stable content (system prompt, tools, policy,
  edge context) while `cacheKey` adds the dynamic tail (`context-provider.ts:96-104,430-446`);
  `src/observability/cache-evidence.ts` projects per-attempt prefix continuity + provider cache
  read/write outcomes, and `test/eval/cache-artifact.ts` gates release on a recorded live cache
  probe with provenance (`pass|hold`, `:38-59`). No explicit cache breakpoints here - provider
  cache instruction is delegated to pi-ai (THIRD_PARTY_NOTICES:3-5).

## Line-level read #3: permissions / sandbox

- Three profiles: `read-only | workspace | full-access` (`src/cli/selectors.ts:72`). Default is
  full-access ("Commands and file writes use your user permissions by default", README.md:262);
  SECURITY.md:3-5 is equally honest ("Do not use it as a security boundary for hostile code").
- The `workspace` profile wires bash through `WorkspaceCommandSandbox`
  (`src/runtime/run-runtime.ts:471-484`): macOS seatbelt or Linux bwrap with fixed launcher
  paths, a functional probe that never runs the requested command (`workspace-command-sandbox.ts:167-207`),
  and **fail-closed**: probe failure or profile refusal makes the bash tool simply not exist
  (`:129-134`, `:159-161` throws; run-runtime only enables shell when `availability().available`)
  - the exact inverse of nanocoder's fail-open jail. Protected targets include `.aws/.ssh/.gnupg/.kube`
  and the harness dataDir (`:32-38`, `run-runtime.ts:481`), host-automation binaries blocked on macOS
  (`:24-29`).
- Approval is durable: `requiresApproval` tools record a `MoweApprovalDecisionRecord` through an
  optional lifecycle sink *before* the effect, lifecycle failure fails closed
  (`src/mowe/types.ts:144`, `src/mowe/executor.ts:389-447`); the ledger binds approval decisions
  to the preceding tool request and enforces requested->admitted->started chains
  (`test/unit/ledger.test.ts:427,491`).
- Tests prove behavior, not existence: seatbelt profile shape + path escaping, bwrap read-only
  host + isolated net + protected metadata, fail-closed on unsupported platforms, profile refusal
  => unavailable (`test/unit/workspace-command-sandbox.test.ts:29,57,67,90,106,163`).
- No OS sandbox exists for the default full-access profile, and no network-egress approval under
  it - enforcement is real but opt-in. This is the ceiling on safety-enforcement.

## Verification

- CI (`.github/workflows/ci.yml`): typecheck + `npm test` (offline: live/eval/smoke excluded,
  `package.json:61`) + `npm run eval` (deterministic eval-harness tests) + build+smoke + `npm pack
  --dry-run`, on Node 22.19 and 24.x. Publish requires the tag commit to be on main with a green
  CI run (`publish.yml:44-52`) - release gating is enforced, not decorative.
- Test corpus names read as property specs: `ledger.test.ts` "rejects an idempotency key reused
  for a different fact" (`:119`), "enforces append-only input replacement and withdrawal CAS"
  (`:213`), `daemon-ledger-fence.test.ts`, `agent-loop-abort-parity.test.ts`,
  `daemon-worker-recovery-journal.test.ts`, recovery/ and protocol/ directories.
- `test/eval/preregistered-contract.ts` is a preregistered experiment design for the Teto feature:
  frozen fixture/scorer hashes, four arms incl. a `teto-shadow` placebo, per-pair budget
  envelopes, 95% bootstrap (`:11-27,32-57`). Live arms exist (`test/live/`, `test/eval/*-live-*`)
  but need credentials and never run in CI. Per the 8-ceiling errata: deterministic evals in CI
  but no model evals or fuzzing in CI -> 8, not 9.

## Dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 7 | clean port separation (`domain/ports.ts`, loop at `main-loop.ts:481` independent of host); docked for `src/runtime/session-controller.ts` 6,365 LOC (`:563` class SessionController fuses wiring/policy/selection/polling - errata >5k docking factor) and `src/cli/interactive.ts` 4,718 |
| verification | 8 | 1,592 unit `it(` blocks + protocol/recovery suites; invariants in names (`test/unit/ledger.test.ts:119,213,491`); CI eval+smoke+pack (`ci.yml:37-45`); no model-evals or fuzzing in CI (8-ceiling) |
| safety-enforcement | 7 | fail-closed OS sandbox w/ functional probe (`workspace-command-sandbox.ts:167-207`, tests `:90,163`), durable approval lifecycle fail-closed before effect (`mowe/executor.ts:389-447`, `mowe/types.ts:144`), plan-mode tool-effect filter (`main-loop.ts:643-648`); docked because default profile is full-access with no sandbox/egress gate |
| token-economy | 8 | two-tier compaction + predicted-gain pressure (`compaction-pressure.ts:53-75`), degradation ladder (`main-loop.ts:742-764`), priority budget reservations with main-slack (`run-token-budget.ts:82-125`), cache-evidence projection + release-gated probe (`observability/cache-evidence.ts:19-33`, `test/eval/cache-artifact.ts:38-59`); cache instruction delegated to pi-ai, no warming |
| orchestration | 8 | execution leases with monotonic fencing tokens (`runtime/execution-lease.ts:9-15`, `file-execution-lease.ts:225`), worker/command recovery journals (`daemon-worker-recovery-journal.ts:15-37`), daemon supervisor + reconciliation suites (`test/protocol/daemon-*.test.ts`), ledger-verified fork lineage (`ledger.ts:488-495`); no loop-detection breaker anywhere |
| interop | 6 | real MCP client with namespacing (`mowe/edges/mcp.ts:822,1137`), headless `--print`/`--json` NDJSON (`cli/args.ts:9,541-552`), library export; no ACP, no IDE surface, no published SDK, A2A is a proprietary cross-run protocol (`a2a/cross-run-contract.ts:376`) |
| operability | 8 | /resume /clone /export /import (`cli/command-registry.ts:280`), permission diagnostics returned with failures (`tools/permission-diagnostics.ts`, `tools/bash.ts:85-88`), cache evidence projection for UI, lease-guarded daemon recovery; young, but mechanisms are broad |
| originality | 8 | Teto advisory observer lane with shadow-placebo preregistered eval (`src/runtime/teto-lane-scheduler.ts:100`, `test/eval/preregistered-contract.ts:13-27`); event-sourced ledger + fencing-token leases as the TS-harness spine (`ledger/ledger.ts:323-328`); cache-evidence release gate. Sandbox internals honestly credited as adapted, not invented |
| durability | 4 | 1 contributor, v0.2.0 beta, no institution; but real release governance (`publish.yml:44-52`) and SECURITY.md exist |
| docs-dx | 8 | 15 in-repo docs that cite their own sources (`docs/runtime-loop-baseline.md:14-24` table matches actual lane executors), bilingual README, CHANGELOG with dated entries, install.sh/ps1 |

**Strongest dimension: orchestration.** Fencing-token leases + recovery journals + ledger-enforced
fork lineage exceed anything but codex in the corpus; the crash semantics are tested, not asserted.
**Weakest dimension: durability.** Bus factor of one, 0.x, no institutional backing.

Weighted: 7*15 + 8*15 + 7*10 + 8*10 + 8*10 + 6*10 + 8*10 + 8*10 + 4*5 + 8*5 = 735 / 10 = **73.5 -> B**.

## Provenance & calibration

- Census "original" stands. Not a sync-fork: it *depends on* pi-ai/pi-tui as packages and
  *adapts* named snippets from Pi, Prime Agent (in-corpus subject), and DeepSeek Harness, all
  MIT with commit hashes in `THIRD_PARTY_NOTICES` - rule (a) does not bite (divergent-by-design
  derivative with attribution, not a tree copy). No identity-confusion risk observed: name is
  unique in the corpus; no similarly named sibling.
- Census corrections: test LOC ~96.2k vs 104.3k measured (minor glob miss, immaterial); LOC sane
  overall; no vendored blobs (node_modules absent, assets/ is SVGs).
- Shallow clone (1 commit): no dead/low-activity claim made; CHANGELOG gives dated releases.
- No calibration demotions. 73.5 is not within 2 pts of any band boundary (78/65).

## Notes for synthesis

- If Prime Agent's own review happens, cross-check its `7787f074` snippets against
  `src/cli/auth-menu.ts`, `queue-selection.ts`, `tool-renderers.ts` for adaptation fidelity
  (THIRD_PARTY_NOTICES claims "minimal, layout-only" ports).
- `src/runtime/l0-agent-loop.ts` (598 LOC) is documented as unused in production
  (`docs/runtime-loop-baseline.md:19`) - a second loop kept for embedding; watch for drift.
