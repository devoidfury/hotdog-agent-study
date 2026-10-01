# ouroboros — T2 deep review

Subject: /data/samples/agents/ouroboros (razzant/ouroboros, v7.5.1, Python, MIT).
Self-describes as a "self-creating AI agent": an agent that rewrites its own repo under a
constitution (BIBLE.md), runtime-mode tiers, and hermetic commit gates. Tier T2 per census.

## Provenance & activity

- Original, not a fork: GitHub API `fork:false`, license MIT, created 2026-02-11,
  pushed_at 2026-09-29 (remote-verified). 1391 stars, 644 forks, 305 open issues.
- Local clone is shallow (`.git/shallow` present, commits=1). Census `contributors: 1,
  commits: 1` is a shallow-clone artifact; per protocol no inactivity/low-history claim is
  made. HEAD itself is fresh (2026-09-27).
- `sync-joi-lab-fork.yml` is an OUTBOUND mirror push to the owner's own mirror, not an
  upstream sync. No calibration rule (a) concern.

## Census oddity: test LOC 536k > prod 335k — verdict: real, not fixture bulk

- `tests/` = 1,427 `.py` files, 582,770 raw lines (census 535,975 consistent after
  blank/comment exclusion). Non-py content in tests: 18 json, 13 sse, 3 fixture, 2 md --
  there is effectively no fixture bulk.
- 1,378 `test_*.py` files, 17,571 test functions, ~79.5k `assert` statements, only 50
  `assert True`. The largest outlier (`tests/test_devtools_benchmarks.py`, 326 KB) is a
  harness test suite that CI actually runs (`.github/workflows/ci.yml:213`), not vendored
  benchmark data.
- Tests prove behavior, not existence. Samples: real-git-tree snapshot semantics with
  sensitive-file veto and untouched-target assertions
  (`tests/test_delegated_run_isolation.py:52-70`); mode-elevation consent refusals and
  downgrades (`tests/test_runtime_mode_elevation.py:87-147`); data-write blocks on
  `settings.json`/grants (`:175-192`); env-scrubbing edge cases
  (`tests/test_v647_megacommit.py:21-37`). Trivial import-only patterns (237 importlib
  mentions) are a small minority.
- Prod side: `ouroboros/` 293k + `supervisor/` 36k + scripts ~9k raw ≈ census 334,735.
  Census row stands, no correction.

## Anchor question

Before scoring: this is closer to **cline** -- both pair a broad product surface with
tested in-process policy and no kernel-level boundary -- but Ouroboros adds a codex-class
verification tail (live-model CI lanes) that cline lacks, and falls short of codex/pi on
enforcement depth and institutional durability.

## Line-level reads (T2 mandatory)

### Core loop — ouroboros/loop.py (886 lines) + 13 loop_*.py leaves (~15.5k)

Single `while True` round loop at `ouroboros/loop.py:456` orchestrating: drain of owner
messages, delegate-hold stepping, early-finalize rails (grace/deadline/cost), pending
budget tails (`loop.py:547-568`), per-round compaction before send (`loop.py:578-590`),
transport-wait reconciliation, cross-model fallback chain (`loop.py:629-648`), tool
execution via `handle_tool_calls`, and typed exception exits (`BudgetExceeded` vs generic,
`loop.py:672-683`). Round identity survives waits: `free_redial` keeps a paid round from
double-billing (`loop.py:457-459`). Resume paths (`load_owner_wait`/`load_budget_pause`,
`loop.py:387-420`) restore model/effort/round_idx/tool-schemas and close unanswered tool
calls without replay. Terminal handling is exhaustive and honest: `provider_outcome_unknown`
never resends without a wake receipt (`loop.py:160-260`).

Docking: the leaf split is real but the seams are held together by a deliberate
historical-facade re-export surface -- `loop.py:686-886` re-exports ~150 private names
explicitly so "tests address these historical loop bindings" and "sibling leaves resolve
their call-time handles through this module". Tests are coupled to private names via
monkeypatch through the facade; state flows through a mutable `ctx` attribute bag
(`ctx._presence_completion`, `ctx._accumulated_usage`, underscore attrs across modules).
This is a complexity tax the anchors (codex crates, pi layers) do not pay.

### Compaction / context economy

- `ouroboros/context_compaction.py`: checkpoint-bound "context-reclaim materialization"
  -- atomic units, per-unit `summary_budget_tokens`, structured `emit_context_summaries`
  tool contract (`:46-70`), canonical-JSON sha256 transcript identity (`:98-104`),
  negative memos (`:667`), persisted reclaim checkpoints (`:746`), fold-and-resplit part
  machinery (`:543-626`). Summarizer guidance demands late facts and exact errors
  (`:38-45`).
- `ouroboros/transcript_prefix.py:1-16`: the append-only byte-prefix invariant between
  sends of one execution, explicitly tied to OpenAI-family prompt caches (issue #906,
  measured). Compaction seams stamp `sanction_rewrite` (`:32`); every unsanctioned break
  increments `prompt_prefix_breaks` in usage (`loop.py:355`) -- cache discipline that is
  not just practiced but metered.
- `ouroboros/context_fit.py`: task-local fit projections measured against what the model
  would actually receive; "Main fit owns automatic decisions"
  (`loop_round_limits.py:278-283` -- only manual reclaim runs per-round; automatic fit
  drives the rest).
- `ouroboros/_usage_cache_splits.py:1-10`: per-task prompt-cache split memo whose failure
  direction is conservative (re-price as fresh cache write, never under-reserve money).
- Tool-schema residency: Nano mode loads meta-only schemas, `list_available_tools` /
  `enable_tools` discovery never grants capability (`tool_policy.py:1-120`,
  `tools/tool_discovery.py:1-12`). Lazy-loading yes, capability separation clean.

### Permission / safety enforcement

- `ouroboros/tool_access_types.py:18-70`: closed vocabulary -- 6 profiles x 9 resource
  roots x 10 operations, one `_POLICY` matrix as SSOT; `decide_tool_access`
  (`tool_access.py:195`) is the single decision seam; subagent profiles summarized to
  parent+child at schedule time from the same matrix (`tool_access.py:88-117`).
- `ouroboros/tools/shell_guards.py`: beyond regex deny-lists -- AST analysis of inline
  interpreter code (`python -c`) to resolve write targets
  (`_python_write_targets_and_unknown:376`), opaque-call detection (`:361`), and a
  scoped fail-closed rule: unresolved targets block only when they COULD land in the repo
  (`_dynamic_write_could_hit_repo:1447-1480`, with the "fail-closed is right where the
  danger exists, not everywhere" comment).
- `ouroboros/runtime_mode_policy.py:1-6`: `advanced` may evolve the app layer but not core
  contracts/safety files; `pro` may, through the triad+scope review gate;
  `mode_has_unrestricted_agency` (`:33-41`) -- `cyber_pro` deliberately makes internal
  permission/review checks advisory-only. Elevation without explicit consent is refused
  and tested (`tests/test_runtime_mode_elevation.py:87-124`). BIBLE deletion and history
  rewrites get a small argv predicate (`runtime_mode_policy.py:47+`).
- No OS-level sandbox: no seatbelt/bwrap anywhere (grep only hits
  `ouroboros/config.py`). Delegated containment via the external Claudexor engine is
  macOS-only, and they say so explicitly with "Disclose instead of forbid"
  (`config.py:233-241`). Enforcement is in-process policy that the agent's own runtime can,
  at elevated modes, edit -- mitigated by consent gates, review gates, and honest
  documentation (`SECURITY.md`). This is the 6 rung ("tested approval/policy, nothing
  underneath"), not codex's 10.

## Verification (the standout)

Five-tier, 3-OS CI (`ci.yml:1-16`): parallel+serial pytest lanes with timeout and
worker-restart-zero (`ci.yml:157,160`), a `size_ratchet` blocking lane in every job
(`ci.yml:175,293`; `tests/test_size_ratchet_ci_shape.py:17`), empty-lane guards via
`--collect-only` for every marker lane (`ci.yml:471-510`), real-provider `integration`
marker lane (`ci.yml:342-371`), live skill-install smoke + paid reviewer flow at ~$1.2/run
(`ci.yml:385-456`), daily KEYLESS `tests/system_e2e` scenario lane (cron 04:37,
`ci.yml:8,571-582`), and a NIGHTLY PAID live-E2E stand with an explicit $30 budget cap and
honest skip without the key (cron 03:17, `ci.yml:9,67,631,657-660`). Playwright
chromium+webkit browser lanes (`ui-browser.yml:54-61`), OpenSSF scorecard workflow,
secret scanning via betterleaks in CI (`ci.yml:324`). Faux provider: stdlib
`MockLLMServer` on 127.0.0.1 (`tests/fixtures_mock_llm.py:16-54`) plus a provider contract
catalog (`tests/provider_contract_catalog.py`, `provider_contract_ci.py`).

Per the erratum, 9+ requires in-CI evals or fuzzing with workflow file:line. The live-model
lanes above are the evidence (skill-reviewer runs and the paid E2E stand drive real models
in CI nightly); no fuzzing and the benchmark harnesses (gaia, swe_bench, osworld in
`devtools/benchmarks/`) are NOT CI-gated -- that is why this is 9 and not 10.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7 | loop.py:456 clean round loop + enforced 1600-line module ceiling (ouroboros/review.py:19-23, CI lane ci.yml:175); docked for facade re-export surface (loop.py:686-886), ctx attribute-bag state, 2210-LOC registered giant (supervisor/workers.py) |
| verification | 9 | 17,571 test functions / ~79.5k asserts; live-model CI (ci.yml:9,443,631); empty-lane guards (ci.yml:471-510); faux provider (fixtures_mock_llm.py:16-54); no fuzzing, benchmarks not CI-gated |
| safety-enforcement | 6 | profile x root x operation matrix (tool_access_types.py:18-70), AST shell guard (shell_guards.py:376,1447-1480), consent-gated elevation tested (test_runtime_mode_elevation.py:87-124); NO OS-level boundary, cyber_pro makes checks advisory (runtime_mode_policy.py:33-41), macOS-only delegated containment disclosed (config.py:233-241) |
| token-economy | 8 | byte-prefix cache invariant + counted breaks (transcript_prefix.py:1-16, loop.py:337-355), checkpoint-bound reclaim with receipts (context_compaction.py:1030,746), cache-split memo (_usage_cache_splits.py:1-10), nano schema residency (tool_policy.py:96-110) |
| orchestration | 8 | supervisor queue state machine + reaper (supervisor/queue_transitions.py, task_reaper.py), in-loop resume journals (loop.py:387-420, owner_wait.py:207-217), worktree-isolated delegated snapshots (test_delegated_run_isolation.py:52-70), background-consciousness wakes with rolling-24h allowance (consciousness.py:1-30) |
| interop | 7 | MCP client (ouroboros/mcp_client.py) + MCP admin gateway (gateway/mcp.py), A2A chat-id ranges (gateway/host_service.py:25,338), web gateway (server.py:17-18), Android host app (android/host/), Telegram skill, extensions with isolated deps; no ACP, no published SDK |
| operability | 8 | events.jsonl journals + rotation with lock-aware archive (supervisor/state.py:676,1006-1044), stale-wait refusal on resume (owner_wait.py:215-217), budget-pause/owner-wait resume, safe_test retained-root honesty (scripts/safe_test.py:1-12), Docker/desktop/android builds |
| originality | 8 | constitution-governed self-modification with tiered source protection (BIBLE.md:7-14, runtime_mode_policy.py:1-6), CI-enforced module-size ratchet (review.py:19-23), acceptance-review panel with typed closed-set reasons + per-root paid cycle ceiling + identical-diff refusal (loop.py:113-119 comment, tools/commit_gate.py:1-27), metered prompt-prefix breaks; all verified in code |
| durability | 6 | solo maintainer (bus factor 1), young (Feb 2026), no institutional backing; but very active (pushed 2026-09-29 remote-verified), MIT, SECURITY.md + SUPPORT + CoC, 1.4k stars |
| docs-dx | 7 | 47+ docs incl. 13-part numbered architecture series (docs/architecture/), 14 governance docs (docs/development/), llms.txt, name-miss guidance with bounded inline catalogs (tool_policy.py:194-260); idiosyncratic voice, some external-pointer reliance |

Weighted total: 7*1.5 + 9*1.5 + 6 + 8 + 8 + 7 + 8 + 8 + 6*0.5 + 7*0.5 = **75.5** → band **B** (upper B).

- Strongest dimension: verification (9).
- Weakest dimension: safety-enforcement (6).
- Boundary check: 75.5 is 2.5 below the 78 A-boundary -- no boundary-risk flag required,
  but this is a band-ceiling candidate: any upward correction on architecture (if the
  facade pattern is judged acceptable) or safety would push it into A territory.

## Calibration notes

- Verification 9 per the erratum gate: in-CI live-model lanes exist with file:line
  (ci.yml:9,71,631 for the $30-capped nightly E2E; ci.yml:443 for the paid reviewer skill
  flow). Still no fuzzing and no scored benchmark gate in CI, so not 10. This is the first
  subject observed past the 8-ceiling noted for codex/cline/pi; if synthesis disagrees, the
  fallback is 8 (total 74, same B band).
- No census correction. Test LOC is a real corps, not fixture bulk (see census section).
- Shallow clone: activity claims made only from the remote (pushed 2026-09-29), not HEAD.
- License MIT file, API-confirmed; portables above are concepts, code copying still not
  recommended for MIT (attribution), but legally adjacent-porting is permissible.
