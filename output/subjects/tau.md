# tau (tau-ai) — T1 review

Repo under review: /data/samples/agents/tau, manifest name `tau-ai`, upstream remote
`github.com/huggingface/tau`. **Identity check: coding agent, not tau-bench.** pyproject
describes "A Python implementation of a minimalist Pi-style coding-agent harness"
(pyproject.toml:7), console script `tau = tau_coding.cli:app` (pyproject.toml:33), MIT LICENSE
file present. No relation to sierra-research/tau-bench (eval suite); no eval-task fixtures found.

Anchor question: **closest to pi** -- tau is a deliberate, self-declared Python port of pi's
three-layer architecture ("Pi-compatible" loop, "Pi-style" thresholds; loop.py:1, context_window.py:174),
so it shares pi's shape and discipline but sits several rungs lower: none of pi's cache warming,
session-tree UX depth, RPC/SDK ecosystem breadth, and none of pi's operability/orchestration polish.

## Census sanity

- Shallow clone (.git/shallow, 1 commit, HEAD "Pin GitHub Actions to commit SHAs (#740)" -- a real
  PR number, so upstream history exists but is hidden). head_date 2026-09-23 is 6 days old; no
  activity claim either way.
- LOC corrections: `src/**/*.py` = **57,229** (census non_test 89,129 overstates -- ~23.7k of that
  is markdown/website/lock). `tests/**/*.py` = **51,129 across 73 files** (census test_loc 41,827
  understates by ~9k). Tier T1 stands either way (5k-100k).
- contributors: 1 (census); Discord-linked community around a single author under the HF org.

## Architecture (7)

Textbook pi-shaped three-layer split, enforced in code: `tau_ai` (providers) / `tau_agent`
(portable brain) / `tau_coding` (app). Verified purity: no `tau_coding` or textual imports inside
tau_agent/tau_ai (grep empty). Core loop is genuinely separate and small:
`src/tau_agent/loop.py` (376 LOC) is a pure async event generator with steering/follow-up drains
(loop.py:100-119, 181), max_turns guard (:84-96), cancellation checks (:281), and tool exceptions
caught as error results -- "tools are an isolation boundary" (loop.py:355). Harness adds queues,
listeners, and interrupted-tool-call repair that flows through events to persistence listeners
(harness.py:160-201, 236-259) -- careful state modeling. Docked per the errata rule: product-side
god files -- `tau_coding/tui/app.py` **8,642 LOC** and `tau_coding/session.py` **5,095 LOC**
(CodingSession fuses durable entries, compaction, provider switching, extensions; session.py:405+).
Latter is worse than crush's agent.go fusion pattern; former exceeds the pi errata's 6.8k docking
factor. 7, not higher.

## Verification (7)

51k LOC of pytest specs that assert behavior, not existence:
- Exact canonical event-sequence equality from FakeProvider scripted streams
  (tests/test_agent_loop.py:53-80).
- Compaction property assertions on live sessions: recount after compact, entry-replacement
  invariants (test_coding_session.py:1683-1719, 2330-2334; 145 tests in that file alone).
- Hook blocking + fail-safe-on-raising-hook tested (test_extensions.py:1094, :1206).
- Trust store/persistence: 35 tests (test_project_trust.py).
- Cross-provider history replay tests (test_cross_provider_history.py, test_pi_event_protocol.py).
CI: pytest + ruff lint + ruff format + mypy on PR/push (ci.yml:41-52), all actions SHA-pinned,
`permissions: {}` default-deny (ci.yml:14). Ceiling: single ubuntu job, no coverage gate, no evals,
no fuzzing -- the 8 ceiling per errata is not reached; crush/nanocoder rung = 7.

## Safety-enforcement (6)

Same shape as pi's rung-6 description: **no default protection, blocking hook enforced, project-trust
gate, exemplary honesty.**
- Bash executes with no approval and no sandbox: `asyncio.create_subprocess_shell` directly
  (tau_coding/tools.py:627-646); no approval/yolo/allowlist concept anywhere in tau_coding
  (grep for approve/confirm/deny hits only project_trust).
- Blocking hook binds inside the loop: `before_tool_call` gate turns a block into an error result
  (loop.py:285-291); extension `tool_call` hook can block (extensions/api.py:464-477) and a
  raising hook blocks fail-safe (test_extensions.py:1206-1218). Weaker than crush's per-call-id
  grants: hook payload "Carries no tool-call id" (api.py:466-467).
- Project trust is an explicit, tested input-loading gate with scopes exact/parent/run
  (project_trust.py:27-29), file-locked persistence, and unusually explicit honesty:
  "deliberately not a filesystem, process, network, tool, model, or prompt-injection sandbox"
  (project_trust.py:1-5), repeated to the user in the TUI prompt (tui/project_trust.py:122).
Not below 6 because there is no misleading claim; not above because there is nothing kernel- or
even allowlist-underneath, by design.

## Token-economy (7)

- Provider-anchored accounting: latest successful assistant's provider-reported usage is
  authoritative for the prefix; only trailing messages/dynamically-added tools are estimated;
  stale-usage invalidation after history rewrites (context_window.py:186-260). Better than raw
  char counting, comparable in spirit to pi's projected-context trigger.
- Threshold = context_window - reserve, Pi-style (context_window.py:173-177), plus
  model-limit-derived `effective_auto_compact_token_limit` (model_limits.py:34).
- Multi-trigger but single strategy: auto before prompt (session.py:3191), after prompt (:3314),
  after continue (:3370), and **overflow-triggered compaction with retry** (`will_retry=compacted`,
  session.py:3255-3265). Summary merging is incremental: previous summary re-fed via
  UPDATE_SUMMARIZATION_PROMPT (context_window.py:276-297).
- Branch summaries for abandoned session-tree branches with structured Goal/Progress/Next format
  (branch_summary.py:16-60).
- Cache discipline: Anthropic `cache_control` breakpoints incl. last-tool breakpoint and a
  gateway-compat off-switch (anthropic.py:452-477), OpenAI prompt-cache affinity key per session
  (openai_cache.py:11). No cache warming, no token-budget fresh window -- below codex(9)/pi(8). 7.

## Orchestration (3)

Nothing in-core: no subagents, no queue, no loop detection, no checkpoint/revert (grep hits for
subagent are only comments about an *extension* -- tui/app.py:3522 "tau-subagents"; git-worktree
hits: none). Durable sessions + RPC exist, but that is operability, not orchestration. Below
codel's 4 (which at least has a restart-tolerant task queue); above 2 because extension runtime +
session durability give hosts the primitives to build queues themselves.

## Interop (5)

- Pi-compatible JSONL RPC frontend, documented in-repo (rpc.py:1, 799 LOC;
  website/content/reference/rpc.md) plus print mode and custom-frontend guide
  (website/content/internals/custom-frontend.md).
- Broad provider plane: openai/anthropic/google/mistral/ollama + OAuth for anthropic, github copilot,
  device flow (oauth_*.py); llama.cpp local backend builtin extension with HF model discovery
  (extensions/builtins/llama_cpp/).
- No MCP. No ACP. No published SDK surface beyond the importable PyPI package. Below pi's 6 (pi has
  SDK docs + mature rpc docs); above crush's shape because tau has an actual documented second
  frontend contract. 5.

## Operability (6)

- JSONL session storage with crash-deterministic temp naming ("A fixed temp name makes crash
  recovery deterministic", storage.py:78-99); session tree entries with parent ids and
  `path_to_entry` (tau_agent/session/tree.py:12-22); resume via `--session` with a helpful
  rename shim for `--resume` (cli.py:405-407) and "To resume this session:" guidance (cli.py:584).
- Session export (session_export.py 1,780 LOC), usage/stats accounting, diagnostics.py, update
  checker + updater.
- No checkpoint/revert, no fork journals, no documented crash-log posture. nanocoder rung. 6.

## Originality (3)

Deliberate and honest derivative: "Pi-compatible", "Pi-style" throughout (loop.py:1,
context_window.py:174, harness.py:1); AGENTS.md states the goal is porting pi's architecture.
Real added substance, but incremental: response timing instrumentation
(first-output/total duration persisted, loop.py:243-280), provider-anchored receipt accounting,
fail-safe hook blocking, llama.cpp builtin extension, models.dev catalog loader, updater. Nothing
here should be copied that pi (or codex) doesn't already do better; the port itself (typed Python,
pydantic wire models, FakeProvider streams) is the value. 3.

## Durability (4)

Single contributor (census; consistent with Discord-personality project), shallow clone so commit
volume unknown, but HEAD 6 days old, PyPI publish workflow, SHA-pinned CI, HF-org hosting, 32-page
docs site built in CI. Young, single point of failure; no SECURITY.md. nanocoder rung.

## Docs-dx (5)

Strong: website/content has 32 pages incl. internals (agent-loop.md, architecture.md,
design-principles.md, custom-frontend.md), reference (cli, configuration, rpc, slash-commands,
tools), guides (project-trust, sessions, skills), and the Hugo build is CI-checked (ci.yml:53-83).
README install path clear; CLI gives actionable resume hints (cli.py:405-407). Slightly below
pi's 9 -- less volume, no session-format spec depth observed -- docs are excellent but thin: 7.

## Totals

| dim | score | weight |
|---|---|---|
| architecture | 7 | 15 |
| verification | 7 | 15 |
| safety-enforcement | 6 | 10 |
| token-economy | 7 | 10 |
| orchestration | 3 | 10 |
| interop | 5 | 10 |
| operability | 6 | 10 |
| originality | 3 | 10 |
| durability | 4 | 5 |
| docs-dx | 7 | 5 |

**weighted_total 56.5, band C.** Strongest: architecture/verification/token-economy (7, three-way).
Weakest: orchestration and originality (3, tie; orchestration cited as weakest because the gap is
functional, originality's gap is intentional). Not within 2 pts of any band boundary.

Calibration: no rule (a) exposure -- tau is not a sync-fork of pi (independent Python codebase
porting pi's architecture; no vendored pi source found). No demotions applied.
