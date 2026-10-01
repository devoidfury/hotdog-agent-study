# devon - T1 review

entropy-research/Devon. Python agent core (`devon_agent`, ~8k LOC of real product) + React/Ink
TUI (~1.7k LOC) + electron shell + a 63.7k-LOC vendored-style `devon_swe_bench_experimental`
tree that dominates raw LOC. AGPL-3.0 file license. Snapshot: shallow clone, single commit
`8f68f1d` dated 2024-07-29.

## Anchor question

**Which anchor is this closer to, and why?** Closer to **codel (23.0)** than to nanocoder
(60.5): Devon has a genuine loop, git checkpoints, and sqlite persistence -- real machinery
codel lacks -- but its safety (no gate at all), verification (no CI), and durability sit on the
codel rungs, putting it ~10 pts above codel and far below the C floor.

## Activity / census sanity

- Shallow clone confirmed (`.git/shallow`), so HEAD date alone proves nothing. Remote check:
  `pushed_at 2025-05-26`, `archived: false` (GitHub API, 2026-09-29). Last push ~16 months
  before session date => dead >12mo; rule (b) cap at B applies (non-binding, see below).
- **Snapshot staleness**: the local clone is 10 months older than the remote tip. All findings
  are snapshot-bounded ("evidence-limited" for anything merged 2024-08..2025-05).
- **Census corrections**:
  - `test_loc: 27,571` is inflated ~17x. Real test code: `devon_agent/test/` = 597 LOC
    (8 files, one entirely commented out) + ~1.0k LOC of swe-bench test scripts inside
    `devon_swe_bench_experimental` (which also contains a duplicate `environment/test copy/`
    dir the glob swallowed).
  - `head_date 2024-07-29` is the shallow clip date, not repo activity (see above).
  - non_test 63,886 is dominated by the swe-bench experimental tree (63.7k py LOC), not the
    agent product.

## What the code actually does

Core loop: `Session.run_event_loop` (`devon_agent/session.py:788`) walks an append-only
producer/consumer event list (`Task`, `ModelRequest`, `ModelResponse`, `ToolRequest`,
`ToolResponse`, `Interrupt`, ...). `step_event` (`session.py:845`) is a match on event type;
`ModelRequest` calls `ConversationalAgent.predict` (`devon_agent/agents/conversational_agent.py:172`),
`ModelResponse` text-parses `<THOUGHT>/<COMMAND>` tags (`conversational_agent.py:30-46`, brittle
`.split()` chain wrapped so any parse failure becomes a `Hallucination` re-prompt), `ToolRequest`
(`session.py:913`) dispatches straight to the tool -- no gate between model output and execution.
Concurrency is `time.sleep(1)` polling (`waitForEvent`, `session.py:60`) and a sleep-loop pause;
`terminate` blocks until a flag flips (`session.py:259-265`).

Versioning/checkpoints: session works on a `devon_agent` git branch; `Checkpoint`
(`devon_agent/config.py:16-25`) stores commit hash + event_id + full agent chat history + state.
`Session.revert` (`session.py:229`) rolls back code, event log, and prompt history together.
`git_setup("load")` (`session.py:267` onward, ~400 LOC of near-identical error blocks)
reconciles drift: user commits made while the agent was away are detected and injected into the
chat history as a synthetic user message. `merge` (`session.py:1199`) applies the agent-branch
diff patch onto the user branch. This is the most substantial mechanism in the repo.

Token economy: none. OpenAI prompt path resends the entire `chat_history` every turn
(`conversational_agent.py:151,169`); Anthropic path flattens history into a bash-transcript
rebuild per request (`conversational_agent.py:119`) -- no cache reuse, guaranteed quadratic
growth. Only context shaping is editor-view pagination (PAGE_SIZE, `conversational_agent.py:81-101`).
No compaction, no token counting, no overflow detection anywhere in `devon_agent` (grep: zero
hits for compact/trim/overflow).

Safety: no approval, no deny-list, no sandbox in the product path. `ShellTool`
(`devon_agent/tools/shelltool.py:30`) writes `fn_name + args` to the stdin of a persistent
`/bin/bash -l` (`environments/shell_environment.py:94,111`); unknown tool names fall through to
that same shell (`session.py:967-998`). Docker environment exists but is for the swe-bench eval
harness, not the user session. The FastAPI control plane binds `host="0.0.0.0"` with no auth
(`__main__.py:41`) exposing session start/event/response endpoints (`server.py:238,331,496`) --
anyone on the network can drive an agent that executes arbitrary bash locally. PostHog telemetry
is on-by-default, opt-out only on exact env string "true" (`utils/telemetry.py:106-113`).

Verification: 8 pytest files (~597 LOC). Some are genuine behavior tests -- `test_edit_tool.py`
applies edit blocks against a temp-dir shell env; `test_local_shell_environment.py` exercises
the polling reader. `test_event_system.py` is 163 lines of commented-out code. No CI at all:
`.github/` contains only `autopr.yaml`. The `evals/` dir and swe-bench tree are eval scaffolding,
nothing runs them automatically.

Orchestration/operability: single agent; no subagents/queues. Persist-to-sqlite every step
(`session.py:1191`), resume via `from_config` (`session.py:187`) which re-runs git reconciliation;
server exposes pause/resume/revert/reset/terminate/diff endpoints (`server.py:301-527`).
Crash semantics are thought about (corruption detection returns "corrupted" -> new session) but
unproven -- nothing tests the load/reconcile path.

Interop: private REST + events-stream contract for its own TUI/electron front-ends; no MCP, no
ACP, no published SDK. Providers: OpenAI + Anthropic via litellm with hardcoded `max_tokens: 4096`
(`model.py:39-117`). Tool docstrings are rendered into the prompt as manpages/docstrings
(`session.py:1111-1124`) -- an early tool-catalog rendering, done fresh per request.

Docs/DX: README is marketing-forward and its license badge says "Apache 2.0 License"
(`README.md:12`) while `LICENSE` is AGPL-3.0 -- a misleading claim. MANIFESTO.md, CONTRIBUTING.md,
install.sh exist; no docs/ tree, no SECURITY.md, no reference docs for the HTTP API.

## Scores (anchor-referenced)

| dim | score | rationale / best evidence |
|---|---|---|
| architecture | 4 | Real event-driven loop + env/tool separation (`session.py:788,845`) beats codel's no-loop oracle, but 1,299-LOC Session god-object fuses loop + git state machine + merge logic; sleep-polling throughout (`session.py:60`). nanocoder's 5 rung has parallel re-implementations; devon has one loop but a bigger fuse. |
| verification | 3 | ~600 LOC of mostly-honest tool tests (`test_edit_tool.py`, `test_parse_commands.py`), one fully commented suite, zero CI. Below crush/nanocoder's 7 by a mile; above codel's 0 because tests exist and assert behavior. |
| safety-enforcement | 2 | Nothing binds: direct model->bash dispatch (`session.py:913` -> `shelltool.py:30` -> `shell_environment.py:111`), unauthenticated 0.0.0.0 control plane (`__main__.py:41`). Git-branch isolation is recovery, not enforcement (shell can escape it). codel's 2 rung ("reads misleadingly") -- Devon doesn't even claim a gate, but the pair-programmer UX + open port is the same blast radius. |
| token-economy | 2 | Editor pagination only; full-history resend per turn (`conversational_agent.py:169`); no compaction/overflow handling anywhere. Just above codel's 1 because pagination and history-flattening exist; below everything else. |
| orchestration | 4 | Single-session resume + checkpoint journals (`config.py:16-25`, `session.py:1191`); rate-limit modeled as event (`session.py:880-893`); no subagents/queues/budgets. codel's 4 is a DB task queue; devon's reconcile-on-load is comparable substance. |
| interop | 3 | Undocumented private REST API for own UIs (`server.py:140-527`), litellm providers, no MCP/ACP/SDK. Between codel's 2 and pi's 6 headless-contract rung. |
| operability | 5 | Checkpoint revert tri-consistency (code+events+history, `session.py:229-252`), drift detection on resume, full session lifecycle endpoints. Weakest relative bar: below nanocoder's 6 (no crash-recovery posture evidence), well below crush's 8. |
| originality | 5 | 2024-era "pair programmer on a git branch" shape executed with real mechanism (revert/merge/drift-reconcile); fossil versioning alternative (`versioning/fossil_versioning.py`). Distinctive but ancestor-shaped -- most of it exists in better form in aider-lineage successors. |
| durability | 1 | Org-backed once, 3.4k stars, but pushed_at 2025-05-26 remote-verified (~16mo), no CI, no SECURITY.md, snapshot shows 1 contributor. codel's 0 was remote-verified abandoned; devon is the same minus the archived flag. |
| docs-dx | 3 | No API docs, README license badge contradicts LICENSE (`README.md:12` vs AGPL file), install.sh only. |

**Weighted total: 33.5 -> band D.** (6 + 4.5 + 2 + 2 + 4 + 3 + 5 + 5 + 0.5 + 1.5)

Strongest dimension: **operability** (5). Weakest dimension: **durability** (1).

## Calibration / provenance

- Provenance: original per census; remote is not a fork (GitHub `fork: false`). No identity
  collisions in corpus.
- Rule (b) dead-cap B: applied, non-binding (already D).
- AGPL-3.0: all findings are concepts-only; `effort_for_us` values assume clean-room; no code
  porting recommended. The README Apache-badge/AGPL-file mismatch is itself a hazard for anyone
  citing the repo as permissively licensed.
- Not a leaked-snapshot suspect: repo is the canonical origin.
