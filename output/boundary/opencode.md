# Boundary re-review: opencode (independent)

Provisional 77.0 (B, band-sensitive per integrator). Independent re-score below is code-level,
built without reading subjects/opencode.md, scores/opencode.json, or the T3 run dir.
Shallow clone (1 commit, HEAD 2026-09-26, v1.18.32): activity is current, but commit-volume
claims are evidence-limited per protocol rule 2.

**Anchor proximity (protocol §6):** closest to **cline** -- same shape (large TS monorepo,
server-first core with thin TUI/web/desktop hosts, mid-flight dual-tree migration,
approval-only safety with no sandbox, tests in CI on two OSes), and it inherits cline's exact
docking points. opencode clears cline on interop (ACP, which cline lacks) and on an
in-repo CI'd route-contract gate; it stays below pi's architectural minimalism.

## Which path was scored: V1 shipping loop

The repo ships **two parallel session engines**:

- **V1** = `packages/opencode/src/session/` -- the loop that serves product traffic. The HTTP
  prompt route imports it directly (`packages/opencode/src/server/routes/instance/httpapi/handlers/session.ts:11`),
  as do the CLI and GitHub handler (`src/cli/cmd/github.handler.ts:30`). Core loop:
  `runLoop` while-true at `packages/opencode/src/session/prompt.ts:1088`, stream handling in
  `processor.ts`, compaction/permission/reminders wired as services.
- **V2** = `packages/core/src/session/` -- event-sourced projector
  (`projector.ts`), context epochs (`context-epoch.ts`), runner (`runner/`), and a per-session
  run coordinator with coalesced wake (`run-coordinator.ts:6-17`). It IS wired into the
  shipping server layer graph (`server/routes/instance/httpapi/server.ts:301`) as the
  execution/admission plane, and V1 reads V2's SQL tables -- but the turn loop itself still
  runs V1 (`SessionV1` types throughout prompt.ts, `:2-3`).

I scored **V1 as the shipping path** (all traffic goes through it), crediting V2 mechanisms
(coordinator, projector, durable inputs) only where they are actually in the server graph, and
docking architecture for the dual-engine migration state. Scoring V2 as the primary loop would
be scoring vaporware: no product prompt reaches `SessionRunner` today.

## Dimensions

### architecture — 8 (w15)
Effect service-per-module discipline with explicit Layer DI (`LayerNode`): loop
(`prompt.ts:1088`), stream processor (`processor.ts:76` handle/Result contract), permission
(`permission/index.ts:67`), tools registry, LSP, storage are separate services; state is
sqlite-backed (drizzle) + an event bus with a versioned manifest (`src/event-manifest.ts`).
Dominated residual dock: two parallel loop engines with bridge layers in both directions
(`src/event-v2-bridge.ts`, `packages/core/src/v1/`, V1 importing
`core/session/execution/local`) -- the core/package boundary is currently blurred. Largest
non-generated shipping files: `provider/provider.ts` 2072, `session/prompt.ts` 1631 (loop +
title + subtasks + structured output fused), `lsp/server.ts` 1983. This matches the cline 8
rung (SDK/server-first core, thin hosts, dual-tree dock; cline's own residual was a 2825-LOC
host file). ERRATA god-file rule applied: no product-mode file >5k LOC in the agent core
(largest UI-surface file `app/src/context/server-session.ts` 1427), so no dock on either side
of that rule. Not 9: pi/codex hold one loop engine; opencode holds two mid-swap.

### verification — 8 (w15)
~130k test LOC (opencode 96k + core 35k); 399 test files in the two core packages alone.
CI evidence: `bun turbo test` on push/PR across linux+windows
(`.github/workflows/test.yml:66-68`); a route-coverage **HttpApi exerciser** gate running
coverage/auth/effect modes with `--fail-on-missing --fail-on-skip`
(`test.yml:77-80` -> `packages/opencode/package.json:11`, harness
`test/server/httpapi-exercise/index.ts:1-16` -- every public route must have a decode/auth/
mutate scenario); Playwright e2e on both OSes (`test.yml:136`); scripted SSE
`TestLLMServer` (`test/lib/llm-server.ts:637`) plus recorded-provider tests
(`test/session/llm-native-recorded.test.ts`). Tests assert policy semantics, not existence:
last-match-wins rule ordering (`test/permission-task.test.ts:60`), global-deny disables task
tool (`:87`), default-ask for doom_loop/external_directory (`test/agent/agent.test.ts:469-475`),
race specs (`test/session/snapshot-tool-race.test.ts`). ERRATA 8-ceiling honored: no in-CI
model evals and no fuzzing anywhere (grep over all 27 workflows found none) -- 9+ impossible.

### safety-enforcement — 6 (w10)
The "tested approval, nothing underneath" rung, identical to cline/pi/crush. Approvals bind
in-loop: `Permission.ask` blocks tool execution on a Deferred and deny rules hard-fail
(`permission/index.ts:73-78,98-107`); every tool asks (`tool/shell.ts:263-290`,
`tool/edit.ts:102`, `tool/write.ts:54`, ...); outside-worktree access gated by
`external_directory` containment (`tool/external-directory.ts:21-46`, used from shell scan
`tool/shell.ts:264-279`); subagents cannot shed parent denies and get task/todowrite denied
by default (`agent/subagent-permissions.ts:15-30`), spawn depth capped (`tool/task.ts:106-114`);
doom-loop detection re-enters as an approval request (`processor.ts:356-373`). No OS sandbox
-- and SECURITY.md is exemplary about it: "OpenCode does **not** sandbox the agent. The
permission system exists as a UX feature" (`SECURITY.md:15-17`), escape-table row
`:30`. `--yolo`/`--dangerously-skip-permissions` exists and is honestly labeled "dangerous!"
(`cli/cmd/run.ts:244-247`). Not 5: default is ask, no fail-open path. Not 7: nothing below
the prompt.

### token-economy — 7 (w10)
Overflow trigger = last reported usage vs. usable budget with configurable reserve
(`session/overflow.ts:8-36`); compaction has a retained-tail budget with lazy estimation
(`compaction.ts:228-267`), summary-message chaining that excludes prior summarized ranges
(`:360-370`), and an opt-in tool-output prune tier with protect/minimum thresholds
(`:28-31,273-315`). Cache discipline exceeds cline: per-provider cache-point placement
(`provider/transform.ts:364-379`) and `prompt_cache_key = sessionID`
(`transform.ts:1325,1334`); cost/tokens tracked per message (`prompt.ts:1169-1172`), stats
plane exists. Below pi's 8: the trigger measures the *previous* request's usage rather than
the projected outgoing context; no cache warming; no branch summarization; prune off by default.

### orchestration — 7 (w10)
Subagent task tool with permission derivation + depth limit (above), background-job registry
with promote/wait/cancel (`src/background/job.ts` -> `core/background-job.ts`), and the
per-session-key run coordinator with coalesced wake and interrupt-and-settle
(`core/src/session/run-coordinator.ts:6-17,49-60` -- a mid-run steering/wake primitive) wired
into the server (`httpapi/server.ts:301`). Resume/fork/continue CLI (`cli/cmd/run.ts:147-158`).
Loop-detection breaker (above) -- still rare among subjects. Not 8 (cline): no durable
queue/cron plane in the shipping path; crash recovery of in-flight turns relies on V1 mutable
rows -- V2's durable input journal is in-flight, not shipped as a turn engine.

### interop — 9 (w10)
Broadest surface stack seen in this study: MCP client with OAuth + catalogs
(`src/mcp/oauth-callback.ts`, `mcp/index.ts`), full **ACP** server implementation in the
shipping package (`src/acp/service.ts`, 1226 LOC, MCP-server pass-through
`acp/service.ts:196`), Effect HttpApi server with lazy OpenAPI schema
(`httpapi/server.ts:183-188`) and codegen'd SDKs (`httpapi-codegen` pkg; published
`@opencode-ai/sdk` v1+v2 -- `packages/sdk/js/src/v2/gen/sdk.gen.ts` 7219 LOC), headless
`run` with event streaming (`run.ts:14`), custom TUI (opentui), web app, desktop, VS Code
extension (`publish-vscode.yml`), GitHub action publishing (`publish-github-action.yml`),
plugin package (`packages/plugin`), skill system, LSP. Above cline's 8 (no ACP, 4 surfaces);
on par with codex's 9 in breadth -- opencode lacks MCP-*server* mode and a Python SDK but has
ACP which codex lacks. Held at 9, not 10: single-host instance model, no versioned protocol
guarantees like codex's app-server contract.

### operability — 8 (w10)
sqlite-backed sessions with continue/--session/--fork (`run.ts:147-158`), revert/unrevert to
message boundaries via git-object snapshots + patch reversal with diff preview
(`session/revert.ts:38-81`, `snapshot/index.ts:75-148`), export/import incl. share-URL
rehydration (`cli/cmd/import.ts:26-29`), `debug` command family incl. redacted diagnostics
(`cli/cmd/debug/redact.ts`), config migration tooling (`config/tui-migrate.ts:25,118`),
`opencode db` access. Not 9 (codex/pi): no rollout-style append-only journal on the V1 path
(V2 projector is the fix, in-flight), no fork-per-session tree navigation, crash-recovery
posture for active runs unproven.

### originality — 8 (w10)
All verified in code, not marketing: (1) confined JS interpreter as an orchestration tool over
the MCP catalog -- tree-walking Effect-native runtime with first-class promise tool calls
(`packages/codemode/src/interpreter/runtime.ts:602,1909`, tool surface
`src/tool/code-mode.ts:15-18`) -- the only code-mode runtime besides codex's v8 found so far;
(2) LLM-generated command-**arity** dictionary used to collapse shell commands into
human-scoped approval patterns (`permission/arity.ts:12-46` -- the generation prompt is
preserved in the source); (3) doom-loop -> approval (not hard-stop) at
`processor.ts:356-373`; (4) grammar-parsed (tree-sitter bash/ps) extraction of directories and
patterns for approvals (`tool/shell.ts:257-290`); (5) route-coverage exerciser as a CI gate
(`package.json:11`). Below the 9s (codex/pi) only because arity-dict and exerciser are
tactics rather than systems that reshape the field; code-mode is a second implementation of an
existing anchor idea.

### durability — 8 (w5)
Funded institution (anomaly/SST infra: `sst.config.ts`, `infra/`, AWS SSO scripts), v1.18.32,
HEAD current 2026-09-26, 27 CI workflows, Nix + bun lockfiles, SECURITY.md, 20 translated
READMEs, commercial cloud plane (`packages/console`). Shallow clone (1 commit) hides
contributor/commit volume -- history claims evidence-limited, but every independent signal
agrees on heavy active maintenance. 9 reserved for rows with verified institutional volume (codex/cline).

### docs-dx — 8 (w5)
Full in-repo docs site (`packages/web/*.mdx`, `docs.json`, essentials + ai-tools trees),
layered config with documented file precedence (`config/config.ts:141,273-274`), TUI-config
migration command, informative errors that enumerate valid values (`prompt.ts:1168-1172`
agent-not-found hint with available agents), honest SECURITY.md threat table. Not 9 (pi): no
architecture/internals docs comparable to how-pi-works; the V1/V2 engine split is undocumented
in-repo -- exactly the confusion cline was docked for.

## Totals

| dim | score | w | pts |
|---|---|---|---|
| architecture | 8 | 15 | 120 |
| verification | 8 | 15 | 120 |
| safety-enforcement | 6 | 10 | 60 |
| token-economy | 7 | 10 | 70 |
| orchestration | 7 | 10 | 70 |
| interop | 9 | 10 | 90 |
| operability | 8 | 10 | 80 |
| originality | 8 | 10 | 80 |
| durability | 8 | 5 | 40 |
| docs-dx | 8 | 5 | 40 |

**weighted_total = 77.0 -> band B** (coincidentally identical to the provisional 77.0, from an
independent pass; the integrator's three flagged lanes all land at 8 for me -- none was
under-scored, and none survives a push to 9: arch fails on dual engines, verification fails
the ERRATA evals/fuzzing ceiling, originality's best entry is a second implementation of
codex's code-mode).

A-floor verdict: **not crossed**. The single most defensible path up is token-economy 7->8
(=78.0) and I examined it: pi's 8 requires measuring what the model would actually receive
plus cache warming; opencode triggers on last-turn usage and has no warming. Holding 7.

## Boundary notes
- Distribution sanity: 77.0 sits above cline (76.5, interop + surface depth), below pi (78.5,
  architectural unity), far below codex (88.5, kernel safety). Consistent.
- If the V2 engine ships as the primary loop with its projector/epoch model proven in CI,
  architecture and operability each plausibly move to 9 -- re-review at that commit.
