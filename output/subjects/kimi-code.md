# kimi-code — T2 deep review

Subject: /data/samples/agents/kimi-code (Kimi Code CLI, MoonshotAI, MIT, TypeScript pnpm monorepo)
Census row: 376,584 non-test / 357,083 test LOC; shallow clone (1 commit, head 2026-09-24); license MIT; provenance "original" with vendored pi-tui note.

## Anchor question

Which anchor subject is this closer to, and why? **Cline (76.5, B ceiling)** — same overall profile: broad interop surface (IDE + SDK + ACP + server), approvals that bind in-loop and are tested but have nothing underneath, a massive real test corps capped by no evals/fuzzing, and docking factors from mid-flight tree migration — but kimi-code's loop rigor, cache discipline, and state model lean codex-ward, which is why it sits at the very top of B rather than mid-B.

## Census sanity

- TS code (incl. tests, excl. node_modules/dist): 666,784 lines by cloc; my per-dir split gives test `.test/.spec.ts(x)` ≈ 357k (matches census 357,083) and non-test TS ≈ 345k; census 376,584 plausibly adds JS/Vue/other. Census LOC: sane, no correction.
- `contributors: 1, commits: 1` are shallow-clone artifacts, not real. `.git/shallow` present; head_date is 5 days old so activity is fresh at HEAD; no dead/low-activity claims either direction (per appendix).
- Provenance flag "original" verified: only vendored piece is `packages/pi-tui` (38.4k LOC incl. tests), which carries `UPSTREAM.md` pinning `earendil-works/pi` subtree `packages/tui` at commit `53816d7…` (2026-09-14) with an intent-card fork-governance process. Both MIT. This is a documented library vendor, not a sync-fork of pi the product; rule (a) does not apply.
- Identity hygiene: kimi-cli (Python, separate repo) — no facts imported from it. No relation to pi as an agent product beyond the TUI library.

## What I read (line-level, T2 mandatory)

- Core loop: `packages/agent-core-v2/src/agent/loop/loop.ts` (interface: prompt states, steering, quiescence, error-handler chain, `buildAttachBundle`), `loop/loopService.ts:965-1035` (step gate: afterChain ordering, maxSteps enforcement with a user-actionable error at loop.ts:57-63, hooks, abort plumbing), `loop/machine/engine.ts` (xstate actor as pure turn/tool state machine, event projection, attach bundle for engine re-creation).
- Compaction/context: `fullCompaction/strategy.ts:17-27` (trigger 0.85 / block 0.85 / reserved 50k, overflow-reduction), `fullCompactionService.ts:500-531` (block/compact checks against `tokenCountWithPending`, per-turn compaction caps), `:844-849` (TODO-list re-injection post-summary), `contextProjector/projection.ts:8-18` (9 anomaly classes repaired: reordered/synthesized tool results, dropped duplicates/orphans, merged assistants), `tokenCounting/configSection.ts:16,43` (`measured+estimated` default: provider-reported usage for measured messages, estimates for pending).
- Permissions: `permissionPolicy/permissionPolicyService.ts:39-57` (13-policy first-match chain), `permissionGate/permissionGateService.ts:30,40-63` (binds on executor `onBeforeExecuteTool`, veto/approve/ask with telemetry per decision), `permissionRules/matchesRule.ts` (picomatch tool patterns + tool-owned `matchesRule(argPattern)`), `permissionPolicy/policies/dangerous-command-ask.ts` (tree-sitter-bash AST walk, `MAX_NESTED_SHELL_DEPTH = 4`, `UNSAFE_OPERAND` bail-out, privilege-wrapper handling), `policies/sensitive-file-access-ask.ts` + `tool/path-access.ts:51+` (sensitive basename/prefix/dot-variant matching), `permissionMode/permissionModeOps.ts:23` (default mode `manual`), workspace trust gating project MCP servers (`workspace/workspaceTrust/`, `app/mcpRegistry/mcpRegistryService.ts:12`, TUI `dialogs/trust-prompt.ts:28-42`).
- Tests/CI: `test/harness/scripted-generate.ts` (faux streaming provider driving the loop), `test/agent/loop/loop.test.ts` (61 `it(`), `test/agent/fullCompaction/fullCompaction.test.ts` (95 `it(`), `test/agent/permissionGate/permissionGate.test.ts`, `test/agent/toolApproval/toolApproval.test.ts` (22), `test/wire/resume.test.ts` + `test/wire/migration/`; `.github/workflows/ci.yml` (build+bundle smoke :29, 5-shard vitest :36-49, pi-tui own runner :53+, vscode legacy-v1-engine job :70+, **Windows job disabled `if: false` :90-93**, lint, tsgo typecheck).
- Orchestration/misc: `agent/task/taskService.ts:1553-1554,1614` (subagent resume wording incl. the agent_id-vs-source_id footgun guard), `features/swarm/` (swarm mode events), `features/cron/cronService.ts:41-44` (`cron_scheduled/fired/missed/deleted` event-sourced states), `agent/toolDedupe/toolDedupeService.ts:63-66,536-550` (streak 3/5/8 reminders, 12 = force-stop), `features/btw/btw.ts` (side-channel read-only agent), `features/fileHistory/fileHistoryService.ts:394-403` (sha256 content-addressed per-turn backups), `state/eventDispatcherService.ts` (immer + replayable/undoable state keys), `wire/repair.ts:20-29` (journal repair), `packages/minidb` (pure-Node WAL DB), `runtime/runtime.ts` (remote fs/process/terminal runtime abstraction), `apps/kimi-code/src/cli/commands.ts:55,71` (`--yolo` semantics, `--output-format`), ACP package `packages/acp-server/`, skills `.agents/skills` generic roots (`features/skill/catalog/skillRoots.ts:9-11`), `agentsMdReminder/agentsMdReminderService.ts:355,363` (late-applied AGENTS.md reminders + on-disk drift detection).

## Dimension scores

### architecture — 7
Loop split is genuinely good: `loopService.ts` owns queue/steering/gates/projection while `machine/engine.ts` is a pure xstate turn/tool machine re-attachable via `buildAttachBundle`/`attachMachineEngine` (loop.ts:196-238, engine.ts:271-300); context projection, compaction, permissions, tools are separate services behind VS Code-style DI decorators/scopes. No file >5k LOC (largest non-test: TUI host `apps/kimi-code/src/tui/kimi-tui.ts` 4,238 — under the errata threshold). Docked to 7 like cline for migration debt: dual v1/v2 engines shipped together (`KIMI_CODE_LEGACY_FLAG` CI job ci.yml:70-88, `packages/migration-legacy`, `sessionLegacy`), and `loopService.ts` at 2,285 LOC still fuses queue, gate, event projection (projectMachineEvent :1275) and turn bookkeeping.

### verification — 8
357k LOC tests, ~16,700 `it(` calls; tests assert behavior, not existence: permission gate veto/approve/ask tested through the executor hook (permissionGate.test.ts), approval flow with recorded telemetry (toolApproval.test.ts), 95 compaction cases, loop tests driven by a scripted streaming provider (test/harness/scripted-generate.ts — faux-provider rung). CI runs build+bundle-smoke+5-shard vitest+legacy-engine vscode job+lint+tsgo on every PR (ci.yml:13-160). Ceiling holds per errata: no in-CI model evals, no fuzzing; plus the Windows matrix job is disabled (ci.yml:93 `if: false`). Not 9.

### safety-enforcement — 6
The "tested approval, nothing underneath" rung, executed well: gate binds before every tool execution and can veto (permissionGateService.ts:30,55-61), default mode is manual (permissionModeOps.ts:23), policy chain includes grammar-based dangerous-command ask, sensitive-file ask, git-control-path ask, user allow/ask/deny rules, session-scoped approvals; workspace-trust gates project MCP servers (mcpRegistryService.ts:12). But there is **zero OS-level enforcement anywhere** — no seatbelt/bwrap/landlock/seccomp hits in any src tree; once approved, everything runs as the user. Nuance: `nonInteractive` drops the dangerous-command ask from the chain (permissionPolicyService.ts:42-44), so headless runs with user allow-rules lose that extra guard (fallback-ask still exists, and `--prompt`+`--yolo` are mutually exclusive, options.ts:79-80).

### token-economy — 8
Trigger/block ratios + reserved context + overflow-reduction with min-reduction floor (strategy.ts:17-27; fullCompactionService.ts:500-531); token accounting is `measured+estimated` — provider-reported usage for measured messages, estimates only for pending (configSection.ts:16,43), i.e., close to pi's "measure what the model receives". Cache discipline is real and mostly automatic at the wire layer: Anthropic `cache_control` on last block/tools/system (anthropic/format.ts:53-60,230-246), `prompt_cache_key` for OpenAI-family and Kimi (openai/format.ts:62, llm-kimi/trait.ts:72), a prompt-cache probe (usage/cacheProbeService.ts:40), and the btw side-channel deliberately keeps disabled tool definitions visible to preserve the cache (btw.ts:17). Tool-result truncation, media strip/degrade (contextProjectorService.ts:62-65), TODO re-injection after compaction. Single compaction tier (no codex-style no-LLM budget window), so 8 not 9.

### orchestration — 8
Prompt queue with steering and cancellation as first-class loop primitives (loop.ts:150-238), quiescence acquisition, registered error-handler chains; subagents are resumable with explicit crash-recovery guidance in the tool output itself (taskService.ts:1148-1207,1553-1554); swarm mode as enter/exit events; durable event-sourced cron that models `cron_missed` (cronService.ts:43) and jittered next-runs; loop breaker is graduated: reminders at streak 3/5/8, force-stop at 12 (toolDedupeService.ts:63-66,536-550, tested in test/agent/toolDedupe). Below codex's 9: no queue-as-CLI/daemon plane, budgets are just `max_steps_per_turn`.

### interop — 8
Full ACP server package (acp-server incl. terminal/fs bridges), MCP client plus conversational `/mcp-config`, published TS SDK (`@moonshot-ai/kimi-code-sdk`) and contract-driven klient with byte-identical memory/ipc transports, WS server (kap-server) with a documented server-api, VS Code extension, headless `-p` with `--output-format` (commands.ts:71), remote-control, plugin marketplace, generic `.agents/skills` discovery (skillRoots.ts:9-11). No evidence it *exposes* itself as an MCP server; cline-tier 8.

### operability — 8
Wire journal with repair (`wire/repair.ts:20`) and migration-tested resume (test/wire/resume.test.ts, test/wire/migration); replayable+undoable state keys make conversation undo a state fold, not an afterthought (state/eventDispatcherService.ts:564-594); file-history checkpoints with sha256 content-addressed backups per turn (fileHistoryService.ts:394-403); agent fork (agentLifecycleService.ts:410); a dedicated debug app (apps/kimi-inspect) and debugEvents feature; config errors carry fix instructions (loop.ts:57-63); the embedded DB itself truncates crash-torn tails (minidb README:20-21).

### originality — 8
Real, code-verified ideas rarely (or never) seen elsewhere in the corpus: pure-Node embedded WAL datastore with skip-list indexes and a RESP server (packages/minidb, DESIGN_NOTES.md); intent-card fork governance for a vendored upstream (pi-tui/UPSTREAM.md — a genuinely novel answer to "how do we carry a fork"); event-sourced agent state with repair migrations; grammar-parsed (tree-sitter) bash for permission decisions instead of regex — the corpus anti-pattern `regex-denylist` inverted; graduated force-stop loop breaker; btw cache-preserving side-channel; late-arriving AGENTS.md reminders with drift detection; a Runtime abstraction that decouples agent from fs/process/terminal host. Not 9-10: no single field-defining mechanism at codex's depth.

### durability — 8
Institutional backing (Moonshot AI), MIT, changesets+release CI gated on repo owner, 9 workflows, nix flake, SECURITY.md with GHSA channel, active head 5 days old. Bus factor unverifiable from a 1-commit shallow clone; contributors-breadth claim not asserted either way.

### docs-dx — 8
Bilingual in-repo docs tree (58 md; en/zh) matching shipped behavior on sampled points, config reference with TOML sections, ACP/server-api references, install-one-liner with no Node required, honest flag help ("routine edits run automatically; risky actions still ask", commands.ts:55), error messages that name the fix (loop.ts:57-63).

## Totals

| dimension | score | weight | weighted |
|---|---|---|---|
| architecture | 7 | 15 | 105 |
| verification | 8 | 15 | 120 |
| safety-enforcement | 6 | 10 | 60 |
| token-economy | 8 | 10 | 80 |
| orchestration | 8 | 10 | 80 |
| interop | 8 | 10 | 80 |
| operability | 8 | 10 | 80 |
| originality | 8 | 10 | 80 |
| durability | 8 | 5 | 40 |
| docs-dx | 8 | 5 | 40 |

**Total 76.5 → band B (upper end). Boundary-risk: 1.5 below the 78 A line; if synthesis re-scores architecture at 8 (cline precedent for dual-tree) this is 78/A.** Strongest dimension: originality. Weakest: safety-enforcement.

## Provenance / calibration notes

- Provenance: original; one documented vendored fork (pi-tui from earendil-works/pi@53816d7, MIT, intent-card diffs). Rule (a) N/A — kimi-code is not a derivative of pi the agent; it vendored one UI library with exemplary attribution. For reference, the score is below pi's 78.5 anyway.
- No license risk (MIT everywhere incl. vendored pkg). No demotions recorded.
- Windows test matrix disabled (ci.yml:93) — kept inside the verification-8 band rather than docking, since the sharded suite runs on ubuntu for every PR.
