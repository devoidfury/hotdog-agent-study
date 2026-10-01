# neovate-code — T1 standard review

**Anchor question:** Closest to **cline** — one canonical provider-agnostic loop (AI SDK v3
interface) serving many thin hosts through a typed in-process RPC bridge, approval-bound safety
with real denial tests but no OS sandbox underneath, behavior-asserting test corps thin on the
loop/TUI; it sits below cline on test depth and breadth, and below crush on operability/CI.

## Subject shape

TypeScript, `@neovate/code`, MIT, upstream github.com/neovateai/neovate-code (census
"original" confirmed; not a fork per GitHub API). Shallow clone (1 commit); remote
`pushed_at 2026-03-24` matches HEAD — repo has ~6 months of no pushes at review date (not
over the 10-month archived threshold; treat as thinning, not dead). `src/` 66.0k LOC, vendored
`vendor/ripgrep` 22 MB binary blobs (LOC-safe: binaries not counted). Census `test_loc: 459`
is wrong: 39 `*.test.ts` files totaling **~8.4k LOC** (glob miss). Census `contributors: 1`
is a shallow artifact: GitHub contributors API shows sorrycc 1117 + xierenyuan 306 + 8 more
(top-10 ≈ 1,476 commits). Ant-Group ecosystem team (sorrycc = UmiJS founder; assets on
alipayobjects CDN), 1561 stars, no SECURITY.md.

Largest files: nodeBridge.types.ts 1930 (generated contract), slash-commands/builtin/plugin.tsx
1581, commands/commit.tsx 1446, ui/store.ts 1422, utils/git.ts 1173, tools/bash.ts 992,
loop.ts 804. No 5k-LOC product file (errata threshold cleared); ui/store.ts is a god-store
starting to fuse host state.

## Core loop (read at line level)

`src/loop.ts:163` `runLoop` — standalone, UI-free, model behind `LanguageModelV3` creator:

- Abort-signal mirror with cleanup (`:186-211`); cancellation honored at top of turn, inside
  stream chunk loop, before/after tool results — cancel path is genuinely careful.
- Retryable-vs-fatal stream classification with exponential backoff that stays cancellable
  (`:33-52`, `:437-465`); empty-response detection both on `finish` and on chunkless stream
  (`:366-372`, `:390-399`).
- Per-turn `History.compress` when `autoCompact` (`:226-230`).
- Prompt built system+llmsContexts+history, `@file` expansion once per input
  (`At.normalizeLanguageV3Prompt`, `:247-252`), then cache marks (`:254`).
- Approval: per-tool-call `onToolApprove` may approve, **edit params** (`:645-651`,
  `:655-659`), or deny with a user message; a bare denial short-circuits the whole batch with
  `tool_denied` and synthesizes denial results for un-run tools (`:661-697`).
- Approved tools run concurrently via `Promise.allSettled` (`:723-734`); exceptions become
  tool-error results fed back, not loop crashes.
- **Budget quirk:** `turnsCount -= approvedToolUses.length` (`loop.ts:737`) — a turn with N
  approved tool calls consumes 1-N of the 50-turn cap, so tool-heavy loops are effectively
  uncapped against `max_turns_exceeded` (`:214-224`). Anti-pattern finding.
- No loop-identity detection anywhere (grep: no loopDetect/nudge/repeat patterns) — runaway
  same-tool-call loops are bounded only by the (deflated) turn cap.

## Compaction (read at line level)

`src/history.ts:268-300` `compress` + `src/compression.ts` — two tiers, LLM-last:

1. **Tool-output pruning** (`compression.ts:127-212`): reverse walk, protects last
   `protectTurns=2` turns (`constants.ts:103`), protected tools (skill/task,
   `constants.ts:105`), a 40k-token recency budget (`constants.ts:101`), stops at the first
   already-pruned part, and commits only if ≥20k tokens would be freed
   (`constants.ts:102`, `compression.ts:190-208`); pruned parts keep metadata
   ("[Output pruned at ...]").
2. Only if still overflowing after re-measuring does it run LLM compaction
   (`history.ts:283-300`, `compact.ts:27-46` — XML `context_summary` prompt,
   `compact.ts:52-60`), replacing history with one summary message (`history.ts:307-316`).

Trigger is *reported* usage of the last assistant message vs `context × 0.7`
(`history.ts:216-236`, `compression.ts:64-92`, `constants.ts:98`), below
`MIN_TOKEN_THRESHOLD≈25.6k` no-op (`constants.ts:76`) — usage-based, not projected-prompt
(pi's 8-rung measures what the model would actually receive). No branch summarization, no
cache warming. Prompt-cache discipline: `promptCache.ts:12-15` gates on **model-name
substring** (`claude|sonnet|opus`) and stamps ephemeral cache control on first-2-system +
last-2-non-system messages (`:26-28`) — a static heuristic that slides each turn and never
fires for non-Claude cache-capable providers.

## Permission / sandbox code (read at line level)

- Model: tools declare `approval.needsApproval(context)` (`tool.ts:303-317`); loop enforces
  via `onToolApprove` and denial is tested (`loop.test.ts:106-108`); default mode is
  `default`, approval required (`config.ts:31,131`); yolo is one config flip away.
- **No sandbox of any kind** — grep for seatbelt/bwrap/sandbox hits only a doc string in
  `agent/builtin/neovate-code-guide.ts`. Bash runs via plain shell.
- Bash policy (`tools/bash.ts`): 24-root banned list (`:18-43`, incl. rm/curl/bash/sh),
  high-risk check with **pipeline-segment fallback evaluation explicitly credited to codex
  PR #7544** (`:248-273`), command-substitution rejection (`:283-287`, tested with quoted/
  escaped edge cases in `bash.test.ts:14-79`). But root-name matching trivially bypasses
  (`python -c`, `find -exec`, `xargs rm`), and bans only raise `needsApproval=true`
  (`:975-990`) — the enforcement is the approver's UI, not the executor.
- **Safety hole (ACP):** "Always allow" decisions are cached per `toolName:category`
  (`commands/acp/session.ts:113-119`) and replayed **without re-consulting per-command risk**
  (`:131-137`) — one `allow_always` on `bash:command` auto-admits every later command
  including banned/high-risk ones, since the cached rule short-circuits before the tool's
  own `needsApproval` logic would even surface risk.
- No SECURITY.md, no honesty doc about the absent sandbox (contrast pi's exemplary
  SECURITY.md at the same rung).

## Other dimensions in evidence

- **Verification:** 39 vitest files, ~8.4k LOC, assertions not existence (compression
  protect rules `compression.test.ts:120-172`, command-substitution edge cases
  `bash.test.ts:14-79`, session filter/fork 48 expects, pluginRegistry installer/registry/
  marketplace tests). Loop tests thin: one `describe`, parallel-exec timing + denial
  (`loop.test.ts:71-244`). CI: single workflow, ubuntu+windows × node 18/20/22,
  build+typecheck+format+test, plus `scripts/cli-integration-test.ts` — a **live** provider
  smoke hitting modelwatch/gemini-3-flash with a "hello" prompt (`test.yml` steps,
  `cli-integration-test.ts:13-16`). No evals-in-CI (errata cap 8 stands regardless), no
  fuzzing, no coverage gate; ui/, commands/, nodeBridge slices largely untested.
- **Orchestration:** task-tool subagents via AgentManager (`tools/task.ts:14-47`,
  `agent/executor.ts`), background bash with move-to-background event channel
  (`tools/bash.ts:843-860`, `backgroundTaskManager.ts`), git-worktree workspaces
  (`worktree.ts`, `commands/workspace/{create,complete,delete,list}.ts`), session fork
  (`/branch`, `slash-commands/builtin/branch.tsx:9-36`) + `/rewind` to file checkpoints
  (`rewind.tsx:1-30`, `snapshot/FileHistory.ts:393-443`). No queue, no workflow journal,
  no budgets, no loop breaker. Cancel path repairs incomplete tool uses into the session
  log (`nodeBridge/slices/session.ts:463-478`).
- **Interop:** MCP client (stdio + streamable-HTTP, `mcp.ts:2,43`), ACP server
  (`commands/acp/`, own test), programmatic SDK over the same typed bus (`sdk.ts:27-33`),
  headless `run` command (`commands/run.tsx`), web+websocket server
  (`commands/server/{server,web-server,websocketTransport}.ts`), 702-LOC VS Code extension,
  37 provider adapters incl. OAuth codex/copilot (`provider/providers/`), CLAUDE.md/AGENTS.md
  rules compat (`rules.ts:12-40`), skill installer reads Claude skill dirs
  (`skill.ts:732-741`). No MCP server mode, no published rival-harness interop story.
- **Operability:** session JSONL logs + `Session.resume` (`session.ts:39`), fork, rewind,
  `/context` token-breakdown (`slash-commands/builtin/context.tsx:10-22`), `/export`,
  `/status`, output styles, api-key rotation (docs/designs/2025-11-12). Token accounting in
  `usage.ts` is token-only — grep finds no dollar/cost anywhere.
- **Docs/DX:** README quickstart clean; in-repo docs are `docs/designs/*` dated design notes
  + `development_commands.md` only — product docs live on external neovateai.dev; no
  SECURITY.md; HandlerMap contract is code-generated (api-extractor.json).

## Scores vs anchors

architecture 7 · verification 6 · safety-enforcement 5 · token-economy 7 · orchestration 6 ·
interop 7 · operability 7 · originality 6 · durability 5 · docs-dx 4 → **62.0, C**
(boundary note: within 2 of the 65 B-floor; held at C on the untested-bridge depth, 5-rung
safety, and docs/durability drags).

**Strongest dimension: interop** (7) — the widest client set at this size: MCP + ACP + typed
SDK + headless run + WS server + VS Code + 37 providers, all riding one MessageBus contract.
**Weakest dimension: docs-dx** (4) — near-zero in-repo product documentation vs pi 9 / codex 7.

## Census corrections

- `test_loc: 459` → ~8,396 (39 `*.test.ts` in src/; glob miss).
- `contributors: 1`, `commits: 1` → shallow-clone artifacts; remote shows 10+ contributors,
  ≥1.4k commits.
- Activity: remote `pushed_at 2026-03-24` (verified) — 6 months stale at review, above the
  10-month archived bar; no archived/dead claim made.
