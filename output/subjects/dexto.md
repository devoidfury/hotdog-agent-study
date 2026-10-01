# T3 review: dexto (truffle-ai/dexto, TypeScript) — B (73.5, upper-mid band)

Tier: T3 giant (census 402k non-test LOC — inflated by a 20,809-LOC generated model registry; hand-written ~183k .ts + 58k .tsx). Census test_loc=4,214 is a **measurement artifact**: 256 test files, ~86.2k LOC measured (`find … | wc -l`). Shallow clone (1 commit, 1 contributor) — history judgments evidence-limited per census caveats; head 2026-09-24, five days before the run: emphatically alive. License Elastic-2.0 verbatim (census "other (file)" = classifier miss). Provenance **original** (scout + econ independently: remote matches, first-party `@dexto/*` naming throughout, zero rival-harness imports; the Codex touchpoint spawns the external binary over stdio JSON-RPC — `codex-app-server.ts:1243`).

**Closest anchor: cline.** All three lanes converged on it independently: server-first harness with a publishable typed SDK, multi-surface (CLI/TUI/webui/HTTP), workmanlike cost-aware token economy, machine-synced docs — below codex/pi on cache discipline and crash-durable orchestration, above crush/nanocoder on orchestration breadth and protocol surface. Totals agree: cline 76.5 vs dexto 73.5 — dexto loses on safety (5 v 6), orchestration (7 v 8), operability (7 v 8) and durability (7 v 9), gaining only originality (8 v 7); architecture, verification, interop, token-economy and docs-dx sit at cline's rungs.

## Dimensions

### architecture — 8 (lane-core)
- The loop is a dedicated module: `TurnExecutor`/`TurnDriver` (`turn-executor.ts:454/:536`) instantiated by the service layer (`vercel.ts:191/:314`), owned per-session (`chat-session.ts:649`), with an enforced-and-tested phase state machine — `TurnDriverStateSchema` strict-zod discriminated union (`:199-247`), `getState()` refuses to checkpoint mid-preparation/mid-step/mid-tool-execution (`:619-627`), refusals asserted at `turn-executor.integration.test.ts:554/:788/:1188/:1435`. Integrator re-verified both citations directly.
- Strict one-way layering: core imports zero hosts; TUI/server/CLI/webui all consume `@dexto/core`. No parallel-loop tell (TUI `processStream.ts` is a UI adapter). Scout's god-file suspect `DextoAgent.ts` (3,724) verified as a wide honest facade: zero provider calls in-file, single `session.stream` entry (`:1394`) — the crush `agent.go` fusion failure is absent (dexto-c7).
- Docked from 9: the resume half of the driver seam has zero production consumers (dexto-c2), `context/utils.ts` is a 34-export cohesion island (c8), `DextoAgent.stream` runs ~500 lines of inline event fan-in before delegating.
- Verdict: cline@8 rung — "defended properties enforced and tested"; the best loop module since codex's `turn.rs`.

### verification — 8 (lane-safety, at the study's 8-ceiling)
- 256 test files, ~86.2k LOC (census wrong by 20x); integration suites confirmed running in CI: root `vitest.config.ts:41` includes `**/*.integration.test.ts` and `build_and_test.yml:30` runs plain `pnpm test` — scout's open risk resolved.
- Tests prove properties, not existence: approval precedence matrix, replay-conflict rejection (`approval/manager.test.ts:236,:341,:583`), traversal rejection (`path-validator.test.ts:168-191`), checkpoint roundtrips, retry-budget exhaustion with observable `llm:retrying` events.
- The 8→9 door is closed by errata: no evals-in-CI, no fuzzing, `test:ci --coverage` exists but CI never invokes it; and the one safety-relevant module with zero tests (command-validator) caps any upward argument.

### safety-enforcement — 5 (lane-safety)
- The enforcement half is 6-rung-plus: approval binds in-loop (`turn-executor.ts:1919-1935`), durable identity-scoped approval records with conflicting-replay rejection, manual mode fail-closed for tools without authored policy (incl. MCP) — better approval plumbing than any T3 subject reviewed (dexto-s4).
- But the default is the permissive mode: `DEFAULT_PERMISSIONS_MODE='auto-approve'` (`tools/schemas.ts:9`) short-circuits before any tool-authored policy (`tool-approval-policy.ts:59`); the shipped scaffold inherits it (`agents/agent-template.yml:79`); and the docs claim the opposite ("manual (Default)", `permissions.md:35`) — nanocoder's exact wrong-default docking plus a security-relevant docs/code contradiction pi/cline/crush don't have (dexto-s2).
- Novel hole: pattern approval keys tokenize on whitespace ignoring shell metachars (`command-pattern-utils.ts:49-59`, integrator re-verified), so a remembered `git push *` key blesses `git push x && <anything>`, persisted to storage by default (dexto-s1). The one backstop under auto-approve is an untested regex deny-list with self-documented holes whose second output channel (`requiresApproval`) is dead code (dexto-s3). No sandbox, no injection defense, full `process.env` to every child.

### token-economy — 7 (lane-econ)
- Cline@7 rung via a different mix: reactive-only compaction (0.9 threshold) alone would be a 6, rescued by per-step tool-output pruning (40K protect / 20K min-savings, `turn-executor.ts:2258-2318`, integrator re-verified the constants) and an actuals-based next-input projection with calibration-drift logging (`manager.ts:1392-1417`, re-verified) — pi@8's projected trigger in miniature. Cost + cache-read/write accounting complete.
- Not 8: no proactive tier, no provider-overflow recovery (zero context-length handling in `llm/errors.ts`), flat 1000-token image estimate, one static system-prompt cache breakpoint (`vercel.ts:191-205`).

### orchestration — 7 (lane-econ; the highest 7 in the study)
- Durable sqlite cron with compensating deletes (`tools-scheduler/manager.ts:77-79`, `storage.ts:113-146`), DB-backed steer/follow-up queues consumed mid-turn (dexto-m1), governed subagents: self-spawn block, concurrency + iteration + reasoning-downgrade budgets, allow-list (`agent-spawner/runtime.ts:261-267,:396-410`).
- Ceiling: the orchestration plane itself is an in-process `Map` with `Math.random` ids and 5-min TTL (`task-registry.ts:31-50`) — restart silently kills background tasks and waiters; no loop detection, no orchestration journal. The durable-substrate argument that carried cline to 8 fails here.

### interop — 8 (lane-econ)
- cline rung via a different route: MCP client with all three official transports (`mcp-client.ts:161,:226,:318`) *and* a dual-transport MCP server with ephemeral-session lifecycle (`mcp-handler.ts:22,:66-80`) *and* an A2A v0.3.0 server no anchor ships (`a2a/jsonrpc/methods.ts:4`, integrator re-verified the header) *and* a versioned `private:false` SDK (`@dexto/client-sdk` v1.12.1).
- Docked from 9: the primary headless entry `dexto run` emits prose `[tag]` lines only (`headless.ts:26-44`) — no machine-readable event contract; no ACP, no IDE plugin.

### operability — 7 (lane-core)
- WAL sqlite + Postgres advisory locks, fork-with-rollback (`session-manager.ts:318-394`), persisted tool executions, cancel-consistent partial persistence (`stream-processor.ts:492/:502` — no orphaned tool-call blocks after Ctrl-C), OTel span tree per loop step plus a `dexto span/trace` CLI, auth-on-by-default with warning.
- Not crush@8: crash continuity is the weakest sub-axis — the checkpoint/resume seam has zero production callers and the server itself TODOs restart survival (`messages.ts:319-321`); the `RuntimeEventStore` journal is shipped but never written (dexto-c6); error shapes self-admittedly inconsistent across mcp/queue/a2a/sessions routes (`error.ts:4-11`); no rewind anywhere.

### originality — 8 (integrator-scored)
- Verified in code, not marketing: strict-zod serializable turn state that refuses transient snapshots (re-read `turn-executor.ts:199-247,:619-627` — no anchor has a validated serializable continuation state); history-unchanged-as-precondition retry guard (re-read `turn-executor.ts:1692-1733`, direct answer to the retry-blind-backoff anti-pattern); actuals-anchored token projection with drift logging (re-read `manager.ts:1392-1417`); pruning tier with read-time placeholders while the DB keeps originals; A2A server (re-read `methods.ts` header); CI drift gates including custom ESLint route-contract rules that ship with their own tests (`eslint-rules/require-openapi-*.js` + `.test.ts`) — mechanism-level docs-vs-code discipline no anchor was observed to have.
- Docked from 9: compaction is the standard reactive-summarize design; MCP/A2A implement other bodies' specs rather than inventing protocols; and one genuine *negative* novelty — the whitespace-tokenized approval-key scheme (dexto-s1) invents a bypass shape none of the anchors exhibit. 8 sits above the cline/crush 7 ("competent standard design"), below pi/codex 9 (field-defining engine novelty).

### durability — 7 (integrator-scored, from census metadata)
- Activity: head 2026-09-24 (5 days pre-run), manifest v1.1.3, head commit `chore: version packages (#924)` — PR number plus `.changeset/` imply a long real history hidden by the shallow clone; census contributors=1/commits=1 are measurement artifacts per the standing caveat, not bus factor. Nine CI workflows including `changesets-publish.yml` and `release-standalone-binaries.yml` — institutional release engineering, not hobby cadence.
- Institutional backing: Truffle AI (own org, docs site, 22 shipped agent configs, CODE_OF_CONDUCT/CONTRIBUTING, Docker packaging).
- Caps: license is Elastic-2.0 — real and verbatim, but non-OSI source-available, which caps community-derivative longevity vs cline's Apache-2.0 (9) and crush's MIT (8); governance maturity is partial (SECURITY.md exists only under `packages/server/`, not at root); bus factor unverifiable from the sandbox. Rung match: pi's 7 (active + backed, smaller proven base than the 8-9s).

### docs-dx — 8 (lane-econ)
- 65-page Docusaurus tree + rendered OpenAPI reference, CI drift gates on spec *and* CLI README plus the custom ESLint contract rules (dexto-e12), curl installer, 2.4k-line setup wizard, recovery-bearing typed errors with option lists.
- Not 9: compaction knobs and the strategy DI seam undocumented (token math lives in a 949-line internal `feature-plans/` write-up), architecture section 3 thin pages, docs/api vs docs/docs split, no root SECURITY.md.

## Score disagreements resolved

The three lanes scored disjoint dimensions, so there are no numeric conflicts to arbitrate; four qualitative tensions are resolved explicitly:

1. **"Weakest dimension" claim collision.** Lane-core called crash continuity "the weakest observed dimension" (docking operability 7); lane-safety scored safety-enforcement 5. Resolution: both statements stand within their lanes, but on the ten-dimension board the weakest is **safety-enforcement (5)** — crash continuity is a weak sub-axis *inside* operability, and its evidence (dexto-c2/c6) is already priced into the 7. The reported weakest_dimension is safety-enforcement.
2. **SECURITY.md.** Scout said none exists; lane-safety corrected: `packages/server/SECURITY.md:225` exists with a reporting contact; lane-econ still docked docs-dx for "no SECURITY.md." Resolution: both are true — the file is server-scoped only, no repo-root security policy. The docs-dx dock stands (root-level absence is what users and the crush docking precedent measure), and lane-safety's correction is honored as a fact, not a score change.
3. **Docs honesty vs docs-dx 8.** Dexto-s2 shows the docs actively misstate the permissions default — a docs defect that could argue docs-dx down. Resolution: it is docked where its blast radius is (safety-enforcement 5), following the code-run precedent of using one fact in at most one dimension per reviewer; lane-econ's 8 reflects breadth/contract discipline. Flagged here rather than double-counted; if Phase 3 treats a wrong default claim as a docs-dx fact, docs-dx 7 → total 73.0, band unchanged.
4. **Anchor-vector split.** Lane-core anchored architecture to cline@8, lane-safety anchored safety to nanocoder@5, lane-econ anchored overall to cline. No conflict: dimension-scoped anchors are consistent, and cline is adopted as closest_anchor on weight — architecture+verification (30 of 100) sit at cline's rung, and the aggregate profile (SDK, multi-surface, workmanlike economy) is cline-shaped.

## Verdict
**Strongest: architecture (8, tied on raw score with verification/interop/docs-dx, chosen on weight and evidence density).** The validated-checkpoint-union turn loop (enforced at `turn-executor.ts:199-247/:619-627`, pinned by a 4,456-LOC integration suite that drives the real loop through its public step interface) is the strongest anchor-grade architecture signal found in the study since codex's `turn.rs` — the facade delegates, the layering is one-way, the retry guard and cancel-persistence are tested properties, not aspirations.

**Weakest: safety-enforcement (5).** Real, durable, idiomatically-tested approval machinery sitting on `auto-approve` by default (`tools/schemas.ts:9`) with docs claiming the opposite, a metachar-blind approval-key bypass that persists to disk (`command-pattern-utils.ts:49-59`), and a dead-code zero-test regex validator as the only backstop — machinery plus wrong default plus a novel hole, one rung below cline/pi/crush.

Weighted 73.5 → **B** (upper-mid; 4.5 below the A floor, 8.5 above the B/C floor; no boundary-flag threshold hit). Not demoted: provenance original (rule: sync-fork must not outrank upstream — no upstream applies; econ's rival-import grep clean), not archived/dead (head 5 days pre-run, active release pipelines), no demotion occurred, nothing silent.
