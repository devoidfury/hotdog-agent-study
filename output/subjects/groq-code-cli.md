# groq-code-cli -- T1 review

- Sample: /data/samples/agents/groq-code-cli, shallow clone, HEAD a303eb4 "feat: add support for kimi-k2-instruct-0905 (#18)".
- Upstream: https://github.com/build-with-groq/groq-code-cli (org: build-with-groq, Groq Inc). GitHub API: fork=false, MIT, created 2025-07-30, **pushed_at 2025-12-19**, not archived, 742 stars, 118 forks, 4 contributors (36/4/2/1).

## Anchor question (before scoring)
Closer to **nanocoder** than anything else -- same Ink/React TUI shape with approval-over-nothing-underneath -- but materially below every nanocoder rung: no CI test job, no sandbox, no session persistence, no interop plane; the honest placement is low-C-to-D, between codel (23, no loop) and nanocoder (60.5).

## Census sanity check
- `non_test_loc: 14535` is inflated ~2.6x. Actual TypeScript/TSX in tree: 5,920 lines total; 352 are tests (`src/tests/proxy-config.test.ts` 185, `test/*.test.ts` 167) => **~5,568 non-test LOC**. The excess is almost certainly package-lock.json (9,671 lines) bleeding into the count. Still inside T1 (>5k), but barely.
- `test_loc: 303` ~ right (352 raw).
- `contributors: 1, commits: 1` = shallow-clone artifacts. Remote shows **4 contributors, ~43 commits**.
- `head_date 2025-09-05` is the shallow HEAD. Remote activity ran through **2025-12-19** (~9 months before review date; under 12-month dead threshold, archived=false). No dead/archive cap applies, but momentum is clearly spent: issues disabled (`has_issues:false`), only a terraform-managed stale-PR bot in CI.

## Core loop (read: src/core/agent.ts:263-538)
Single synchronous tool-calls loop in an `Agent` class: push user msg -> `chat.completions.create` (non-streaming, max_tokens 8000, all 9 tools every call, agent.ts:344-356) -> if `tool_calls`, execute each via `executeToolCall`, push results, `iteration++`; else final message, return. Outer loop resets the 50-iteration cap on user consent (agent.ts:283, 528-541 with MaxIterationsContinue overlay). Error path: API errors become in-conversation system messages or a user retry overlay (agent.ts:487-517); 401 terminates. Interruption: AbortController per request + `isInterrupted` flags before each iteration and tool (agent.ts:287, 407) -- clean and tested-ish at the surface level.

Notable: the loop is a plain UI-agnostic class wired to the Ink UI purely through callbacks (`setToolCallbacks`, agent.ts:156-180), and tests construct `Agent` without any UI (test/agent-context-load.test.ts:26). That is a better boundary than the nanocoder rung (loop-in-React-hook). But state is a bare in-memory `messages` array with mutation hacks: `setModel` rewrites the system message by finding it via `content.includes('coding assistant')` (agent.ts:230); global mutable task list (`currentTaskList`, tools.ts:32-39) and module-global read tracker; nothing is persisted anywhere, ever.

## Permission / safety code (read: agent.ts:539-636, tool-schemas.ts:385-394, tools.ts:262-305, executeCommand tools.ts:588-640)
- Two-tier gate: `APPROVAL_REQUIRED_TOOLS=[create_file,edit_file]` (session-auto-approvable) vs `DANGEROUS_TOOLS=[delete_file,execute_command]` (never auto-approvable). `canAutoApprove = requiresApproval && !isDangerous && this.sessionAutoApprove` (agent.ts:576-578). Missing approval callback => default deny (agent.ts:610). Rejection injects a stop-the-turn system message (agent.ts:424-431). This is a genuinely considered policy shape, untested.
- Nothing underneath the approvals: `execute_command` is raw `promisify(exec)` with a 30s default timeout, no sandbox, no argv parsing (tools.ts:612-620); `read_file`/`create_file`/`edit_file` accept any absolute path (`path.resolve` only, tools.ts:123,185,222) -- with session auto-approve on, the model can read then rewrite `~/.bashrc`. `delete_file` is the only contained tool and its guard is `targetPath.startsWith(currentWorkingDir)` (tools.ts:264) -- the classic sibling-prefix bypass (`/work/proj` also admits `/work/proj-evil`).
- Read-before-edit enforcement (`validateReadBeforeEdit`, agent.ts:567-575 via validators.ts:11-18) is good tool-usage hygiene, tracked via a shared `Set` (tools.ts:39, validators injected at load).

## Context/compaction (read: agent.ts:66-86, utils/context.ts, grep sweep)
**Zero compaction, zero context-window awareness.** `messages` grows unbounded; grep for compact/trim/limit finds only: project-context-file trim at 20k chars (`GROQ_CONTEXT_LIMIT`, agent.ts:71-73), tool-arg-JSON-truncation error handling (agent.ts:549-557), and display truncation. An overflow hits the generic API-error retry overlay (agent.ts:487-517) and "retry" re-sends the same oversized payload forever. Cost visibility is decent, though: per-request usage callback (agent.ts:336-343) feeding TokenMetrics/Stats UI + `/stats`.

The one real context feature is `/init`: a fully deterministic, no-LLM project-context generator (utils/context.ts:35-80: walk with maxDepth 3 / maxEntries 1500, language histogram, package.json facts, config/notable file lists, ASCII tree) emitting `.groq/context.md` + `.groq/context.json`, auto-loaded as a second system message with `GROQ_CONTEXT_FILE/DIR/LIMIT` env overrides (agent.ts:66-86).

## Tests / CI
4 test files, 352 lines, all asserting real behavior (proxy priority chain with env save/restore, proxy-config.test.ts; context auto-load, test/agent-context-load.test.ts; context generator shape; init command). **But `package.json` ava config pins `files: ["src/**/*.test.ts"]` (package.json:53-56), so `npm test` runs only `src/tests/proxy-config.test.ts`; the three files in `test/` are orphaned from the runner.** CI is a single terraform-managed stale-PR workflow (.github/workflows/stale.yaml) -- zero test/CI jobs ever. Nothing tests the loop, approval gate, tool dispatch, or interrupt.

## Ops / interop / orchestration
Slash commands: /help /login /model /clear /reasoning /stats + registry (src/commands/). Debug mode writes `debug-agent.log` plus a full per-request `debug-request-N.json` and a masked-curl repro line to cwd (agent.ts:664-680, generateCurlCommand called at agent.ts:325) -- a nice replay-diagnostic habit. HTTP/SOCKS5 proxy support with documented `--proxy > HTTPS_PROXY > HTTP_PROXY` precedence, tested. API key stored plaintext in `~/.groq/local-settings.json` but with `mode 0o600` + explicit chmod (local-settings.ts:44-51). No resume/session/rewind whatsoever; process exit loses the conversation. No headless mode, no JSON output, no MCP/ACP/SDK; Groq-SDK-only provider. Inline `create_tasks/update_tasks` tool pair gives UI-rendered plan state (tools.ts:645-770) -- plan-as-display, not orchestration.

## Scoring rationale (weights per protocol)
| dim | score | best evidence |
|---|---|---|
| architecture | 5 | loop as UI-agnostic testable class (agent.ts:156-180, test/agent-context-load.test.ts:26) beats hook-coupling; docked for global mutable state (tools.ts:32-39), string-matched system-msg rewrite (agent.ts:232-236), 830-LOC tools.ts mixing dispatch+formatting+glob |
| verification | 3 | real-behavior tests (proxy-config.test.ts) but 3/4 orphaned by ava glob (package.json:53-56), no CI (only stale.yaml), zero loop/approval coverage |
| safety-enforcement | 4 | enforced two-tier gate w/ never-auto dangerous set + default-deny (agent.ts:583,610); no sandbox, unrestricted file paths, prefix-bypassable delete guard (tools.ts:270), untested |
| token-economy | 2 | unbounded history, no compaction, overflow-as-generic-error (agent.ts:487-517); usage metrics UI + context-file trim only (agent.ts:71-73) |
| orchestration | 3 | max-iter governor + consent reset (agent.ts:528-541), display-only task tools; no subagents/queue/persistence/loop-detection |
| interop | 2 | TUI-only, Groq-locked, no MCP/ACP/headless/SDK anywhere in src/ |
| operability | 4 | interrupt works (agent.ts:253-262), proxy config tested, 0600 key file, curl-replay debug export (agent.ts:664-680); no sessions/resume/diagnostics otherwise |
| originality | 5 | no-LLM deterministic /init context generator (context.ts) + env-var context override trio; "blueprint CLI" positioning is honest at 5.6k LOC |
| durability | 4 | Groq Inc backing + npm publish + 742 stars; effectively 1 active maintainer, issues disabled, last push 2025-12-19, no SECURITY.md |
| docs-dx | 5 | 265-line README matches cli.ts flags exactly, npx quickstart, proxy precedence documented, annotated source-tree map; no docs/ beyond images |

**weighted_total = 36.5, band D.** Strongest: architecture (5). Weakest: token-economy (2, tied with interop).

## Calibration notes
- No rule (a): original repo (fork=false), no upstream. No rule (b): not archived, remote activity within 12 months of review (pushed_at 2025-12-19) -- the shallow-HEAD "1 commit, 1 contributor" census row would have wrongly triaged this toward dead-adjacent; corrected.
- Identity hygiene: distinct from kimi-* subjects despite bundling kimi-k2-instruct as a selectable Groq-hosted model (HEAD commit); no fork relations to any corpus subject.
- Score is a genuinely-low D by rubric, not a cap: it is a deliberately minimal teaching/blueprint CLI, honest in docs about what it is (no phantom safety claims), but it lacks compaction, CI, persistence, and any interop surface.
