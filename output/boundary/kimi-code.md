# Boundary re-review: kimi-code (MoonshotAI/kimi-code, MIT)

Provisional 76.5 (B). Swing question: architecture 7-vs-8 (claimed v1/v2 dual-engine docking). Named originality items to verify. Pi-tui vendoring ruling. Verification-9 check.

Anchor sentence: closest to **cline (8, 76.5 B)**: an engine-as-package (`agent-core-v2`) consumed by thin multi-surface hosts with tested-approval safety and nothing underneath; kimi edges cline on originality and ACP, trails on durability evidence; sits at the same band ceiling, not above it.

## 1. Architecture: 7 -> 8

The claimed docking reason is **stale at HEAD**: the v1 engine is *gone from the tree*. There is no `agent-core` v1 package; `ls packages/` shows only `agent-core-v2` (+ `migration-legacy`, which migrates **kimi-cli data directories**, not engine code - `packages/migration-legacy/src/index.ts:1-3`). `grep -r KIMI_CODE_LEGACY` over all source returns zero hits; the only remaining trace is a dead CI job (`ci.yml:72-88` sets `KIMI_CODE_LEGACY_FLAG: "1"` that no code reads) plus CHANGELOG history. So the cline-style *live* dual tree - the actual reason cline is docked at 8 - does not exist here.

Structural case for 8:
- DI service-registry boundaries (VS Code-style `@IService` scoping, `packages/agent-core-v2/src/_base/di/`); loop (`agent/loop/loopService.ts`, 2285 LOC) is separated from policy (`agent/permissionPolicy/*` as 15 small composable policy files), state (`agent/state` + replay fold), and tool exec - contrast crush's fused `agent.go` 2393 + `coordinator.go` 1877 (anchor 7).
- Hosts consume the engine over a transport (`kap-server`, `node-sdk` published as `@moonshot-ai/kimi-code-sdk`, `packages/node-sdk/package.json:2,38`): TUI, VS Code, ACP, vis. Engine-first, like cline's SDK-first.

Remaining docking (keeps it at 8, not 9, per ERRATA): `apps/kimi-code/src/tui/kimi-tui.ts` 4238 LOC (`:338 class KimiTUI`) - under the 5k product-surface threshold but the largest own-authored file; `packages/node-sdk/src/sdk-rpc-client-v2.ts` 2889; `agent-core-v2/src/agent/media/webp-dec-wasm.ts` 1839 (generated-ish); dead legacy CI job.

**God-file attribution:** the large files are **kimi's own** (`kimi-tui.ts` 4238, `sdk-rpc-client-v2.ts` 2889). Vendored pi-tui's largest is `packages/pi-tui/src/components/editor.ts` 2625 - pi's god files (`interactive-mode.ts` 6852, `agent-session.ts` 4023 per ERRATA) were **not** vendored. No pi god-file pattern was inherited at material scale.

## 2. Verification: 8, ceiling stands (verification-9 blocking line CONFIRMED ABSENT)

- Whole-repo grep of `.github/workflows/*.yml` for eval/fuzz: **zero hits**. No in-CI model evals anywhere - confirmed, as expected.
- The only fuzzing is component-level and deterministic: `packages/tree-sitter-bash/test/fuzz.test.ts:3-10` (fixed-seed PRNG: token soup, fixture byte-mutation incl. NUL, nesting bombs; contract "parse never throws", run via the blocking sharded test job `ci.yml:36,49`). Literally it crosses the errata's "fuzzing" gate, but it fuzzes one parser, not the system, and there are no evals; anchors' 8 ceiling stands for the whole suite.
- Test corps is enormous and behavior-asserting: 997 test files / 404,649 LOC; policy tables assert nested detection (`test/agent/permissionPolicy/permissionPolicyService.test.ts:280-303`: `bash -c 'eval "shutdown now"'` -> ask, `env -i FOO=bar shutdown` -> ask).
- **CI gaps (dock within 8, findings b1/b2):** minidb excluded from root vitest projects (`vitest.config.ts:8`) and no workflow runs `pnpm --filter @moonshot-ai/minidb test` - the WAL store's 33 test files are never gated; `test-windows` disabled (`ci.yml:93 if: false`); `test-vscode-legacy` is a phantom job (flag unread).

## 3. Originality: 8 - all four named items survive verification as real tested code

1. **grammar-parsed-bash-policy: REAL.** `agent/permissionPolicy/policies/dangerous-command-ask.ts:121-149` consumes `IBashParserService`; unparseable/aborted parse => `unanalyzable` => **ask in default mode** (`:145`), bypass only in explicit yolo/auto. Backed by a pure-TS reimplementation of tree-sitter-bash 0.25.0 (`packages/tree-sitter-bash`) tested **differentially against the official wasm build byte-for-byte with a known-diff registry that fails loudly when a documented deviation is silently fixed** (`test/differential.test.ts:3-16`) plus the seeded fuzz above. This is the strongest single idea in the subject; distinct from codex's execpolicy DSL (allow/deny rules vs AST analysis).
2. **minidb WAL store: REAL, with a caveat.** `packages/minidb/src/wal.ts` (483 LOC, group commit, everysec/no fsync policies) + `test/wal.test.ts:53-90` (order/durability, 1000-append group commit), fault-injection (`compaction-fault.test.ts`); consumed for real by `agent-core-v2`, `kap-server`, `apps/kimi-code` (package.json deps). Caveat: its tests never run in CI (b1) - mechanism real, gate missing.
3. **intent-card fork governance: REAL.** `packages/pi-tui/UPSTREAM.md` pins upstream `earendil-works/pi` subtree `packages/tui` at commit `53816d7...` (2026-09-14), defines reconstruction procedure and keep/absorbed intent cards ("decision + why not in the app"); cards map to regression tests in-tree (`test/regression-overlay-cjk-boundary.test.ts`, `regression-regional-indicator-width.test.ts`, `bug-regression-isimageline-startswith-bug.test.ts`). Enforcement is procedural (no drift-gate script in `scripts/`), but the pi-tui suite is CI-gated (`ci.yml:53-67`). Genuinely novel vendoring discipline.
4. **event-sourced undoable state: REAL.** Wire records folded to state (`agent/replayBuilder/fold.ts:30-50`: `context.append_message`, `context.undo`, `context.apply_compaction`, `forked`, `plan_mode.*`...) with versioned wire migrations (`migrateV1_4ToV1_5`); `agent/undo/undoService.ts` emits `ContextUndone` events; tested (`test/agent/contextMemory/undoPrecheck.test.ts`, `splice-replay.test.ts`).

Also original, unclaimed in the named list: KAOS environment abstraction (local/SSH execution plane, `packages/kaos/src/kaos.ts:8-14`, `ssh.ts`) and tower (git-worktree parallel-change protocol, `features/tower/protocol/store.ts:1-30`).

## 4. pi-tui vendoring ruling: library dependency, rule (a) N/A

Documented pin + MIT + per-file diff procedure + its own CI job = a governed vendored **library**, not lineage: kimi-code's architecture, loop, policy, state, transports are entirely its own; nothing sync-fork-like. Rule (a) (derivative must not outrank upstream) does not apply; kimi-code is not a pi derivative. Inherited-god-file check: done above - negative. Provenance field records this.

## 5. Other lanes (anchored)

- safety-enforcement **6**: exactly the cline/pi/crush "tested approval, nothing underneath" rung - permission pipeline is tested and fail-closed on unanalyzable input, workspace-trust gates MCP (`app/mcpRegistry`, `workspace/workspaceTrust/`), but zero sandbox evidence (grep seatbelt/bwrap/sandbox across engine + kaos: none).
- token-economy **7**: ratio-trigger full compaction with reserved context, overflow-reduction ladder, per-turn caps (`agent/fullCompaction/strategy.ts:6-30`) tested (172k LOC test file); cache discipline present but passive (per-provider `prompt_cache_key`/`metadata.user_id` propagation, `src/human/test/llm/cache-key.test.ts:105-163`; fork cache-hit probing, `agent/usage/cacheProbeService.ts:22-28`); no projected-context measurement or warming (pi-8 features).
- orchestration **8**: swarm subagents with timeout budgets, goal budgets, cron, tower worktrees, wire-replay resume + fork (`fold.ts` `forked`); above crush-7, comparable to cline-8; crash semantics not demonstrated.
- interop **8**: MCP client, ACP server package, published SDK, VS Code + CLI + remote-control + vis; headless `-p`. No MCP-server surface (codex-9 gap).
- operability **8**: event-sourced undo/rewind, sessions/resume, kimi-cli migration tooling, vis transcript viewer, debug events; crash posture undoc.
- durability **8**: Moonshot AI institutional backing, changesets + release/pkg-pr-new/docs-deploy workflows, PR #4018 volume, HEAD 2026-09-24 active; shallow clone -> contributor/bus-factor evidence-limited.
- docs-dx **8**: 58 in-repo docs (28 en + zh mirror), configuration/reference/guides trees, SECURITY.md with GHSA channel.

## Verdict

77.0, band **B**. Architecture moves to 8 (stale dual-engine docking; single-engine DI design with cline-grade but slightly cleaner boundaries), and all four originality claims survive; minidb's CI exclusion, phantom legacy CI job, no sandbox, and no evals-in-CI hold it below the A floor. **A floor not crossed.** Same ceiling as cline: this is a B-band ceiling subject on current evidence.
