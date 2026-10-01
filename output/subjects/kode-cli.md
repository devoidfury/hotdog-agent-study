# kode-cli (T2) - @shareai-lab/kode v2.2.0

Reviewed: 2026-09-29. Read-only static review of /data/samples/agents/kode-cli.
License: Apache-2.0 (LICENSE + manifest). Shallow clone (`.git/shallow`, 1 commit, head 2026-08-28).

## Anchor question (before scoring)

Closer to **cline**: a broad-interop TypeScript agent whose approval policy binds in-loop and is
tested, whose OS sandbox is present but not the default, and whose context management is
threshold-summarize based -- except kode flips cline's default posture to YOLO and back-fills the
gap with a second-layer LLM gate cline does not have. Numerically it lands in the crush band (mid-B).

## Provenance / lineage (dispatcher lore says "cline lineage" -- code says otherwise)

- Zero cline markers: `attempt_completion`, `ClineProvider`, `ClineIgnore`, `summarize_task`,
  `condense`, shadow-git checkpoints -- all 0 hits repo-wide (`grep -rl` over packages/ apps/).
- Strong Claude-Code-internal markers in readable TypeScript:
  - `packages/core/src/engine/query-executor.ts:22` -- `process.env.USER_TYPE !== 'ant'` gates a
    dual-response "binary feedback" feature (ant = Anthropic internal user type).
  - `packages/core/src/ai/llm/anthropic/client.ts:97` -- `USER_TYPE === 'ant'` key handling.
  - `packages/core/src/ai/llm/retry.ts:5,49` -- `USER_TYPE === 'SWE_BENCH'` retry profile.
  - `packages/core/src/logging/log/jsonLog.ts:173`, `messages.ts:13`,
    `mcp/client/config.ts:157` -- `USER_TYPE === 'external'` branches.
- Internal vocabulary is claude-shaped: `canUseTool`, `ToolUseContext`, `INTERRUPT_MESSAGE`,
  ToolUseQueue with concurrency-safe barrier, TodoWrite/BashTool/FileEdit tool names, plan mode,
  PreToolUse/Stop hooks with `permissionDecision` JSON, VCR fixture helper
  (`services/vcr.ts:14-56`).
- Conclusion: the codebase descends from Claude-Code internals (same org's anon-kode route), not
  from cline. There is no upstream in this corpus to apply rule (a) against; score on merits, but
  see the license-risk finding on origin provenance.

## Census sanity

- `test_loc: 23069` understates: 221 `*.test.ts(x)` files, 27,450 raw lines
  (`find ... | xargs wc -l`). Not the 10x misses of crush/cline, but ~+4.4k.
- `contributors: 1` is a shallow-clone artifact; do not read as bus-factor 1 without remote check.
  head_date is recent (2026-08-28); no activity claim needed.
- LOC sane: cloc TS = 145,294 code total; minus tests ≈ 118-120k non-test, census 139k includes
  non-TS and slightly overstates, within tolerance. T2 tier correct.
- `yoga.wasm` (88K) and `packages/kode-bin-*` (8K stubs each) are distribution shims, not blobs.

## Core loop (line-level read)

`packages/core/src/engine/turn.ts:21-56` (runTurn: builds system prompt + context, delegates) ->
`orchestrator.ts:18-62` (query: per-message JSONL session persistence wrap via
`appendSessionJsonlFromMessage`) -> `message-pipeline.ts:90-473` (the actual agent loop):

- Explicit `maxTurns` (message-pipeline.ts:100-116, `MaxTurnsExceededError`) and `maxBudgetUsd`
  (118-128, `MaxBudgetUsdExceededError`) budgets enforced at every recursion.
- Compaction hooks into the loop before every LLM call: micro-compact then auto-compact
  (130-146, see below).
- Background-bash notifications injected as synthetic assistant messages so the model decides when
  to poll TaskOutput (150-196) -- pull-based, no interrupt-in-place.
- Hook events wired at real loop points: UserPromptSubmit (205-256, incl. hook-initiated block
  path), Stop/SubagentStop with re-entry cap `MAX_STOP_HOOK_ATTEMPTS = 5` (382-424), transcript
  kept fresh for hook scripts (199).
- Tool execution via `ToolUseQueue` (`engine/pipeline/tool-use-queue.ts:61-318`): input schema
  validated before scheduling (90-96), concurrency-safe tools run in parallel, non-safe act as
  ordering barrier (106-112, 250-262), sibling-error and user-interrupt synthesize `is_error`
  tool_results so the API contract never sees orphaned tool_use ids (35-57, 160-190).
- Recursion at 447-456 passes an immutable messages snapshot; abort checked at 3 places.

No god files anywhere: largest source file repo-wide is `packages/tools/src/tools/system/LspTool/call.ts`
at 863 LOC; largest CLI file 555 (`apps/cli/src/utils/Cursor.ts`). Loop/state is fully separated
from the Ink product surface -- by the errata criterion this is the 8->9 property -- but the
9-rung also demands field-defining structure; kode's layering is excellent execution of a known
shape (core/runtime/tools/protocol/client zones), hence 8.

## Compaction / token economy (line-level read)

Two tiers behind shared threshold math (`utils/autoCompactThreshold.ts:9-16`: 10% reserve capped
20k, auto-compact margin 13k tokens, warning margin 20k):

1. **Micro-compact** (`utils/microCompactCore.ts:19-22` constants, `:306` entry): offloads old
   tool-result blocks to disk keeping a 400-char preview reference, keeps last 3 tool uses, only
   fires when >= 20k tokens would be saved and usage is past the warning margin (211-224).
   Idempotent (skips already-persisted ids, 96-124), images/non-strings left intact (349).
2. **Auto-compact** (`utils/autoCompactCore.ts:135-201`): structured 8-section summary prompt
   (71-96) enriched with live **task-list / skills / MCP / plan-file snapshots** (215-241),
   PreCompact hooks that can block compaction and inject custom instructions (157-183),
   **fit-aware compression model selection**: prefers the `compact` model pointer, falls back to
   `main` when the compact model's 90%-context budget cannot hold the history (251-276), file
   recovery of recently-touched files appended post-summary (315-330), compaction boundary
   persisted as a JSONL summary record so resume screens survive (333-355).

Weakness: the trigger measures `estimateTokens` (chars/4 x4/3 heuristic,
`utils/tokens.ts:59-61`), not the provider-usage truth that `countTokens`
(`tokens.ts:3-28`) already computes -- pi's projected-context rung (8) is where this caps kode at 7.
Prompt-cache discipline exists (`ai/llm/anthropic/cacheControl.ts:14-85`: priority-ranked
cache_control within Anthropic's 4-block limit, strips foreign markers) but there is no
cache-monotonicity concern for compaction (compaction busts the prefix by design, as everywhere).
Cost visibility: `cost-tracker.ts` + maxBudgetUsd + per-session cost fields. No loop detection.

## Permissions / sandbox (line-level read)

`permissions/engine.ts:71-354` (`hasPermissionsToUseTool`, the single chokepoint the loop calls):

- Modes: yolo / default / acceptEdits / plan / dontAsk / bypassPermissions.
- **bypassPermissions keeps a write-safety floor** (engine.ts:96-133): unsafe write paths are still
  denied unless the user explicitly sets `KODE_BYPASS_SAFETY_FLOOR`; tested in
  `test/unit/bypass-permissions-safety-floor.test.ts` (53 LOC).
- **YOLO auto-approves only what would prompt** (engine.ts:344-351): explicit deny rules
  (`shouldPromptUser === false`) still win under YOLO -- deny > yolo precedence, which most
  "yolo flag" implementations get wrong.
- Rule engine: allow/deny/ask groups flattened from settings + CLI `--allowedTools` into one
  matcher (152-215), path-rule matching with symlink expansion and edit permission suggestions
  on the prompt (226-303). Bash rules are parsed, not naive prefix: subcommand splitting with
  continuation normalization, compound-command detection, redirection and sed safety, path checks
  (`permissions/bash/engine.ts:1-80`, shellTokens.ts, sed.ts, paths.ts, xi.ts/xiChecks.ts a
  layered ask/passthrough checker).
- Bash call path adds an independent **classifier -> LLM reviewer ladder**
  (`packages/tools/src/tools/system/BashTool/call.tsx:131-203`): 14-category regex data-loss
  classifier (`dataLossRules.ts`, e.g. fs_delete/privilege/credentials/obfuscation) decides
  `needsLlmGate`; a purpose-built quick-model verdict blocks or allows (`llmSafetyGate.ts:138-180`,
  300s timeout, stop-sequence, `</final>`); on gate error, fail-open only if the command will run
  sandboxed -- unsandboxed + gate failure = block (`call.tsx:192-203`). Because the gate lives in
  the tool call, it survives YOLO auto-approval. Tested: `bash-llm-gate.test.ts`,
  `dataLossRules.test.ts`, `bash-tool-validate-input-no-banned-commands.test.ts`.
- OS sandbox: bwrap with net namespace + `--ro-bind` plans on Linux
  (`packages/runtime/src/shell/linuxSandbox.ts`, `bunShellSandboxPlan.ts`), dynamic seatbelt
  profiles on macOS incl. deny-unlink regex rules and TMPDIR pinning
  (`runtime/src/shell/macosSandbox.ts:36-60`), violation output captured under a tagged channel,
  network funneled through auditable HTTP/SOCKS proxies (`core/src/sandbox/
  sandboxNetworkInfrastructure/httpProxy.ts, socks5Proxy.ts, linuxBridge.ts`). Linux seccomp
  assets buildable (`scripts/build-seccomp-assets.mjs`). Docs state honestly it is the "first
  layer" with named gaps (`docs/system-sandbox.md:1-30`).
- But: sandbox is **opt-in** (`sandboxConfig.ts:364` -- `sandbox.enabled !== true` yields no
  plan), and the shipped default is **YOLO** (`utils/permissionModeState.ts:7`
  `ACTUAL_DEFAULT_MODE: PermissionMode = 'yolo'`; README.md:46 states it plainly and recommends
  `--safe`). Honest, still the wrong default.
- Trust scoping is thin: the only trust gate found is workspace-scoped MCP `headersHelper`
  (`mcp/client/connection.ts:121-124 checkHasTrustDialogAccepted`); project-defined hooks spawn
  plain subprocesses with no trust decision (`hooks/executor.ts:31`).

## Verification

- 221 test files / 27,450 LOC, and they assert behavior, not existence:
  `bash-sandbox-permission-matrix.test.ts` drives the real `hasPermissionsToUseTool` against
  fixture `.claude/settings.json` trees; `hooks-pretooluse.permissionDecision.test.ts` runs real
  child-process hook scripts through `runToolUse` and asserts "JSON deny blocks even with exit 0";
  maxTurns/maxBudget, dontAsk, plan-mode, tool-scheduler concurrency, stream-json protocol,
  ACP contracts (`test/contract/acp-contracts.test.ts`), daemon fs-path security
  (`integration/daemon-fs-path-security.test.ts`) all have specs.
- CI (`.github/workflows/ci.yml`): 3-OS matrix, pinned action SHAs, frozen lockfile, and four
  meta-gates on every push: format, **architecture boundaries** (`scripts/check-architecture.mjs`
  zone dependency DAG with a reverse-edge list that "may shrink" only, :42-46), `bun audit
  --audit-level=high`, typecheck, then `bun test` + build.
- Ceiling reasons: no in-CI evals, no fuzzing (anchors' 8-ceiling); provider integration tests
  silently `test.skip` without API keys (`integration/integration-cli-flow.test.ts:17-21`); only
  2 files use `mock.module` -- no scripted-SSE faux provider drives the loop in CI; the VCR
  fixture harness (`services/vcr.ts`) has no committed fixtures. TUI covered by 3 ink-harness
  files only. 7.

## Interop / orchestration / operability

- MCP client with OAuth + a **MCP server mode** (`mcp/server.ts` + `mcp-cli` bin), ACP agent
  (`kode-acp` bin, `apps/server/src/acp/kodeAcpAgent.ts`, contract + smoke tests), headless
  claude-SDK-compatible `-p --output-format stream-json` incl. control requests
  (`runNonTextPrintMode.ts`, `protocol/src/streamJson.ts` + parity tests), published SDK exports
  (`package.json` `./protocol`, `./daemon-client`), web UI + host server
  (`apps/server`, `apps/web`, ~9.6k LOC), AGENTS.md standard with Codex-style discovery rules and
  `.claude`/CLAUDE.md legacy compat (`compat/legacyClaude.ts`, `docs/compatibility.md`). 8.
- Orchestration: TaskTool subagents foreground/background with transcripts
  (`TaskTool/callForeground.ts:378 upsertBackgroundAgentTask`), TaskOutput/TaskStop tools, plan
  mode with enter/exit tools + reminder injection, expert-model consultation tool
  (`AskExpertModelTool`, `@ask-model-name`), model pointer system (main/compact/quick) with
  multi-model collab. Missing: loop detection, durable cron/queue, budget per subagent. 7.
- Operability: JSONL session journal appended per message, `--resume/--continue/--fork-session/
  --session-id` with validation errors (`cliParser/rootAction.ts:356-430`), session resume
  discovery + cleanup-retention tests, debug logger with debug-latest symlink, JSONL error log,
  statusline, startup profiler. **No checkpoint/rewind** -- `controlRequests.ts:98-99` answers
  `rewind_files` with "not supported in Kode yet". 7.

## Docs / durability

- Curated docs tree with an explicit anti-drift policy (`docs/README.md:3-5`: stale plans kept
  out of repo), honest system-sandbox status doc (zh), model-compatibility profiles for
  constrained endpoints (`docs/model-compatibility.md:17,64`), ACP/MCP/versioning/release docs. 7.
- Durability: npm publish with provenance (`publishConfig.provenance: true`), tag-driven
  publish workflow, per-platform binary distribution, release-notes-in-UI machinery. Shallow
  clone hides real contributor count; small org (ShareAI-lab), no SECURITY.md found. Evidence-
  limited; 5.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | check-architecture.mjs:42-46 + ci.yml:42; flat tree (max 863 LOC LspTool/call.ts); loop separated from hosts (turn.ts -> orchestrator.ts -> message-pipeline.ts) |
| verification | 7 | bash-sandbox-permission-matrix.test.ts; hooks-pretooluse.permissionDecision.test.ts; ci.yml 3-OS + 4 meta-gates; but skip-without-key integration (:17-21), no faux provider, no evals/fuzz |
| safety-enforcement | 6 | engine.ts:96-133 bypass floor, :344-351 deny>yolo; BashTool/call.tsx:131-203 LLM gate fail-closed-unless-sandboxed; macosSandbox.ts/linuxSandbox.ts; but permissionModeState.ts:7 yolo default + opt-in sandbox (sandboxConfig.ts:364) |
| token-economy | 7 | microCompactCore.ts:19-22,306; autoCompactCore.ts:251-276 fit-aware compact model, :333-355 persisted boundary; cacheControl.ts:14-85; trigger on chars/4 estimate (tokens.ts:59-61) not usage truth |
| orchestration | 7 | tool-use-queue.ts:61-318 barrier scheduler; TaskTool bg/fg + TaskOutput/Stop; plan mode; no loop detection/queues/budgets per subagent |
| interop | 8 | kode-acp + acp-contracts.test.ts; mcp server+client+OAuth; stream-json headless parity tests; SDK exports; .claude compat corpus |
| operability | 7 | rootAction.ts:356-430 resume/fork flags; JSONL journal (orchestrator.ts:30-43); no rewind (controlRequests.ts:98-99) |
| originality | 7 | classifier->LLM bash gate surviving yolo, fail-closed tied to sandbox state; model pointers with fit-fallback; @ask expert consult; arch reverse-edge ratchet |
| durability | 5 | publish provenance + tag workflow; shallow => contributor evidence-limited; no SECURITY.md |
| docs-dx | 7 | docs/README.md:3-5 anti-drift; system-sandbox.md honesty; bilingual README; security notice README.md:46 |

**Weighted total: 70.5 -> band B.** Strongest: architecture (8, tied with interop; named for the
CI-enforced boundary ratchet + god-file-free tree). Weakest: durability (5).

## Calibration notes

- Rule (a) not applicable: no in-corpus upstream; not a cline sync-fork (zero cline markers).
  Lineage is Claude-Code-internals-derived per embedded markers; treat as divergent by origin and
  score on merits. Portables below are concept-level pending the license-risk finding.
- No demotions applied. 70.5 is mid-B, not boundary-risk (nearest edges 65/78 are >2 pts away).
- Census: test_loc understated ~4.4k lines; contributors=1 is a shallow artifact; tier T2 correct.
