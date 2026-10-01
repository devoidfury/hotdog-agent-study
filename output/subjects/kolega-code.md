# kolega-code — T2 deep review

**Anchor question:** Closest to **cline** — the same "one canonical product-agnostic loop
serving many thin hosts, approval-bound safety with real tests but nothing kernel underneath,
big spec-shaped test corps" shape; kolega adds stronger workflow-resume semantics and cache
discipline than cline, and lacks cline's published SDK and four-IDE breadth. (crush is too
monolithic vs this module split; pi is too small-surface.)

**Line-level reads done (mandatory surfaces):**
- Core loop: `kolega_code/agent/baseagent.py:2552-2985` (`_agent_loop_stream`)
- Compaction: `kolega_code/agent/compression.py` (all 370 lines) + loop-side triggers
  `baseagent.py:2588-2622`
- Permission enforcement: `kolega_code/permissions.py` (all 566 lines) + gate at
  `baseagent.py:1479-1527`, execution at `agent/tool_backend/terminal_tool.py:456-535`

## Subject shape

Python, v0.40.0, `kolega_code/` 142.5k LOC (of which 30.6k is vendored `_bundled_skills`
synced from the same org's kolega-skills repo, Apache-2.0 headers, `scripts/sync_bundled_skills.py:20-22`),
`tests/` 121k LOC, ~4,800 test functions across 405 files. Positioning: local-first terminal
coding agent whose flagship ("Gigacode") is a model-authored multi-agent workflow engine:
the model writes a Python program against a `agent/parallel/pipeline/phase/budget` primitive
API and a curated sandboxed namespace executes it (`how-gigacode-works.md`, verified against
`agent/orchestration/runtime.py`, `executor.py`, `journal.py`).

## Core loop (read at line level)

One shared generator, deliberately message-independent (`baseagent.py:2555-2560`), used by TUI,
ACP, `ask`, and gateway hosts:

- Iteration cap is **optional** (`max_iterations: Optional[int] = None`, `:218`, enforced `:2571`).
- Swallowed-cancellation re-honoring: terminal tools consume `CancelledError` into structured
  `status="cancelled"` results; the loop re-checks `Task.cancelling()` at top and bottom
  (`_pending_turn_cancellation` `:2537-2550`, checked `:2568`, `:2978`) so a cancel cannot be
  laundered into a completed turn. Unusual care.
- Per-iteration measured token count on the **same** repaired history as the request
  (`:2578-2580` — comment notes the O(history) repair pass previously ran twice).
- Silent-turn guard: reasoning-only/empty `end_turn` gets escalating nudges, capped at 3
  (`:171`, `:2896-2932`), terminates with a surfaced `[no output]` notice, not silence.
  Truncated-turn guard: `max_tokens` with visible text gets continuation nudges capped at 3
  (`:177`, `:2940-2966`), then delivers the partial. Reset semantics are explicit (a
  non-silent / non-truncated response resets its counter, `:2820-2832`).
- Stop hooks may block a natural stop with a cap of 5 overrides (`:163`, `:2968-2975`).
- Journal-failure vs tool-failure separation: tool errors are recoverable by the model, but a
  session-persistence error is terminal and re-raised (`:2981-2983`, `:1620-1622`).
- DeepSeek server-side silent truncation detection at the wire output cap (`:2790-2803`).
- Prompt-cache checkpoint per iteration (`:2562` → `mark_cache_checkpoint` `:738-745` →
  `conversation.py:975-1009`: two rolling breakpoints one turn apart, everything else cleared).

## Compaction (read at line level)

Single strategy, but hardened unusually well:
- Trigger is a **measured** count against the resolved max input at 0.8 (`compression.py:171-174`,
  `baseagent.py:2588`), not a chars heuristic.
- Incremental: only the span aged out since the prior boundary is summarized, prior summary folded
  in (`compression.py:152-162`, `apply_compaction` boundary via `conversation.compaction_split_point`).
- The compaction request itself is budget-fitted: a descending ladder of rungs (100%→90%→80%…) of
  `max_input_tokens`, each rung re-widens a **middle omission gap** (half oldest / half newest,
  `_middle_gap` `:92-106`) that never splits a tool exchange (`snap_split_point` +
  `_bundles_tool_results` walk `:270-276`), with stall detection (`:277-280`) and no re-dispatch
  of an identical gap (`:287-290`). Rationale (an unfitted request reads as `llm_error` and
  permanently disables auto-compaction) is documented at `:196-203`.
- Empty-summary retry x3 for reasoning models that spend the whole budget on CoT
  (`:152-158` constants, `:307-334`).
- Loop-side zero-tail fallback: if the ordinary pass leaves the tail over budget, one aggressive
  pass with `keep_recent=0`, skipped when the failure was `llm_error` or `too_few`
  (`baseagent.py:2596-2612`); then **fail-closed** dispatch only under the paired
  `--context-window-tokens + --max-output-tokens` strict budget (`:2618-2622`).
- Raw history is never mutated; transcript rendering keeps tool results un-escaped for ~30%
  fewer tokens (`compression.py:36-41`).

Missing vs the 9-rung (codex): no no-LLM token-budget tier, no branch summarization, no cache
warming; threshold is fixed 0.8, not projected against the next request.

## Permissions / sandbox (read at line level)

- Modes: ASK | AUTO; **ASK is the CLI default** (`cli/app.py:211`). Gate at
  `baseagent.py:1479-1527`: a permission-callback exception **denies** (fail-closed,
  `:1491-1510`), denial returns a model-visible tool error; the tests prove the deny-on-everything
  semantics — `tests/acp/test_permissions.py:114` (reject), `:121` (cancelled outcome denies),
  `:128` (client error denies), `:139` (timeout denies), and rules persist only on
  allow-always (`:88`).
- Rule store: project-local `.kolega/permissions.json` with schema version + strict parse
  (`permissions.py:58-196`), private-dir writes (`:184-190`); kinds command/edit/mcp/message;
  eval cells shaped as `[py] <code>` so executable rules cover kernels (`:216-228`).
- **Safety hole (filed):** command rules match the whole command string by exact /
  `startswith(pattern + " ")` / first-shell-token (`permissions.py:299-305`), and execution is
  `asyncio.create_subprocess_shell(..., shell=True)` (`terminal_tool.py:497-503`). An approved
  executable rule like `git` also admits `git status && curl evil`; no metacharacter splitting,
  no per-sub-command approval. Mitigations that exist: an **opt-in** LLM pre-exec safety check
  (`terminal_tool.py:66` default False, `:96-153`, itself fail-closed on error) and the fact
  that ASK-mode users see the full command string before approving. This is the cline-crush-class
  "approval binds, nothing underneath" hole at the rule layer, not a fail-open default.
- Hooks carry trust scoping: project `.kolega/hooks.json` loads only after project trust,
  otherwise surfaces as a diagnostic (`hooks/config.py:4-5`, `:169-196`).
- Workflow scripts: restricted builtins (no import/open/eval/exec/time/random), depth cap 1
  (hard max 2), 1,000-agent lifetime cap, 4,096 fan-out cap, concurrency 8 with jittered stagger
  (`orchestration/runtime.py:33-50`, `executor.py`), plan-mode forces read-only workers; the
  docs say outright the namespace is "a soft sandbox, not a security boundary"
  (`how-gigacode-works.md`) — good honesty. But local exec of terminal commands has **no OS
  sandbox**: `sandbox/` is a cloud-provider (E2B) abstraction and `LocalSandboxManager` admits
  "In local mode, we don't actually use sandboxes" (`sandbox/local.py:13-14`). No kernel
  enforcement anywhere in the local path → ceiling at the 6 rung.

## Verification

CI (`ci.yml`): PR+push, 4-Python matrix (3.11-3.14) `:53-58`, pre-commit, `run_tests.sh` with
`-m "not slow"` fast lane and `--cov-fail-under=45` (`run_tests.sh:13,63-64`), coverage-badge
automation with a token-verification step (`:180-188`), pip-audit of `uv.lock` with per-advisory
documented acceptances tied to wheel-availability reasons (`:256+`, rationale in
`pyproject.toml:30-44`), eval-bundle recompile-diff + pip-audit (`:237-255`), CodeQL workflow.
Test corps asserts properties: the compaction-budget regression fake re-implements the
production run-budget check inside `stream` (`BudgetEnforcingFakeLLM`,
`tests/agent/test_compaction_request_budget.py:55+`); resume tests pin cache-invalidation
semantics (`test_resume_replays_cached_prefix` `:538`, `test_resume_reruns_after_change` `:553`,
`test_resume_reruns_when_max_agent_depth_changes` `:565`). ~4,800 test functions.
No in-CI model evals (the eval bundle is dependency-audited, not run), no fuzzing → verification
8 ceiling per errata holds.

## Orchestration

Gigacode is the standout: per-run artifact directory (script byte-for-byte, run.json,
journal.jsonl, transcripts, per-agent transcripts, `orchestration/journal.py:3-16`);
**content-keyed resume** — every completed `agent()` call is journaled under a `cache_key`
hashing prompt + semantically significant inputs, label/phase excluded, same-key calls replay
FIFO in issue order, so replay survives `pipeline()` order drift and script edits; replays are
re-recorded so resume-of-resume is exact (`journal.py:18-27`, `:44+`). Token budget with 75%/90%
warnings and bounded overshoot on in-flight calls; budget exhaustion cancels and drains children
rather than swallowing (`how-gigacode-works.md` runtime-API table, confirmed against
`accounting.py`, `runtime.py:127+`). Plus typed sub-agents (general/investigation/coder/browser),
cross-session peer messaging with recipient-scoped permission kinds (`permissions.py:29-34`,
`session/inbox.py`), goal mode with model-judged verdicts (`baseagent.py:1895-1913`), snapshots
with revert (`tool_backend/snapshot_tool.py`), git worktrees with filelock-protected clone-local
exclusion (`worktrees.py:18-27`). No queue/daemon surface at codex's level. 8.

## Interop

ACP server as a full product surface (`acp/` — permissions bridge, plan approval, session
restore, sub-agent transcripts; Zed/VS Code), MCP client with three transports + OAuth
(`mcp/transport.py`, `mcp/oauth.py`), headless `ask` with JSON output, a gateway daemon with
Telegram adapter + STT (`gateway/adapters/telegram`, `gateway/stt.py`), web session server
(fastapi stack pinned as a core dep, `pyproject.toml:53-58`), browser extension dir. No MCP
server mode, no published SDK. 8.

## Token economy

Measured-count trigger, incremental compaction, two rolling cache breakpoints with a
4-breakpoint-budget contract test (`tests/agent/test_cache_breakpoints.py:1-9`), volatile
context separated into its own user turn so operator churn doesn't invalidate the cached
prefix (`baseagent.py:2516-2535`), skill-catalog metadata budgeted at 2% of the context window
with truncation reporting (`cli/skills.py:33,50-57`), usage ledger + per-call origin tagging
(`llm/ledger.py`, `helper_origin` at `compression.py:127`). No warming, no LLM-free tier, no
branch summarization. 7.

## Operability

Session list/resume/export, `doctor` (`cli/main.py:196`), turn recording that journals the
rendered system context and tool schemas deduplicated by fingerprint
(`baseagent.py:2491-2504`), workflow resume across process restarts, snapshots/checkpoints,
worktrees, settings panel, migration doc. No fork/clone/queue verbs. 8.

## Originality

Content-keyed workflow resume (not positional) is the most defensible resume semantics in the
corpus so far; in-kernel tool bridge — persistent Python/JS kernels per session, shared with
sub-agents, `tool.<name>(args)` from inside the kernel routes through the executing agent's
standard permission-gated tool path (`agent/eval/kernel.py:1-11`); `[py]` command-rule shaping
so one executable rule allowlists a language's cells (`permissions.py:216-228`); recipient-scoped
outbound-message permissions; tool-subject secret redaction for transcript rows
(`tool_subjects.py`). Model-authored orchestration itself is converging industry-wide; the
journal discipline is the genuine delta. 8.

## Durability

Corporate licensor (KLG Tech Innovations Limited), trademark-clean BUSL boilerplate, v0.40.0,
92k CHANGELOG, release/nightly-style workflows, dependabot active (HEAD is a dependabot merge,
PR #696), SECURITY.md + CodeQL + private vuln reporting. Census "contributors: 1, commits: 1"
is a shallow-clone artifact — HEAD is `228b174 Merge pull request #696`, so real commit count
is ≥ hundreds; head_date 2026-09-25 is fresh, no remote check needed under the 10-month rule.
Bus factor unmeasurable from this clone; single-org governance, unknown team size. 7.

## Docs / DX

`how-gigacode-works.md` is a 23k architecture doc that cites file paths and admits limitations
(soft-sandbox honesty, schema-conformance caveats); Astro docs site (`docs/`), MIGRATION.md,
CHANGELOG, curl install script, doctor command, trilingual? (no — single language) . Real but a
chunk of reference docs live on the built site rather than in-repo. 8.

## License adjudication (dispatcher task)

**There is no file-vs-manifest conflict; the census misread the file.** The `LICENSE` file is
Business Source License 1.1, first line (`LICENSE:1`), with Licensor KLG Tech Innovations
Limited, an Additional Use Grant permitting production use except competing hosted/embedded
offerings, **Change Date 2030-08-12** and **Change License "GNU Affero General Public License,
Version 3.0 or later"** (`LICENSE:27-29`). The census's "AGPL-3.0 (file)" is almost certainly a
tool matching that Change-License line as if it were the file's license. `pyproject.toml:14`
(`license = "BUSL-1.1"`) therefore *agrees* with the LICENSE file. What binds: BUSL-1.1 now, for
each version until its change trigger (Change Date or 4 years after that version's first public
distribution, whichever is earlier — `LICENSE` Terms ¶3), after which that version becomes
AGPL-3.0+. The manifest field alone would not bind; the LICENSE file's grant does, and they are
consistent. Consequences for us: non-open-source today → concepts-only, effort_for_us values
assume clean-room, no code copying; post-conversion AGPL still requires derivative-source
obligations. Filed as `kolega-code-1` (license-risk). No leaked-snapshot indicators: repo is
self-describing, upstream remote consistent, vendored skills are same-org Apache-2.0 with a
provenance-manifest sync script.

## Census corrections

1. `license`: should read "BUSL-1.1 (LICENSE file + manifest); AGPL-3.0+ only as post-2030
   Change License" — the AGPL attribution is wrong (see above).
2. `test_loc: 171,982` overcounts: `tests/` is 121,092 (`find tests -name '*.py' | xargs wc -l`);
   the extra ~51k likely comes from `_bundled_skills` smoke_test.py files and benchmark fixtures.
   `non_test_loc: 135,122` mixes 30.6k of vendored `_bundled_skills` into first-party code
   (~112k first-party non-test).
3. `contributors: 1 / commits: 1` is the shallow-clone artifact; HEAD being PR #696's merge
   disproves low activity. Do not read bus-factor 1 from this row.
4. Tier T2 stands (112k first-party + 121k tests).

## Boundary note

Weighted total 76.5 — within 2 of the 78 A-floor. What holds it at B: safety-enforcement 6
(no local OS sandbox, compound-command rule bypass) and token-economy 7. If synthesis weighs the
tested cache discipline + measured-trigger compaction toward 8, this crosses to A; the safety
rung is the honest anchor, so I keep it B and flag rather than silently demote/promote.

**Strongest dimension: orchestration. Weakest: safety-enforcement.**
