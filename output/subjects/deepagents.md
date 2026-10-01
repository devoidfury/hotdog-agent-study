# deepagents — T2 deep review

Subject: deepagents (langchain-ai/deepagents, MIT) | Tier: T2 | Reviewer lane: static read-only
Census row: Python, non_test 279,143 / test 471,723, contributors 1, head 2026-09-27, shallow=true, provenance "original".

## Anchor question

**Which anchor subject is this closer to, and why?** pi — both are harness-first codebases whose
enforcement stops at tested approval policy with documented honesty about what is underneath
(nothing), with property-named test corps; deepagents differs where pi does: institutional
backing, published packages, and eval infrastructure instead of an authored loop and session tree.

## What this codebase is

A monorepo: `libs/deepagents` (the SDK — middleware-assembled agent factory over langchain's
`create_agent`), `libs/code` (the `dcode` terminal coding agent product), `libs/acp` (ACP
connector), `libs/talon` (experimental channel host: WhatsApp/Telegram/Discord + cron),
`libs/evals` (tau2/tau3, BFCL, DRBench via "harbor"), `libs/partners` (daytona, modal, runloop,
vercel sandboxes; quickjs code-mode), `openwiki/` (38-file generated architecture wiki), 58 CI
workflows.

## Census sanity — CONFIRMED INFLATED, corrected

Reproduced the census exactly: `cloc` over every directory named `tests` sums to **471,723**, and
total cloc code (750,866) minus that equals the census non_test 279,143. So the census glob is
"everything under any `tests/` dir, all languages", which pulls in:

- **212,273 lines of JSON data** inside tests dirs — dominated by the vendored tau2-airline
  dataset `libs/evals/tests/evals/tau2_airline/data/db.json` (205,192 lines) plus
  `tasks.json` (3,521) and `libs/evals/tests/evals/data/benchmark_samples/bfcl_v3_final.json` (1,533).
- Real test logic: **~250,151 Python code LOC (cloc; 545 files) + ~7.3k JS** ≈ 257k. Still 1.5x
  the ~172k non-test Python code LOC. **Verdict: the "tests > prod" pattern is real, not fixture
  bulk** — but the headline 471k overstates test logic by ~45%.
- The census test glob also sweeps 30 task-fixture `tests` dirs under
  `libs/evals/datasets/context-retrieval-evals/cb-cloud-*/` (30 lines total; negligible).

Shallow clone (`.git/shallow` present, 1 commit): contributors=1 and commits=1 are shallow
artifacts. head_date 2 days old at review; no archived/dead claims made in either direction.

Provenance: original per census; no corpus fork candidates. Internal lineage noted:
`deepagents-code` "forked from `deepagents-cli` at v0.1.0" (libs/code/THREAT_MODEL.md:3) — both
live in this repo; irrelevant to rule (a). License MIT — clean for portables.

## T2 mandatory line-level reads

### Core loop (framework)

The ReAct loop itself is *upstream* — `create_deep_agent` assembles a middleware stack and hands
off to langchain's `create_agent` (libs/deepagents/deepagents/graph.py:964-984). What deepagents
owns is the stack and its ordering contract:

- Skills → Filesystem → SubAgent → Summarization → PatchToolCalls → [user MW] → profile MW →
  prompt-caching → Memory → HITL → UnsupportedContent (graph.py:843-960, documented at 333-372).
- `_REQUIRED_MIDDLEWARE` (graph.py:246-268): FilesystemMiddleware and SubAgentMiddleware cannot
  be excluded via `HarnessProfile.excluded_middleware` — exclusion raises `ValueError` rather
  than yielding "a silently degraded agent" (comment graph.py:250-256). Security scaffolding is
  tamper-evident by construction.
- `DeepAgentState.messages` uses `DeltaChannel(_messages_delta_reducer, snapshot_frequency=50)`
  "to reduce checkpoint growth from O(N²) to O(N)" (graph.py:74-77; reducer
  `_messages_reducer.py:31-60` — dedups by id, tombstones via `RemoveMessage`, skips
  `convert_to_messages` on the fast path, deliberately does NOT assign ids because replay would
  diverge — the docstring at `_messages_reducer.py:12-17` shows someone thought about replay
  determinism).
- `recursion_limit: 9_999` (graph.py:976) — effectively no loop ceiling at the framework level.
- Fork-mode subagents (experimental): inherit parent conversation and *rebuild* the parent's
  system prompt by mirroring prompt-producing middleware in the parent's order
  (graph.py:661-761). Cache-prefix discipline extends into subagent design.

### Compaction (summarization.py, 2,315 LOC)

Three tiers plus a recovery ladder, all in one middleware:

1. **Tool-arg truncation** at a low threshold (`TruncateArgsSettings`, summarization.py:171-196):
   clips `write_file`/`edit_file` args before the keep window only.
2. **Fraction-triggered summarization** with AND-clause triggers `TriggerClause` (tokens/messages/
   fraction, :150-161); defaults computed from the model's profile `max_input_tokens`
   (`compute_summarization_defaults`, :257-286: 0.85 trigger / 0.10 keep when profiled, else a
   conservative 170k tokens / 6 messages for unprofiled models).
3. **Overflow recovery ladder**: if the provider throws a recognized context error, the request
   falls back to summarize + clip large trailing tool results and gets *exactly one* budgeted
   retry, then raises `ContextOverflowError` rather than looping
   (docstring :1502-1510; `wrap_model_call` :1542-1548 `_is_context_overflow` catch;
   `_check_reduction` :1415-1423; `_overflow_clip.py` 206 LOC).

Support machinery: full evicted history is offloaded to `/conversation_history/<session>.md`
*before* summarizing so eviction is reversible ("Offload to backend first so history is preserved
before summarization. If offload fails, summarization still proceeds" with a loud
`warnings.warn` that older messages are unrecoverable, :1571-1580); base64 media blocks are
extracted to files and re-referenced by `<image path/>` tags with a failed-offload placeholder
that counts toward an aggregate warning (:1092-1227, `_OFFLOAD_FAILED_PLACEHOLDER` :296-299);
token counting includes tool schemas and probes the counter's signature rather than probing-by-call
(:226-254); one shared count per turn because "tool-schema conversion makes each count expensive"
(:1525-1526). `SummarizationToolMiddleware` exposes `compact_conversation` as an agent/HITL-callable
tool with eligibility thresholds (:1950-2270).

### Permission / safety code

Framework (`libs/deepagents`):

- `FilesystemPermission` rules — allow/deny/interrupt, first-match-wins, default allow
  (filesystem.py:354-369). Enforced per tool call for the 8 built-in fs tools
  (read/write checks at filesystem.py:1866-2273) and enforced *conservatively* on recursive
  delete: a deny pattern anywhere in the ancestor chain or on wildcard-overlapping descendants
  blocks the delete, with the conservatism documented (:502-587).
- Interrupt-mode rules are compiled into `when` predicates for langchain's
  `HumanInTheLoopMiddleware` (`middleware/_fs_interrupt.py`): tool calls get "exact" vs "bulk"
  scope, and a pathless bulk call (`grep(path=None)`) fires unconditionally for any
  interrupt-mode rule because it "can touch anything" (:24-31); glob's `pattern` arg can redirect
  the search root and is gated separately (:36-46).
- **The honesty centerpiece**: permissions + an execution-capable backend together raise
  `NotImplementedError` at construction — "Tool-level permissions for the execute tool are not
  implemented. Either remove permissions or use a backend without execution support."
  (filesystem.py:1792-1801). Rather than ship a policy that `execute` silently bypasses, the
  combination fails closed at build time.
- `THREAT_MODEL.md` (pinned: "Generated: 2026-03-28 | Commit: e859077f") states scope, out-of-scope,
  and 8 assumptions including "the library does not provide OS-level process isolation"
  (assumption 5) and the OpenAI-responses-data-retention caveat (assumption 8).

Product (`libs/code`):

- `auto_mode.py` (4,452 LOC) — three-path approval: `deterministic` policy allow/deny →
  `classifier` (LLM batch review of tool-call batches, `_ClassifierVerdict`/`AutoDecisionBatch`
  :298-343) → `fallback` to human. Dispositions: `deterministic_allow | classifier_allow |
  policy_deny | classifier_unavailable | require_human` (:467-491). The middleware subclassing
  HITL is at :2370.
- Anti-oscillation and availability logic: consecutive-denial (3) and consecutive-unavailable (2)
  counters force a human ask (:154-156); a classifier spec that once failed to build is *latched*
  so later batches route straight to human instead of silently denying forever — the docstring
  explains why a counter alone oscillates deny/deny/ask (:449-456). Separate construction vs
  verdict deadlines because "the first batch was likeliest to be denied, and reported as 'the
  classifier did not respond' for a model that was never built" (:145-150).
- Tests assert the nasty properties, not existence (132 in `test_auto_mode.py`): "irreducible
  context overflow fails closed" (:742), "counter write failure is not reported as classifier
  approval" (:2339), "classifier uses only trusted user metadata" (:2678), "classifier accepts
  only selected same-turn ask_user answer" (:2818), "omitted later answer withholds older
  consent" (:3012), "yolo routing leaves a hook-denied row paused" (:1069).
- Second `THREAT_MODEL.md` for the product, pinned 2026-08-19.

### Tests & CI (beyond counts)

- Faux-provider e2e: `GenericFakeChatModel` (tests/unit_tests/chat_model.py:21-50, streaming
  chunking configurable, call recording) drives *real* `create_deep_agent` graphs end-to-end;
  async-subagent loop verified against a mocked LangGraph SDK asserting `async_tasks` state
  transitions (test_end_to_end.py:3441-3500). 126 e2e tests, 133 graph tests, 124 permission
  tests with param'd deny-pattern geometry (test_permissions.py:440).
- Prompt-assembly golden snapshots: `tests/unit_tests/smoke_tests/test_system_prompt.py` snapshots
  the assembled system prompt against fake models — prompt/cache-prefix regressions are pinned.
- CI: `ci.yml` runs lint+unit tests for changed packages on every PR, full on main/merge_queue
  (ci.yml:11-15); even the label-driven warnings-bypass fails closed when the labels API call
  errors ("the strict ripgrep install will be ENFORCED", _test.yml:135). Nightly cron = CodSpeed
  perf benchmarks only (_benchmark_nightly.yml:8-10). **Model evals are `workflow_dispatch`-only**
  (evals.yml:36) — best-in-class infra (per-provider matrices, N-trial aggregation with 6h budgets
  per trial `evals_trials.yml`, branch-comparison runs and frozen "lite" task sets
  `unified_evals.yml:19-30`) wired to no PR/cron trigger. No fuzzing anywhere (zero hypothesis use).

## Dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 7 | Clean protocol split in SDK (backends/protocol.py:886 SandboxBackendProtocol; middleware-as-composition; graph.py assembly with tamper-evident required-stack guard :246-268); client/server process split in product (libs/code/ARCHITECTURE.md:14-31). Docked: the loop is upstream langchain `create_agent` (graph.py:964), and the product has a 31,203-LOC god file `deepagents_code/app.py` (per errata, >5k product-mode file docks both sides — this is 4.5x pi's worst). |
| verification | 8 | ~257k real test-logic LOC (census-corrected); adversarial property names (test_auto_mode.py:742,2339,2818,3012); faux-provider full-loop e2e (test_end_to_end.py:3441-3500); golden prompt snapshots; nightly perf benchmarks. Ceiling per errata: evals not CI-triggered (evals.yml:36), no fuzzing. |
| safety-enforcement | 7 | Above the tested-approval 6 rung: deny-policy with conservative recursive-delete semantics (filesystem.py:502-587), bulk-interrupt predicates (_fs_interrupt.py:24-46), fail-closed construction guard on the permissions↔execute gap (filesystem.py:1792-1801), classifier ladder with latched-failure + fail-to-human counters (auto_mode.py:154-156,449-456) and 132 tests. Below codex: zero OS-level enforcement in-tree; isolation delegated to partner packages or the user (THREAT_MODEL.md assumption 5). |
| token-economy | 8 | Three-tier compaction + overflow ladder (summarization.py:171-196,257-286,1542-1548), reversible offload with honest degradation warnings (:1571-1580), media offload (:1092-1227), cache-prefix-ordered stack (graph.py:904-905), cache_control preserved through prompt assembly (graph.py:380-383) and asserted (test_graph.py:723), cost tracking on an auto-updated price catalog (cost_tracking.py:35-43; bump_genai_prices.yml). No cache warming/stats (pi) and no no-LLM fresh-window tier (codex) → 8 not 9. |
| orchestration | 7 | Sync `task` subagents + experimental fork (graph.py:661-761), async remote subagents with start/check/update/cancel/list (test_end_to_end.py:3447+), rubric-gated completion loop (middleware/rubric.py:1-10), talon cron scheduler. No queue primitive, no loop breaker, recursion_limit 9_999 (graph.py:976). |
| interop | 8 | ACP connector + `dcode --acp` one-command server (libs/acp/README.md:1-25); MCP with per-server allowedTools/deny validation (mcp_tools.py:533-549,805) and 4,284 LOC of auth tests; headless `-x` mode sharing the interactive runtime (ARCHITECTURE.md:41-45); PyPI packages deepagents/deepagents-code/langchain-quickjs; Zed via ACP. No own IDE plugin, no published rival-interop SDK beyond packages. |
| operability | 8 | Sqlite-checkpoint session threads (sessions.py:1), state-dir hardening (:20 `harden_state_dir`), config-generation semantics with `/reload` and never-erase-on-parse-failure (ARCHITECTURE.md:44-50), `dcode doctor`, graceful-exit teardown tested on every exception branch (test_app.py:~15007-15036), 71 session tests. |
| originality | 7 | Latched-failure classifier ladder (auto_mode.py:449-456), O(N) checkpoint delta reducer (graph.py:77), rubric grader middleware (rubric.py), quickjs code-mode middleware with no fs/net/stdlib by default (libs/partners/quickjs/langchain_quickjs/_prompt.py:25-29), harness profiles, openwiki generated-docs pipeline (openwiki-update.yml). Real mechanisms, none field-defining. |
| durability | 9 | langchain-ai institutional backing, MIT, 58 workflows incl. release-please fanout watch, dep-freshness/pin bumpers, PR governance automation; fresh HEAD (shallow clone — activity beyond head_date treated as evidence-limited, not asserted). |
| docs-dx | 8 | openwiki 38 files (architecture/concepts/testing/workflows) + per-lib ARCHITECTURE/DEVELOPMENT/HOOKS/EXTENSIONS + two pinned THREAT_MODELs + 16 examples. Deeper product docs live off-repo (docs.langchain.com) — that cost codex a point at 7; in-repo substance here is much greater than codex's 206 lines. |

**Weighted total: 76.0 → band B.** Strongest dimension: **verification** (8, the largest genuine
property-testing corpus in the corpus so far and census-corrected). Weakest: **architecture**
(7; upstream-owned loop + a 31k-LOC product god file).

## Calibration notes

- **Boundary risk: 76.0 is within 2 of the 78 A-boundary.** If synthesis considers the three-tier
  compaction + cache-ordered stack worth token-economy 9, the total reaches 77; A would need
  safety 8 or verification 9, and the errata's verification-9 bar (in-CI evals or fuzzing) is
  objectively unmet (evals.yml:36 is dispatch-only).
- Census correction applied (test_loc inflated ~45% by vendored JSON fixtures under tests dirs).
- Shallow clone: contributors=1 is an artifact; durability scored on workflow governance and
  institutional backing visible in-tree, no activity claims beyond head_date.
- No fork/demotion adjustments; MIT license, no copy restrictions.
