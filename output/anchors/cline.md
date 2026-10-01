# Anchor review: cline (cline/cline, TypeScript) — B (76.5, top of band)

Tier: T2/T3-scale monorepo (census 917k non-test / 39k test — test figure badly wrong, see notes). Full git history: 350 contributors, 7449 commits. Apache-2.0.

**Closest anchor: n/a — is anchor.** Structurally closest to codex (sdk-first core + IDE/CLI hosts) but with approvals instead of sandboxing as the safety layer.

## Dimensions

### architecture — 8
- SDK-first split: one loop (`sdk/packages/agents/src/agent-runtime.ts`, 2825 LOC) consumed by vscode/cli/hub hosts; `apps/vscode/src/core` is now a 33k-LOC host with no >1.1k-LOC god files (max: hook-factory.ts 1054).
- Extension points are the architecture: beforeTool hooks chain before tool policy and approval (documented and enforced in `sdk/packages/core/src/extensions/tools/command-guard-extension.ts:11-13`).
- Deduction: mid-migration dual-tree (sdk/packages + apps/vscode legacy core) and 2.7k-LOC `runtime/host/local-runtime-host.ts`, 3k-LOC `cloud/controller.ts`.

### verification — 8
- 255,600 LOC in 792 `*.test.ts` files (census said 39k — census pattern missed `*.test.ts`); property-style loop tests: `agent-runtime.test.ts:2355` asserts approval is requested when a policy disables auto-approve; :1821 asserts approval ordering with parallel tools.
- Compaction tested heavily, incl. live-provider variant: `core/src/extensions/context/compaction.test.ts` 4593 LOC, `compaction.live.test.ts`.
- CI: `sdk-test.yml`, `ext-vscode-test.yml`, `ext-vscode-test-e2e.yml`, `desktop-test.yml` on push.
- Deduction from 10: evals (`evals/benchmarks/tool-precision`, `evals/e2e/run-cline-bench.ts`) exist in-repo but no workflow runs them; no fuzzing.

### safety-enforcement — 6
- Approvals genuinely bind in the loop: policy resolution then `requestToolApproval`, rejection skips the tool (`agent-runtime.ts:2384-2397`), and that behavior is tested (`agent-runtime.test.ts:2355`).
- Plan-mode command blacklist enforced as a hook ahead of approval so blocked commands never prompt (`command-guard-extension.ts:34-79`); telemetry on blocks.
- The gap: no OS-level sandbox anywhere in the SDK — an approved `run_commands` runs with full user privileges (no sandbox module under `sdk/packages/core/src`; only "sandbox" hits are auth/cloud test names). Standard design, tested, honest (docs sell auto-approve as the control), but a 6 by anchor definition next to codex's 10.

### token-economy — 7
- Two compaction strategies (agentic vs basic) behind a threshold ratio (`context/compaction.ts:379 COMPACTION_TRIGGER_RATIO`, `agentic-compaction.ts`, `basic-compaction.ts`), foreign-history fold on resume so a resumed session opens with a summary (`compaction.ts:692-732`).
- Budget projection with tests (`context/budget-projection/project.test.ts`), cost tracking fields in `sdk/packages/shared/src/agent.ts`, telemetry for unexpected reasoning tokens (`agent-runtime.ts:40 captureAgentUnexpectedReasoningTokens`).
- Deduction: no prompt-cache engineering evidence (no cache-key discipline comparable to codex client.rs:354-390); auto-compact can reshape prefixes freely.

### orchestration — 8
- Multi-agent teams: `extensions/tools/team/multi-agent.ts` (1943 LOC), `spawn-agent-tool.ts`, `configured-agent-tool.ts`; docs `docs/cli/agent-teams.mdx`.
- Durable scheduling: sqlite-backed cron (`cron/store/sqlite-cron-store.ts` 1816 LOC, `cron/runner/cron-runner.ts`) — agents runnable on schedule, rare in corpus.
- Crash/recovery: hub event journal (`hub/server/hub-event-log.ts`), resume tested (`apps/vscode/src/core/hooks/__tests__/taskresume.test.ts` 681 LOC).
- Deduction: no per-run budget ceilings found.

### interop — 8
- MCP client with config loading (`extensions/mcp/client.ts`, `config-loader.ts`); published SDK (`sdk-publish.yml`) so rival harnesses can import core; four surfaces: VS Code, JetBrains (`ext-jb-test-integration.yml`), CLI, hub/cloud.
- Plugin system (`extensions/agent-plugin/loader.ts`), skills support (`apps/vscode/src/core/context/instructions/.../skills`).
- Deduction: no ACP; no evidence of importing rival rule files.

### operability — 8
- Checkpoint/restore: git-stash snapshots with per-session scratch index and 14-day GC (`hooks/checkpoint-hooks.ts:12-19`), diff + restore services (`session/checkpoint-diff.ts`, `session/checkpoint-restore.ts`), session versioning service (`session/services/../session-versioning-service.ts`).
- State management + remote config (`apps/vscode/src/core/storage/StateManager.ts` 788 LOC, `core/src/remote-config/`).

### originality — 7
- Durable agent scheduling (cron store + runner) and hub/cloud control plane are ideas most harnesses lack; plan-guard-as-hook-before-approval is a neat enforcement placement.
- Verified in code, but the pieces (teams, MCP, checkpoints) each exist elsewhere; no codex-level singular artifact.

### durability — 9
- 350 contributors / 7449 commits (full history), funded company, 18 release/test workflows incl. nightly and desktop channels, Apache-2.0.

### docs-dx — 8
- Full mintlify doc site in-repo (`docs/docs.json`), feature references for auto-approve/auto-compact/subagents/scheduling (`docs/features/auto-compact.mdx`, `docs/cli/scheduling.mdx`), CONTRIBUTING + SECURITY.
- Deduction: mid-migration confusion between legacy extension core and sdk paths is a real onboarding tax (both trees live).

## Verdict
**Strongest: verification / orchestration / interop / operability (8).** Weakest: safety-enforcement (6) — approvals bind and are tested, but nothing under them.
Weighted 76.5 -> **B** (top of band). Dispatcher expected A; strict evidence does not support 78: no sandbox (safety 6), no cache discipline (token 7), evals not in CI (verification 8 not 9). Boundary note: this is the reference for "B ceiling = great engineering culture, missing an enforcement layer."
