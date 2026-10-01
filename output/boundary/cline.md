# Boundary re-review: cline (independent, full rubric)

Subject: `/data/samples/agents/cline`, HEAD `29896ec` (2026-09-25, 4 days old, not shallow).
Scale check: ~538k non-test LOC in sdk+apps (excl. generated `catalog.generated.ts` 193k) + ~255k `*.test.ts`. T3-sized; deep read performed. Independence: did not read anchors/cline.md, anchors/scores-cline.json, or scores/cline.json.

## Closest anchor (one sentence)

Closest to cline's own frozen rung (76.5, B ceiling) with pi (78.5) the nearest neighbor above: this snapshot has gained real interop ground on pi (ACP server, rival-harness session import) but still shares pi's "tested approval, nothing underneath" safety ceiling while lacking pi's cache-warming and session-tree depth, so it does not cross 78.

## Architecture — 8

- SDK-first monorepo: `sdk/packages/{agents,core,llms,shared,sdk,ui}`; the loop lives in one place, `sdk/packages/agents/src/agent-runtime.ts` (2825 LOC) consumed by thin hosts (vscode, cli, hub).
- Loop read: `agent-runtime.ts:811-964` — clean `while (maxIterations)` turn cycle: per-turn error-slate reset (`:824`), provider-retry wrapper, max-tokens recovery branch (`:880-890`), empty-response guard distinguishing provider-executed tool activity (`:843-860`), completion-reminder continuation (`:907-914`), terminal-tool completion (`:946-958`). State model (`state.messages`, `pendingToolCalls`, usage deltas) is explicit and event-emitted per step.
- Docked for the dual-tree migration still in flight: legacy `apps/vscode/src/core` retains 21.5k non-test LOC (controller, locks, storage) and the adapter layer includes `apps/vscode/src/sdk/message-translator.ts` at 2877 LOC; other 2.7-3k hosts: `sdk/packages/core/src/runtime/host/local-runtime-host.ts` (2748), `sdk/packages/core/src/cloud/controller.ts` (3016), `sdk/packages/core/src/hub/runtime-host/hub-runtime-host.ts` (2228).
- Same verdict as the frozen 8 rung.

## Verification — 8

- 255,153 LOC in `*.test.ts` across sdk+apps. Semantics-level, not existence-level: approval-iff-policy asserted with a scripted model that inspects the tool-result message (`sdk/packages/agents/src/agent-runtime.test.ts:2354-2385` "requests approval when a tool policy disables auto-approval" — asserts `isError` + rejection suffix inside the next request).
- CI on PRs: `.github/workflows/sdk-test.yml:1-120` — typecheck + lint + full vitest on ubuntu+windows matrix, sidecar tests, TUI e2e, publishable-package check; `ext-vscode-test.yml:126-236` — quality checks + vitest under xvfb with coverage on ubuntu+windows; JetBrains integration `ext-jb-test-integration.yml`.
- **Evals NOT in CI — verified.** `grep -rn 'evals' .github/workflows/` = zero hits; `evals/README.md:4` states the `cline-evals-regression.yml` workflow is off pending re-point at the new SDK CLI. `evals/cline-bench` (12 production bug-fix tasks) + `analysis/` pass@k/pass^k framework exist but are unwired. No fuzzing anywhere (grep `fuzz` in .github + root configs = 0).
- 8 is the ceiling rung: tested semantics but no in-CI evals and no fuzzing, identical gap to the anchor's stated 8-ceiling.

## Safety-enforcement — 6

- **Phase 1's "no OS sandbox" claim: VERIFIED.** Case-insensitive grep for `seatbelt|bwrap|bubblewrap|firejail|landlock|seccomp` across all non-test `.ts` in sdk+apps returns zero enforcement code; the only namespace hit is container *detection* in `apps/cli/src/commands/doctor.ts:115-124` (diagnostic, not enforcement).
- What binds: tool policy + approval in the loop, fail-closed — missing approval callback returns `approved:false` (`agent-runtime.ts:2416-2421`), approval-callback exception also denies (`:2431-2436`); tested (`agent-runtime.test.ts:2354`). Plan-mode command blacklist runs as a `beforeTool` hook *before* policy and approval (`sdk/packages/core/src/extensions/tools/command-guard-extension.ts:11-13`), with unusually honest limits doc (`command-guard.ts:9-16`: "simple blacklist, not a shell interpreter... will not catch... `python -c "open(..., 'w')"`").
- Two soft spots below the 6 rung's best case: (1) workspace-scoped hook directories `.cline/Hooks` are on the subprocess-hook search path with no trust gate (`sdk/packages/shared/src/storage/paths.ts:494-498`; `sdk/packages/core/src/hooks/subprocess-runner.ts` spawns them; no `trust` identifier anywhere in `hooks/`) — cloned-repo hook execution; (2) headless cron runs set `{"*": {autoApprove: true}}` and default mode `yolo` (`sdk/packages/core/src/cron/runner/cron-runner.ts:88-93`, `sdk/packages/core/src/cron/specs/cron-spec-parser.ts:401`) — deliberate ("cannot wait for a human") but it is unattended full tool access.
- Honesty: `SECURITY.md` is a Bugcrowd VDP boilerplate with zero discussion of what the approval system does not protect against — unlike the anchor's exemplar honesty rung. 6 stands, same as "tested approval, nothing underneath".

## Token-economy — 7

- One threshold, two strategies: `COMPACTION_TRIGGER_RATIO = 0.9` (`sdk/packages/core/src/extensions/context/compaction-shared.ts:17`), `agentic` (LLM summary) vs `basic` (deterministic) behind it (`compaction.ts:296-301`, `basic-compaction.ts`); trigger uses the provider's actual previous input count, not the estimate (`compaction.ts:359-362`).
- Best idea in the module: `underestimateFactor` — when the provider's measured input already exceeds the estimate for the (larger) transcript, the whole budget is scaled down, capped and never loosening (`compaction.ts:342-354` with a genuinely careful comment). Budget projection carries trigger/target/overhead/utilization (`compaction.ts:412-424`).
- Imported-history fold compaction on resume of foreign sessions (`compaction.ts:692-732`).
- Cache: wire-level ephemeral breakpoints ARE plumbed per dialect (`sdk/packages/llms/src/providers/routing/anthropic-compatible.ts:90-106`, `bedrock-cache-point.ts:56`, `generic-compatible.ts:58`) and cache read/write tokens are tracked in usage (`agent-runtime.ts:444-469`) — but there is no *strategy* layer: no warming, no cache-aware breakpoint placement, no monotonic-compaction discipline. Docs claim summarization reuses the prompt cache (`docs/features/auto-compact.mdx:43-45`). This exceeds the frozen note "no cache discipline" but does not reach pi's 8 rung (projected trigger + active warming). 7.

## Orchestration — 8

- Teams-as-tools: `sdk/packages/core/src/extensions/tools/team/multi-agent.ts` (1943) + `team-tools.ts` (916) with 991 LOC of behavior tests (`team-tools.test.ts`); `spawn-agent-tool.ts` (204).
- Durable scheduling: `sdk/packages/core/src/cron/store/sqlite-cron-store.ts` (1816) with `resumeSchedule` (`sdk/packages/core/src/cron/service/schedule-service.ts:369`); hub event journal `sdk/packages/core/src/hub/server/hub-event-log.ts` (cursor-monotonic, `:191`).
- Two independent runaway breakers: `LoopDetectionTracker` (consecutive-identical soft warning + hard escalation, `sdk/packages/core/src/runtime/safety/loop-detection.ts:76-85`) and `MistakeTracker` (`.../runtime/safety/mistake-tracker.ts`), wiring tested at `session-runtime-orchestrator.test.ts:2451` — converges with crush's loop_detection concept.
- Gaps vs codex's 9: no cost budgets for subagents/cron (grep `budget` in cron dir: no cost/token matches); queue is cron-shaped, not a first-class job queue.

## Interop — 9

- **The frozen anchor note "no ACP" is stale for this snapshot.** ACP server is real: `apps/cli/src/acp/acpAgent.ts` (957 LOC against `@agentclientprotocol/sdk`, `:25`), wired via `--acp` (`apps/cli/src/commands/program.ts:64,138`), stdio ndjson (`acp/index.ts:10-25`), standard allow_once/allow_always/reject_once permission options (`acp/permissions.ts:13-32`), plus session-load/updates tests (`acp/session-load.test.ts` 341 LOC).
- Rival-harness session import — convert-and-resume from Claude Code / Codex / opencode transcripts (`sdk/packages/core/src/services/session-import/{claude-code,codex,opencode}.ts`, dispatcher `types.ts:5-14`, sanitize + 1194 LOC test suite), paired with the foreign-tool-schema fold on first resumed turn (`compaction.ts:692-732`). No other studied subject does this; it is copy-me.
- MCP client with OAuth, policies, remote proxy (`sdk/packages/core/src/extensions/mcp/`); published `@cline/*` SDK (`sdk-publish.yml`); four surfaces: VS Code, JetBrains (CI-gated `ext-jb-test-integration.yml`), CLI, hub/desktop.
- One rung below 9 would be fair if anything — missing MCP *server* exposure vs codex's both-sides. Held at 9 on the import capability; flag this as the dimension most likely to draw challenge.

## Operability — 8

- Git-stash checkpoints extended with an untracked-files third parent (`sdk/packages/core/src/hooks/checkpoint-hooks.ts:245,341-362`) and idle-based GC (`retainCheckpointRefs`, `:16`); restore is a worktree transaction (`sdk/packages/core/src/session/session-versioning-service.ts:11-19` beginWorktreeRestoreTransaction / createCheckpointRestorePlan); exposed through `ClineCore.restoreSession` (`sdk/packages/core/src/ClineCore.ts:577`, `hub-runtime-host.ts:968`).
- Session persistence sqlite (`services/storage/sqlite-session-store.ts`); CLI resume by id (`apps/cli/src/commands/program.ts:43`); `cline doctor` inspects hub discovery, connectors, containerized-namespace identity (`apps/cli/src/commands/doctor.ts`).
- Not pi's 9: no session tree / fork / branch navigation visible.

## Originality — 6

- Genuinely unique in corpus: rival-harness import + imported-history compaction policy; untracked-parent stash checkpoints; `underestimateFactor` estimator self-correction; plan-mode command guard with a model-behavior rationale (`command-guard.ts:5-7`).
- Convergent elsewhere: loop breaker (crush), ACP (nanocoder), teams-as-tools common. Core loop itself is a standard well-executed shape. 6; the frozen anchor ladder never displayed a cline originality rung, my read places it just above nanocoder's 6 for the import capability.

## Durability — 9

- 350 authors, 7449 commits (`git shortlog -sn HEAD`, `git rev-list --count HEAD`), HEAD 4 days old; 18 workflows incl. multi-channel releases (vscode stable/nightly/legacy, sdk, ui, desktop, cli publish); Bugcrowd VDP (`SECURITY.md:11`). Institutional backing intact.

## Docs-dx — 8

- 116 md/mdx in a mintlify tree (`docs/docs.json`) covering features incl. `docs/features/auto-compact.mdx` and `docs/features/auto-approve.mdx`, `docs/sdk`, `docs/troubleshooting`; in-repo `sdk/ARCHITECTURE.md`, `.agents/skills/` reference packs. Docked for dual-tree confusion (which tree is authoritative is not stated up front).

## Totals

| dim | score | weight | contrib |
|---|---|---|---|
| architecture | 8 | 15 | 12.0 |
| verification | 8 | 15 | 12.0 |
| safety-enforcement | 6 | 10 | 6.0 |
| token-economy | 7 | 10 | 7.0 |
| orchestration | 8 | 10 | 8.0 |
| interop | 9 | 10 | 9.0 |
| operability | 8 | 10 | 8.0 |
| originality | 6 | 10 | 6.0 |
| durability | 9 | 5 | 4.5 |
| docs-dx | 8 | 5 | 4.0 |

**weighted_total = 76.5 → band B.**

Even the most generous defensible variant (token 8, originality 7) reaches 78.5 only by doubling credit for the same import feature across two dimensions; the honest ceiling for this snapshot remains band-B top. The A-band gap is structural: no OS sandbox (safety 6 is the floor this architecture supports) and evals-unwired CI.

## Where this review diverges from the frozen anchor observations (not scores, facts)

1. ACP now exists in the CLI (`apps/cli/src/acp/acpAgent.ts`) — anchor interop note "no ACP" is out of date at this HEAD.
2. Cache discipline is not "none" — wire-level ephemeral breakpoints exist per dialect (`anthropic-compatible.ts:90-106`); what is missing is the strategy layer.
3. Cline has its own loop breaker (`loop-detection.ts`) — anchor called crush "only anchor with one"; true among anchors' rungs displayed, but cline now converges (soft/hard thresholds, tested).
