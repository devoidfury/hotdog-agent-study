# binharic-cli — T1 review

Subject: @cogitator/binharic-cli (/data/samples/agents/binharic-cli)
Census: TS, non_test 18,221 / test 10,601, 1 contributor, head 2025-10-16, shallow, MIT, provenance original, tier T1.
Manifest version 0.1.0-alpha.6; published to npm; Ink/React TUI; AI SDK (Vercel `ai` v5) based; multi-provider (OpenAI/Anthropic/Google/Ollama).

## Anchor question (answered before scoring)

**Closest anchor: nanocoder (60.5, C)** — same shape: solo-authored TypeScript Ink TUI with the agent loop fused into UI state, real-but-partial safety machinery, and a test corpus needing census correction; binharic sits a full band lower because nanocoder's specs actually import production code and its sandbox/ACP surfaces are real, while binharic's headline loop-control and safety layers are largely unreachable from the product.

## Remote / census verification

- `.git/shallow` present; local HEAD `52ccca70` (main, "Polish README (#6)", 2025-10-16). Remote `main` == local HEAD; remote `develop` has extra commits (`0e2b95c7`). GitHub API: `pushed_at 2025-11-01`, not archived, 19 stars, 1 fork, created 2025-08-16 from **template** `habedi/template-typescript-project` (scaffolding, not an agent fork — no rule (a) constraint). Activity ~11 months at study date: quiet, not dead>12mo.
- LOC correction: `src` TS/tsx = 10,573 lines (wc); census non_test 18,221 almost certainly swallowed `package-lock.json` (7,105 lines). tests dir = 12,951 wc lines / 88 files, but **37 of 88 test files never import anything from src** (see verification).

## Architecture

Two parallel agent-loop implementations:

1. **Real path**: `_runAgentLogicInternal` (src/agent/core/state.ts:544-958) — a recursive async function inside the zustand UI store. History is a flat `HistoryItem[]` in store state; each LLM turn streams text, collects tool calls (state.ts:784), partitions them into auto-run vs pending-approval (state.ts:822-878), then recurses. Interruption via module-level `shouldStopAgent` flag. Error path rolls the history back to its entry length (state.ts:920-927) and retries TransientErrors with exponential backoff (state.ts:933-940).
2. **Dead path**: `createBinharicAgent` + five specialized `Experimental_Agent` factories with budget/stop conditions and prepare-step handlers (src/agent/core/agents.ts:33-62, 108, 144, 177, 210; src/agent/execution/loopControl.ts; src/agent/execution/prepareStep.ts). **Zero product callers** — only tests reference them (agents.test.ts, safeToolAutoExecution.test.ts, anthropicAlignmentBugs.test.ts).

The tools layer is genuinely tidy: one small module per tool with zod schemas (src/agent/tools/definitions/*.ts, ~17 tools), a `runTool` dispatcher, and MCP tools folded in. But the loop living in a 958-LOC UI store mirrors the nanocoder anti-pattern. No provider/state separation problem elsewhere; state.ts is the single fat file.

## Token economy

- Real-path context management: `applyContextWindow` (src/agent/context/contextWindow.ts:63-101) — token-counts all messages, and if over 80% of model context, **drops oldest messages one at a time**. No summarization, no compaction tiers.
- Counting is a heuristic: exact `gpt-tokenizer` only under 12k chars, above that `length * 0.4` tokens (contextWindow.ts:7) — wrong for CJK/code, and non-monotonic at the boundary.
- Secondary silent trim: `config.history.maxItems` slice in provider (src/agent/llm/provider.ts:~230).
- The only summarizing handlers (`createToolResultSummarizer`, `createContextManager`, prepareStep.ts:32-51,106+) sit in the dead agent layer.
- No prompt-cache discipline, no cost/usage accounting in the real loop (loopControl.ts cost stop is dead too; streamText result usage is never read in state.ts).

## Safety enforcement

- What binds: default-deny approval. `SAFE_AUTO_TOOLS` (state.ts:18-30) is a hardcoded read-only allowlist (read_file, list, search, grep_search, get_errors, git_status/log/diff, validate...); bash, edit, create, insert_edit, run_in_terminal all fall to `pendingToolRequest` + `status: "tool-request"` (state.ts:878, 906) and require interactive UI confirmation before `confirmToolExecution` runs them (state.ts:381-460). Default posture is correct.
- No sandbox anywhere. bash tool spawns `/bin/bash -c` with cwd=process.cwd (src/agent/tools/definitions/bash.ts:46-52), guarded only by four regex deny entries (`rm -rf /`, mkfs, dd if=, fork bomb; bash.ts:8-13) — trivially bypassable (`bash -c`, `$(echo rm)`, `rm -fr`), classic regex-denylist.
- Two **phantom safety systems**: `PermissionsManager` (rules, allow/block lists, session/project/global scopes, sensitive-path patterns; src/agent/core/permissionsManager.ts:47,90,120) has **zero call sites in src and zero in tests** — the only real permission logic is the hardcoded Set. And `checkpoints.ts` risk-tiered checkpoint registry (`requestCheckpoint`/`setCheckpointHandler`/`assessRiskLevel`, checkpoints.ts:20,30,58) is never registered or called; the UI renders a "Sacred Checkpoint Required" dialog for `pendingCheckpoint` (App.tsx:90, CheckpointConfirmation.tsx:24) that nothing ever sets.

## Orchestration

Real: circuit breaker wrapping every provider call (wired, provider.ts:207-213; circuitBreaker.ts CLOSED/OPEN/HALF_OPEN with probe), transient-error retry with backoff + consecutive-error fuse (state.ts:683, 933, 942), 12+ prompt-template "workflows" exposed as one `execute_workflow` tool (src/agent/tools/definitions/workflow.ts:79). No subagents in the real path (the specialized agents are dead), no queues, no budgets that bind, no crash recovery or resume journal.

## Interop

MCP client only (src/agent/tools/definitions/mcp.ts:37-38, stdio servers from zod'd config src/config.ts:49). Four providers. No MCP server, no ACP, no headless/JSON mode (cli.ts renders the Ink app and nothing else), no published SDK.

## Operability

No session persistence or resume at all — history is in-memory, gone at exit; exit shows a metrics summary (llmRequests/tool time) and terminal-session tool cleans up child sessions on exit (cli.ts:8). Error-path history rollback (state.ts:920-927) keeps conversation consistent, retry UX exists. Winston logging + stderr suppression (cli.ts:11). Global uncaughtException/unhandledRejection handlers installed (cli.ts:19-35). Config system with validation. No rewind, no checkpoint restore (checkpoint machinery dead), no session list.

## Verification

CI runs on PRs/tags: vitest with coverage → codecov on Node 20/22 matrix (.github/workflows/tests.yml), eslint + tsc typecheck (lints.yml). No evals, no fuzzing, coverage non-blocking (`fail_ci_if_error: false`).

Test corpus quality is the subject's defining weakness: 37/88 test files never import `src`; they reimplement the logic under test inline and assert on the reimplementation (tests/agent/bugs/streamTimeoutBug.test.ts:19-45 rebuilds a local timeout harness) or are tautologies (`expect(toolName).toBeTruthy()`; tests/agent/tools/safeToolAutoExecution.test.ts:59,60-61,77). Existence assertions (`toBeDefined`) at tests/agent/core/codeQualityFixes.test.ts:174,178,200. The 51 files that do import src are decent unit specs (contextWindow, circuitBreaker, fileTracker symlinks, tool input validation, error hierarchy). Net: the suite proves units work but overstates coverage, and the most safety-relevant behavior (the approval gate in state.ts) has no test that drives the real loop.

## Originality

- Tech-Priest persona is genuinely threaded through code, not just README: system prompt (systemPrompt.ts:67), rejection/interrupt copy injected into model history (state.ts:458, 509-512), UI labels ("Sacred Checkpoint Required", CheckpointConfirmation.tsx:64). Cosmetic but real in-code.
- `autofixEdit`: when an edit's search string fails to match, a side-model call (hardcoded `gpt-5-mini`, OpenAI key required) repairs the search string under a verbatim-presence constraint (src/agent/tools/definitions/edit.ts:89; autofix.ts:46-80), plus `autofixJson` repairing malformed edit JSON (autofix.ts:22-43). Real, wired, useful.
- Streaming ``-tag filter across chunk boundaries (src/agent/llm/textFilters.ts:4-60) for leaking reasoning models.
- Circuit breaker on the LLM call path — standard pattern, uncommon here, wired.

## Durability

One contributor (habedi / CogitatorTech one-person org), 19 stars, 1 fork, template-derived, no SECURITY.md, no releases beyond npm alphas, quiet ~11 months (remote-verified `pushed_at 2025-11-01`; shallow clone noted).

## Scores (anchor-ladder-relative)

| dimension | score | best evidence |
|---|---|---|
| architecture | 5 | loop inside zustand UI store state.ts:544-958; dead parallel agent layer agents.ts:33-62; clean per-tool modules tools/definitions/ |
| verification | 3 | tests.yml runs coverage; but 37/88 tests import nothing from src (streamTimeoutBug.test.ts:19-45 replica; safeToolAutoExecution.test.ts:59 tautology) |
| safety-enforcement | 5 | approval gate binds by default (state.ts:825,878,906); no sandbox; regex denylist bash.ts:8-13; PermissionsManager+checkpoints unwired |
| token-economy | 3 | drop-oldest trim contextWindow.ts:63-101; 0.4×/char heuristic :7; summarizers dead in prepareStep.ts:106 |
| orchestration | 4 | wired circuit breaker provider.ts:207; backoff retry state.ts:933; execute_workflow templates workflow.ts:79; no subagent/queue/resume in real path |
| interop | 4 | MCP client mcp.ts:37-38; 4 providers; no headless, ACP, server, SDK |
| operability | 4 | history rollback state.ts:920-927; metrics summary; no persistence/resume at all; dead checkpoint UI App.tsx:90 |
| originality | 5 | autofixEdit wired edit.ts:89 + verbatim constraint autofix.ts:65; think-tag stream filter textFilters.ts; persona in code systemPrompt.ts:67 |
| durability | 2 | 1 contributor, 19 stars, pushed_at 2025-11-01 remote-verified, no SECURITY.md |
| docs-dx | 4 | README 123 lines incl. install/config; docs/ is a 3-line stub behind a "Documentation" badge; ROADMAP 286 lines |

**Weighted total: 40.0 → band D.** Strongest: architecture (5). Weakest: verification / token-economy / durability (tied; verification is the most consequential — it hides the phantom-safety problem).

## Calibration / provenance notes

- Provenance: template-scaffolded original, not a fork of any studied agent; no rule (a) interaction.
- No archived/dead cap applies (11 months, remote-checked), D comes from rubric, not a cap. Not within 2 pts of any band boundary.
- Census corrections: non_test_loc inflated to 18,221 by non-source files (real src ≈ 10,573 wc); head_date 2025-10-16 vs remote pushed_at 2025-11-01 (develop ahead); test_loc 10,601 overstates *effective* tests since 37/88 files never touch src.
