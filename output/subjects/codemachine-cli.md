# codemachine-cli -- T1 review

**Anchor question:** Closer to **codel** than to any working anchor on the dimensions that gate
bands -- verification is zero and safety is *negative-in-kind* (it strips the sandboxes its
engines ship) -- but unlike codel the machinery is real (guarded FSM, crash-recovery detection,
an MCP step-transition protocol), so it is crush's orchestration ambition with codel's
verification profile; the weighted score lands just below the C floor.

## What it is

A meta-orchestrator, not a harness. It runs no LLM loop of its own: it spawns rival coding-agent
CLIs (`claude`, `codex`, `cursor`, `auggie`, `mistral`, `opencode`, `ccr`) in headless mode and
drives them through user-defined multi-step workflows. TypeScript, 55,434 LOC in `src` (499
files), Bun + SolidJS TUI, Apache-2.0, manifest name `codemachine` (matches census; no rename
suspicion). Census non_test 48,363 vs my 55,434 for `*.ts,*.tsx` -- same order, no vendored-blob
distortion.

## Census sanity

- `test_loc: 0` -- **confirmed**. `find` for `__tests__`/`test`/`tests` dirs and `*test*`/`*spec*`
  files in `src` returns nothing anywhere in the tree. The `bun test` script
  (package.json:`test`) runs an empty suite.
- Shallow clone (1 commit, HEAD 572def6). Remote-verified per appendix: GitHub API
  `pushed_at 2026-02-25T21:09:41Z` matches HEAD exactly; not archived; 2,511 stars, 1 contributor.
  ~7 months dormant as of study date -- stale, but not >12mo, so no dead cap.

## Core loop / state model

The "agent loop" is outsourced; the subject's loop is a workflow FSM.
- FSM with typed events, guard-first transition selection, exit/action/enter hooks:
  `src/workflows/state/machine.ts:31-90` (`findTransition` picks first passing guard :50-54;
  final states refuse events :58-61).
- Runner core routes by resolved scenario (interactive x autoMode x chained-prompts):
  `src/workflows/runner/core.ts:62-76` (`buildScenario`), auto-mode continuation prompt with
  mode-switch abort handling :82-140.
- Agent execution: `src/agents/runner/runner.ts:186+` (`executeAgent`) -- engine resolution with
  5-min TTL auth cache :20-48 (explicit fix for "5-minute delay bug" :17-19), engine fallback
  chain :247-296, resume via stored sessionId :212-227, monitoring registration :330-395.
- Boundaries are genuinely clean: `infra/engines|process|mcp` / `workflows` / `agents` /
  `shared` / `cli`; largest file is 849 LOC (`useLogStream.ts`). No god files.
- Docking: implicit cross-module signaling through the global process EventEmitter
  (`process.on('workflow:mode-change')`, runner/core.ts:112) and a shared directive file --
  no contract on who writes what.

## Permission / sandbox code (mandatory read)

There is no permission or sandbox layer of the subject's own, and every adapter **disables** the
protections the underlying engine provides:
- claude: `--dangerously-skip-permissions --permission-mode bypassPermissions`
  (`src/infra/engines/providers/claude/execution/commands.ts:54-56`; identical in ccr
  `ccr/execution/commands.ts:55-57`).
- codex: `--sandbox danger-full-access --dangerously-bypass-approvals-and-sandbox`
  (`src/infra/engines/providers/codex/execution/commands.ts:22-24`).
- mistral: `--auto-approve` (`mistral/execution/commands.ts:62`).
No flag, config key, or code path re-enables any of it (grep across `src` for
permission/approval finds only these bypasses plus workflow-progression gates). The TUI's
"approval" surfaces (checkpoint modal, `signals/mcp/controller.ts:29-37`
approved|rejected|revision_needed) gate *workflow progression*, not tool execution -- a prompt
injection inside any engine session runs with full bypassed access across all seven engines.
This is below the codel rung (2), which at least container-spawned flows: codel misleads about
isolation, codemachine removes isolation it could have inherited and does not claim it exists.

## Compaction / context management (mandatory read)

None. Grep for `compact` across `src` finds zero context-management code; nothing observes or
reacts to context limits -- each engine self-manages its own context and codemachine never
measures it. Partial mitigations that exist: per-step context is built from artifacts/templates
rather than full history (runner.ts:163-177 comment "Prompt building is the caller's
responsibility"; merged MCP tool filtering per agent/step :303-314), and cost/token telemetry is
persisted (`src/agents/monitoring/db/schema.ts:29-34`: tokens_in/out, cached, cache_read, cost)
and surfaced (`src/cli/tui/routes/workflow/adapters/headless.ts:114` `cost:$...`). Cost
visibility without any context discipline: the crush/nanocoder 6-rung ("one standard
auto-summarize") is not reached because there is no summarization at all; it sits above codel's 1
only because cost telemetry and artifact-scoped step prompts exist.

## Orchestration (the product's best surface)

- Crash recovery: step needs recovery iff sessionId present and completedAt absent
  (`src/workflows/recovery/detect.ts:22-36`), restore replays via engine `resume sessionId` +
  continuation prompt and completed-chains tracking (runner/resume path :212-227, core.ts:20-29
  paused-state sync).
- Directives control plane: agents steer the workflow by writing
  `.codemachine/memory/directive.json` (`src/workflows/directives/reader.ts:13`); evaluators turn
  `loop` into step-back repetition with a maxIterations ceiling
  (`directives/loop/evaluator.ts:30-46`) and `checkpoint` into a workflow stop
  (`checkpoint/evaluator.ts:26-31`).
- Inter-agent transition protocol over MCP: Agent A calls `propose_step_completion` with
  artifact path + optional `sha256:` hash + success-criteria checklist validated against JSON
  Schema (`src/infra/mcp/servers/workflow-signals/tools.ts:11-53`), the controller waits for
  Agent B's `approve_step_transition` and returns approved/rejected/revision_needed
  (`src/workflows/signals/mcp/controller.ts:1-37`). Nothing else in the corpus makes
  propose->validate->approve between two engine sessions the core primitive.
- Parallel multi-agent execution via coordinator (`src/agents/coordinator/service.ts:23+`).
- No budgets of any kind (no money or step caps at workflow level; maxIterations is the only
  bound); crash-recovery semantics unproven (nothing runs them).

## Verification

Zero. No test files at all, no PR/CI workflow: `.github/workflows/` contains only
`build.yml` (triggers: `workflow_dispatch` + tag push `v*`, build.yml:3-7) and `publish.yml`
(dispatch/release). Nothing runs lint, typecheck, or tests on push. Husky is in devDeps with a
`prepare` script but no `.husky/` directory exists. The `bun test` script is a ceremony around
an empty suite. This is the codel 0-rung; the 2-rung ("present but misleading") would require
tests that don't prove anything -- here there are none, so 0 with CI existence keeping it from
being worse.

## Interop

Rival-harness import taken to its limit: the entire product is seven headless-engine adapters
(the `harness-adapter-contract` shape seen in aeon, but with no capability manifest and no
contract tests). Publishes two MCP servers consumable by external Claude Code/Codex sessions
(`src/infra/mcp/servers/agent-coordination/index.ts:3-8`), plus per-agent/step MCP tool
filtering via written context files (runner.ts:303-314). Headless adapter for scripted runs
(`cli/tui/routes/workflow/adapters/headless.ts`). No ACP, no IDE surface, no published SDK.

## Operability / durability / docs

SQLite monitoring DB + per-agent log files + registry/status services, agent-level and
workflow-level resume, OTel traces/metrics/logs with a `docker/observability` stack -- a real
operability story, entirely unexercised. Errors are fire-and-forget-logged
(`.catch(err => error(...))`, runner.ts:419-421, 448-451); debug log to `~/.codemachine/logs`.
Durability: solo author, 7 months stale, no SECURITY.md, no governance docs; 2.5k stars and an
external docs site keep it off the dead pile. Docs: no in-repo documentation at all (no `docs/`);
README is marketing pointing at docs.codemachine.co; TUI onboarding wizard is the real onboarding
(`src/cli/tui/routes/onboard/`).

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 6 | guarded FSM `state/machine.ts:31-90`; clean layers, max file 849 LOC; docked for outsourced loop + global-EE/directive-file coupling `runner/core.ts:112` |
| verification | 0 | zero test files (find, whole tree); CI is tag-build-only `build.yml:3-7`; empty `bun test` ceremony `package.json` |
| safety-enforcement | 1 | bypass flags in 4 of 7 adapters: claude `commands.ts:54-56`, codex `:22-24`, ccr `:55-57`, mistral `:62`; no own layer |
| token-economy | 2 | no compaction anywhere; cost telemetry only: `db/schema.ts:29-34`, `headless.ts:114` |
| orchestration | 7 | recovery `detect.ts:22-36`; loop/checkpoint directives `loop/evaluator.ts:30-46`; MCP propose/approve `controller.ts:1-37` -- no budgets, unproven recovery |
| interop | 7 | 7-engine adapter matrix; MCP servers out `agent-coordination/index.ts:3-8`; tool filtering `runner.ts:303-314`; no ACP/IDE/SDK |
| operability | 6 | sqlite monitoring + resume + OTel stack; swallowed async errors `runner.ts:419-421`; recovery unexercised |
| originality | 7 | agent-written directive.json control plane `reader.ts:13`; inter-agent MCP transition gate `tools.ts:11-53` -- real in code |
| durability | 3 | 1 contributor; pushed_at 2026-02-25 remote-verified; no SECURITY.md |
| docs-dx | 4 | zero in-repo docs; external-site-only README; TUI wizard onboarding |

**Weighted total: 42.5 -> band D.** Strongest: orchestration/interop/originality (7, three-way).
Weakest: verification (0).

Calibration notes: no fork (census `original`, GitHub `fork:false` confirmed). No license
restriction (Apache-2.0), though nothing here is code-adjacent portable anyway except adapter
concepts. No demotion applied; D emerges from the rubric, not a rule. Close to the 45 C-floor:
if synthesis weighs the unproven-but-designed recovery story heavier, orchestration could argue
to 8 (+1 -> 43.5, still D); reaching C would need verification >= 1 (there is literally nothing
to count).
