# Boundary re-review: opensquilla (T2-depth independent pass, read-only)

Run: independent boundary re-score of main T3 report (`subjects/opensquilla.md`, 77.5/B).
Subject HEAD d27ba0e (shallow, `.git/shallow` present; no dead-activity claims made from
HEAD, all activity evidence from remote API at review time). Subject never executed.
Main report read for calibration context only; every score below is backed by my own
file:line reads listed here.

**Anchor sentence:** Mechanically closest to **codex** — the only anchor whose combination of
three-tier compaction with a no-LLM fallback, durable reserved queue with tested restart
recovery, MCP on both sides, and three-platform kernel jails opensquilla matches — while the
shipped owner default posture is the **nanocoder** errata shape (real jail, shipped off, docs
claiming otherwise), so on the weighted profile the aggregate sits on **cline** (B ceiling).

## Census recount (errata class)

My counts: python `src/` = 567,497 LOC; python `tests/` = 723,277 LOC across 1,516
`test_*.py`; TS/JS (excl. node_modules) = 369,337. Census says 1,110,865 non-test /
646,597 test: test LOC undercounted ~77k, non-test includes generated
`src/opensquilla/contracts/generated/v4/` (425 modules, counted) and webui trees. Direction
does not change T3 size class; recorded as `opensquilla-b3`.

## Provenance

Original. Remote API at review time: `TokenRhythm/opensquilla` id 1231170332, `fork:false`,
`archived:false`, created 2026-05-06, pushed 2026-09-30 (day of review), Apache-2.0,
org-owned, 7,065 stars. The census `upstream_remote` (opensquilla/opensquilla) resolves to
the same repo id (org migration, not a fork); corroborated in-tree at
`src/opensquilla/observability/update_check.py:58-61` which hardcodes TokenRhythm release
URLs and labels the old URL `LEGACY_V1_RELEASE_TAG_PAGE`. Calibration rule (a) not
triggered; archived/dead B-cap not applicable.

## Dimensions (my scores)

### architecture — 7 (agree)
`Agent._turn_generator` spans agent.py:5829 (`async def _turn_generator`) to ~12650 — a
single ~6,820-LOC method inside a 16,121-LOC / 245-method `Agent`
(`engine/agent.py`; method count via def-grep = 245). That is >2x codex's tolerated residual
(turn.rs 3,167) and the pi-errata >5k-LOC docking factor lands on the loop itself
(`god-file-loop`). Supporting cast is also heavy: storage.py 15,172, runtime.py 14,779.
The `engine/turn_runner/` split is real (8 stage classes) but half-landed: harness.py's own
docstring says the orchestrator "can be introduced after the stage boundaries are ready",
and harness.py alone is 2,043 LOC with 34 `_accepts_keyword_arg` signature-sniffing sites
(worse in aggregate across the stage files). Why still 7 and not lower: outside the loop the
boundaries beat crush's shape (stages are Protocol-shaped, no parallel re-implementations);
why not 8: the pi-errata explicitly bars the 8->9 "loop separated from product surface"
reading, and crush's 7 rung (loop fused in `agent.go:2393`) is exceeded 2.8x here.

### verification — 8 (agree; ERRATA ceiling)
723k test LOC, 1,516 test files; `tests/test_sandbox` alone = 1,691 `def test_`. Tests prove
properties, not existence — I read them: `test_command_policy.py:44-93` covers quoted-control
heredoc splits, comment-hidden heredoc markers, and arithmetic-shift evasion
(`: $((1 << EOF))`) against the shell segmenter; `test_unavailable_backend.py` asserts
no-host-replay-on-denial (pending queue stays empty, no host retry requested). CI: real
bubblewrap install + raw namespace probe + python readiness probe on every PR
(`ci.yml:811-826`); unconditional `dependency-audit` job (`ci.yml:33-66`,
`audit_dependencies.py` with coverage assertions). Not 9 per the frozen errata: no in-CI
model evals (llm-e2e.yml `"on": workflow_dispatch` only, :3-4; no eval job in 18 workflows),
no fuzzing (`tests/test_tools/test_dispatch_properties.py:15-16` documents hypothesis
absent; cases hand-rolled). 8 is the observed ceiling, correctly awarded.

### safety-enforcement — 5.5 (agree)
Depth is real and I verified the machinery: backend selection refuses implicit noop when
sandbox is on (`sandbox/backend/__init__.py:158-160` raises `SandboxBackendError`);
bubblewrap/seatbelt/windows-ACL+WFP backends exist as shipped code (`sandbox/backend/`
seatbelt.py, windows_default_acl.py, windows_default_wfp.py, linux_*.py); guest runs are
structurally pinned to `RunMode.SAFE` (`sandbox/guest_profile.py:46-48` `run_context` returns
`RunMode.SAFE, source="guest_safe"`); FULL mode for a non-host principal raises
`host_capability_required` (`gateway/boot.py:4200-4211`); denial ledger pauses autonomy
(`sandbox/governance.py:156,500,624`; `denial_threshold: int = 3` config.py:103). But the
owner default: `run_mode: RunModeName = "full"` (`sandbox/config.py:95`) and the shipped
config contract self-describes the posture — `opensquilla.toml.example:512-513`
"Both default to false for the out-of-box bypass posture. sandbox = false / security_grading
= false", :532 `default_mode = "bypass"` — and `run_mode.py:28` collapses `"bypass"` to
`RunMode.FULL`, contradicting the same example file's promise that bypass "still blocks
sensitive paths". `docs/sandbox-security.md:10` ("Fresh installations default to Safe mode")
is flatly false against the shipped example. Dead injection refusal confirmed beyond the
main review: `_check_injection_guard` (`tools/dispatch.py:419-441`) no-ops unless
`origin_trace` is set, and provider-layer grep for `origin_trace` writes = 0 hits (only
copy-through at agent.py:15565,15584 and dispatch internals). Per the nanocoder errata
("real machinery + wrong default = below the 6 rung") this cannot reach 6; the docs
contradiction (honesty-about-non-enforcement is explicitly part of this dimension's
definition) blocks 5.5 -> 6 even harder. Strictly above nanocoder's 5: nothing fails open
silently — denial no-replay and noop-boot-refusal are tested.

### token-economy — 9 (agree)
Verified against code, my own reads: replay-projected budgets with the rationale comment
("Budget/skip/cut decisions must measure what the model actually replays",
`session/compaction.py:804-811`, delegating to `estimate_entry_model_replay_tokens` :628);
compaction self-budgeting — `MAX_COMPACTION_LLM_CALLS` cap (`compaction.py:1376`),
`arm_compaction_deadline` (:410), `_compaction_quality_report` (:1030); no-LLM emergency
window whose docstring is the honest instruction: "Select a local request view; never
summarize or mutate session storage" (`engine/runtime.py:12807`). Cache discipline beyond
every anchor's frozen rung: in-loop cache-break monitor diffs the request snapshot against
response `cached_tokens` and warns `prompt_cache.break_detected` (`engine/agent.py:9684-9691`
verified verbatim; `check_response_for_cache_break`), proven-prefix keepalive
(`engine/prompt_cache_keepalive.py`). Held at 9, not 10: the routing headline (lightgbm +
onnxruntime real deps, `pyproject.toml:105,108`) has zero in-CI accuracy evidence — the
`llm_router_acc` marker is excluded from all three CI pytest invocations (`ci.yml:1150,1305,
1594`) and carries zero tests (grep: registration in `tests/conftest.py` only). A
field-defining 10 cannot ship its headline claim as a vacuous marker.

### orchestration — 8.5 (agree)
Durable task runtime: `PendingOverflowPolicy` with REJECT_NEWEST/DROP_OLDEST backpressure
and running-task eviction immunity (`gateway/task_runtime.py:1268-1297`); restart-orphaned
approval expiry (`gateway/boot.py:2573+` `_expire_restart_orphaned_approvals`); governed
subagents via shared `agents/limits.py:11 MAX_SPAWN_DEPTH = 3` consumed by
`engine/subagent.py`; provider-boundary repetition guard with delta-split immunity
documented and structured as a checkpointed character-stream detector
(`engine/repetition_guard.py:1-6`) — stronger than the crush 7-rung breaker. Below codex 9:
no agent-graph/message-board plane, no first-class queue CLI verb (only a dict key in
`cli/gateway_client.py:862`), and the half-landed turn_runner split leaves
economy-critical sequencing state in the legacy path.

### interop — 8.5 (agree)
MCP client (`src/opensquilla/mcp/`: client, sse, stdio, discovery) and server
(`mcp_server/bridge.py`); 425 generated v4 contract modules verified by directory count;
rival importers exist (`migration/openclaw.py`, `migration/hermes.py`,
`migration/opensquilla_home.py` 6,505 LOC); webui/desktop/cli/gateway surfaces. Below 9:
no ACP anywhere (grep over src = 0 real hits) and no published out-of-tree SDK — cline's
boundary 9 rests on exactly those two capabilities opensquilla lacks.

### operability — 8 (agree)
Turn-acceptance journaling verified in schema: `turn_ingress_receipts` table with
`CREATE UNIQUE INDEX ... uq_turn_ingress_receipts_request ON (source_scope,
request_session_key, client_request_id)` (`session/storage.py:1018-1035`) — no anchor does
this. Epoch fencing: `StaleEpochError` appears 8 times in storage.py (main said 7 raise
sites; trivially off, score-neutral). `session/manager.py:1481` `resume` docstring: "Load an
existing session; touch updated_at" — load-and-touch, not codex/pi fork/archive/tree
journals, which is the dock from 9. `cli/doctor_cmd.py` FixStep scaffolding present.

### originality — 8 (agree)
Corpus-first mechanisms I verified live in shipped code: ingress receipts + epoch fencing
(storage.py above), principal-tier SAFE pinning (guest_profile.py:46-48 + boot.py:4211),
proven-prefix keepalive + in-loop cache-break monitor (agent.py:9684-9691), no-LLM emergency
window (runtime.py:12807), delta-split-immune repetition guard (repetition_guard.py:1-6),
coverage-asserting audit CI. Held at 8 not 9: two flagship stories fail the real-in-code
test — router accuracy is an unexercised marker (b5) and the injection refusal headline
cannot fire (b1). The chassis itself (tiers, queue, protocol) is codex-shaped, not invented.

### durability — 6 (agree)
Remote at review time: created 2026-05-06 (~5 months), pushed day of review, fork:false,
archived:false, org-owned, Apache-2.0, 7,068 stars / 576 forks, 18 workflows incl.
signed/desktop/mirror release pipelines, SECURITY.md present and sane. Census
contributors=1 is the depth-1 artifact (main's remote contributor read: 1,245 commits by the
dominant account + ~25-account tail) — bus factor effectively one. Ordering holds:
nanocoder 4 < opensquilla 6 < pi 7.

### docs-dx — 9 (agree, with the main's no-double-charge argument accepted)
64 docs verified by recursive count. My own claim-to-code spot-checks pass: preflight ratio
0.85 in docs matches `context_budget.py:52` parse-default with 0.1-0.95 clamp, used at
`runtime.py:7174-7177`; approvals doc's Safe/Full mode vocabulary matches `run_mode.py`
normalization. I accept the main review's adjudication that the sandbox-security.md:10
Safe-default falsehood is priced once, in safety-enforcement (it is the same defect the 5.5
already pays for); charging docs-dx again would double-count. The README:702 tool-result
escaping overclaim is a distinct defect, filed as `opensquilla-b1` but — being one sentence
in marketing copy against a 64-doc corpus that otherwise matches code — not worth a 0.25
dock; noted, not scored.

## Agreement table vs main

| dimension | main | mine | delta |
|---|---|---|---|
| architecture | 7 | 7 | 0 |
| verification | 8 | 8 | 0 |
| safety-enforcement | 5.5 | 5.5 | 0 |
| token-economy | 9 | 9 | 0 |
| orchestration | 8.5 | 8.5 | 0 |
| interop | 8.5 | 8.5 | 0 |
| operability | 8 | 8 | 0 |
| originality | 8 | 8 | 0 |
| durability | 6 | 6 | 0 |
| docs-dx | 9 | 9 | 0 |
| **total** | **77.5** | **77.5** | **0** |

No dimension-level disagreements. Evidence corrections (non-scoring): StaleEpochError count
8 vs main's 7; census test LOC 723k vs 647k (b3); `docs/sandbox-security.md` Safe-default
claim is at :10 (main cited :8).

## Verdict: HOLD B (77.5)

Hinge dimensions named explicitly: **safety-enforcement** is the only single-move hinge
(5.5 -> 6 lands exactly on the 78 floor) and it is pinned by the nanocoder errata — I
confirmed the shipped posture from the example config's own words ("Both default to false
for the out-of-box bypass posture", opensquilla.toml.example:512-513,532), the bypass->FULL
collapse (run_mode.py:28), the docs falsehood (sandbox-security.md:10), and the unreachable
refusal path (dispatch.py:419-441 with zero origin_trace producers). **Token-economy** is
the second hinge (9 -> 9.5 would reach 78.0) and fails its 10-rung honesty bar: the routing
headline's accuracy marker is excluded from every CI pytest invocation and carries no tests
(ci.yml:1150,1305,1594; conftest-only). Architecture (god-loop) and verification (ERRATA
ceiling) cannot move either. Precedent applied: boundary score authoritative — boundary
agrees with main, so the band is final at 77.5 / B.

Calibration: provenance original (rule (a) N/A), no archived cap, no demotion/promotion
taken, no calibration-notes entry required.
