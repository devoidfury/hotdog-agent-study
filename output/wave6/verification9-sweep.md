# verification-9 consistency sweep (wave6, 2026-10-01)

Read-only audit. Population built by grep of every `output/scores/*.json`,
`output/anchors/scores-*.json`, `output/boundary/scores-*.json` (not from memory).
Rule applied = the mechanical three-part rule from boundary/kilocode-verification9-reaudit.md:
(1) REAL model behavior (live provider calls / behavioral pass thresholds; scripted replay,
canned traces, deterministic smoke, human-click-throughs fail or carry only weakly);
(2) BLOCKING in CI (needs-edge or required-path, verdict hard-fails the job, zero
continue-on-error on the eval); (3) thresholds auditable in the snapshot
(private-harness thresholds = retained flag, not automatic demotion).

## Headline

**No band changes.** Both recommended demotions land 1.5 weighted and stay in band A.
Band-flipping claim of the kilocode re-audit holds. One structural note: the codex
ANCHOR is verification **8**, not 9 -- the 8-ceiling was never broken by an anchor;
the user-expected members codex/pi/grok-build/oh-my-pi/hermes-agent are all 8.

## Population (all files, verification == 9)

| subject (row) | eval mechanism (one clause) | blocking? | thresholds auditable? | verdict | new weighted total | band impact |
|---|---|---|---|---|---|---|
| kilocode (boundary) | live claude-sonnet-4.6 Harbor evals on draft-release binaries | yes, needs-edge publish.yml:606 | flag: private kilo-bench | **final-9 (re-audited; not re-litigated)** | 78.5 | none |
| prime-agent (main) | 28-task base-vs-head SWE comparison, label-gated | yes, verdict hard-fail behavioral-evals.yml:428-430 | yes | **9-stands** (re-verified in kilocode re-audit) | 72.0 | none |
| ouroboros (main) | nightly live-E2E, $30 cost cap + paid-reviewer smoke | yes: unattended, fails the nightly | yes (ci.yml cited) | **9-stands** (ledger #3) | 75.5 | none |
| smelt (main) | 17 oracled fuzz targets, committed seeds replayed ci.yml:269-294 | yes (blocking replay) | yes | **9-stands** (fuzz-exception class) | 74.5 | none |
| agentty (boundary) | oracled sanitizer fuzz in blocking build-test gate | yes | yes | **9-stands** (fuzz-exception class, boundary CLEAN per smelt bar) | 77.5 | none |
| openhands (main) | live-LLM Playwright E2E on non-fork PRs + scripted-provider E2E on main | yes for live half: ci.yml:490-492 verdict | yes (in-tree assertions) | **9-stands, flag retained, weakest rung** (alt 8 => 70.5, B) | 72.0 | none |
| **gemini-cli (main)** | live-model regression evals, but maintainer-approved run + report-only | **NO** | yes | **DEMOTE to 8** | **83.0** (84.5-1.5) | none (A floor 78; now tied open-interpreter 83.0) |
| **deepseek-reasonix (main)** | live-provider e2e-bot, comment-triggered only; fuzz targets never executed | **NO** | yes | **DEMOTE to 8** | **84.5** (86.0-1.5) | none (A floor 78) |
| grinta (main row STALE) | trigger pipeline, no consuming needs-edge | no | - | already fell: boundary **final 8 / 74.0 B** (reconciliation, not a new demotion) | 74.0 (final) | already applied |

Not in the 9-population (confirmed from files): codex anchor 8 (88.5 S), cline 8, pi 8
(anchor and boundary), grok-build 8, oh-my-pi 8, hermes-agent 8, open-interpreter 8
(83.0 A both rows), orca-agent 8 (78.75 A both rows), workground2 8, gptme 8, deepagents 8.

### 8.5 watch-list (not audited, per mandate)

- letta-code 8.5 / 75.25 B (main row, no boundary pair).
- zeroclaw: main row 8.5 / 78.25 A is **stale** -- boundary 8 / 77.5 B is final (scripted
  replay against a stubbed provider != live model behavior; demotion already adjudicated).
  Synthesis must take 77.5 B.
- Reconciliation note (same class, verification 8 not 8.5): grinta main row above.

## Evidence paragraphs (audited first-hand, snapshots under /data/samples/agents)

### gemini-cli -- DEMOTE to 8

Limb 1 passes: eval-pr.yml:87-92 runs `node scripts/run_eval_regression.js` against live
Gemini models with GEMINI_API_KEY, and the harness anti-grader design is real (harness from
main, agent from pinned PR head). Limb 2 fails mechanically: run_eval_regression.js:93-101
is literally annotated "Log status for CI visibility, but don't exit with error" and calls
`process.exit(0)` even when `hasRegression = true` (:72-75 detects the Action-Required marker
from compare_evals.js). So even after a maintainer clicks through the `eval-gate` environment
approval (eval-pr.yml:92), a CONFIRMED regression cannot turn the job red; the sole PR effect
is a comment (eval-pr.yml:167-211). The nightly is telemetry under the gptme rule verbatim:
`continue-on-error: true` on the eval step (evals-nightly.yml:68) feeding an aggregate-only
job (:102-120). Additionally USUALLY_PASSES/USUALLY_FAILS evals skip unless RUN_EVALS
(evals/test-helper.ts:380-387), so even the test-suite surface excludes the flaky-behavior
cases. Corpus precedent is on-point x3: gptme eval-ci.yml:28 continue-on-error = telemetry;
deepagents evals.yml:36 dispatch-only = NOT blocking; workground2 "manual-trigger/report-only"
= 8. gemini-cli is strictly weaker than all of them -- its gate cannot fail.
9->8: 84.5 -> 83.0, band A held (floor 78; ties open-interpreter at the A bottom).

### deepseek-reasonix -- DEMOTE to 8

Limb 1: live provider yes (e2e-bot.yml builds the agent from PR head, runs e2ebench against
the real DEEPSEEK provider with a graded suite and anti-grader checkout from main-v2,
e2e-bot.yml:70-105). Limb 2: fails -- the only trigger is an `/e2e` comment from an
OWNER/MEMBER/COLLABORATOR plus an `environment: e2e-bot` approval (e2e-bot.yml:9-32); no
push/PR/release path consumes it, no needs-edge exists. To the record, the run is NOT pure
telemetry: a report.json verifier exits 1 on any unsuccessful task (e2e-bot.yml ~:168-175),
so IF run it fails -- but a gate that only runs when a human asks is the deepagents
dispatch-only shape, which the corpus already ruled "NOT blocking, 8 correct"
(calibration-notes.md:212). No CI-executed fuzz either: grep for `fuzz` across all 20
reasonix workflows = zero hits, so the score file's own phrase "fuzzing is nominal" means
targets that never execute in CI -- below the smelt bar (which requires seed REPLAY in
ci.yml:269-294). The score file's own note concedes the fall condition: "the eval reports but
never gates" -- written under the pre-kilocode rule, contradicted by the confirmed one.
9->8: 86.0 -> 84.5, band A held. Stays the top A after demotion; does not enter S range
(codex 88.5 S) nor approach the floor.

### openhands -- 9 STANDS (weakest rung, flag retained)

Limb 1: mixed as originally flagged -- mock-llm-e2e.yml is scripted-provider (fails limb 1)
and fires only on push to main (mock-llm-e2e.yml:3-5), i.e. post-merge; but the live half is
independent and real: ci.yml runs `npm run test:e2e:live` with LIVE_E2E_LLM_API_KEY
(ci.yml:265-276). Limb 2: the Playwright step swallows its exit (`set +e` + capture, :266-276)
but the job ends with a hard verdict step, "Fail live E2E job when tests fail" ci.yml:490-492
(`exit $exit_code`) -- the exact prime-agent tolerance-plus-verdict pattern the corpus kept at
9 (behavioral-evals.yml:428-430), and there is no `continue-on-error` on the live-eval job.
Two honest caveats keep the flag: the whole lane skips on fork PRs, artifact-only commits, or
missing key (exit_code forced 0, ci.yml:432-455) -- fork coverage is zero; and whether the
job is a *required* check is branch-protection state not visible in the snapshot
(same unverifiable-tail trust class as kilocode's private thresholds, flagged not silent).
Limb 3: pass semantics = in-tree Playwright assertions (tests/e2e/live), auditable.
Verdict: 9-stands, ranked bottom of the ledger per the kilocode re-audit ordering; if
synthesis ever reads limb 2 strictly-required-only, the honest fallback is 8 => 70.5, band B
unchanged -- the score file itself books this alternative.

### Non-demoted without first-hand re-audit (rationale + prior adjudication)

- **ouroboros**: nightly-but-unattended, blocks on failure, cost-capped (ci.yml:9,631,443 per
  score record). The kilocode re-audit explicitly retired "must be PR-gated" as a withhold
  criterion (it contradicts prime-agent's label-gate and this row); ledger rank 3 stands.
  Fallback 8 = 74.0, band B stable either way -- no swing risk justifies re-opening.
- **smelt / agentty**: fuzz-exception class. The anchors' own erratum is "no in-CI model
  evals, no fuzzing" -- either qualifies for the ceiling break; smelt's seeds REPLAY in the
  blocking CI (ci.yml:269-294) and agentty's oracled fuzzing runs in the blocking build-test
  gate (boundary verdict, ci.yml:476/515-516). They do not satisfy limb 1 (no model runs) and
  must never be cited as evidence for model-eval claims; they are the corpus's second,
  parallel road to 9. No demotion: the rule's limb-1 wording governs EVAL claims; fuzz claims
  keep their own audited standard (oracled + executed in blocking CI, which both clear).
- **kilocode / prime-agent**: final per the dedicated re-audit; not re-litigated here.
- **grinta**: no re-audit needed -- the boundary already applied the rule's limb 2 (missing
  consuming needs-edge) and finalized 8 / 74.0 B. Action item is bookkeeping only: synthesis
  must not let the stale main row (9 / 78.5 A) into the tier list.

## What verification-9 now means in this corpus

> **verification 9** = the subject's snapshot contains a model-behavioral verification gate
> that (1) drives LIVE provider calls and grades behavior (or, on the parallel fuzz road,
> executes oracled fuzzing of the shipped code paths in CI), (2) runs AUTOMATICALLY on a
> CI-required or release path and CANNOT be green when the behavior regresses -- a
> tolerant-steps-plus-hard-verdict job counts (prime-agent, openhands), but
> continue-on-error, report-only scripts that exit 0 on regression (gemini-cli), and
> comment-/dispatch-only manual triggers (deepagents, gptme, workground2, reasonix) are
> telemetry, not verification; and (3) has pass-threshold semantics auditable from the
> snapshot, with out-of-tree thresholds (kilocode's kilo-bench, openhands' required-check
> status) retained as explicit flags rather than silent passes or automatic demotions.
> After this sweep the ledger ranks: smelt > agentty (fuzz class) | prime-agent > ouroboros >
> kilocode > openhands (eval class, openhands bottom with flag); gemini-cli and
> deepseek-reasonix exit the ledger; grinta's fall is reconciled. 10 remains gated on fuzzing
> (of model-bearing code paths) plus eval breadth -- none of the 9s has both.
