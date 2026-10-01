# Calibration notes

Every demotion/promotion against the mechanical census tiers, with reason. Never silent.

## Tier adjustments after census (2026-09-29, before Phase 1)

| subject | census tier | study tier | reason |
|---|---|---|---|
| codex-infinity | T3 | T0 (differential triage) | sync-fork:codex; full giant review on a clone is waste; promote back only if triage finds real divergence |
| open-interpreter | T3 | T0 (differential triage) | rename-clone:codex (manifest codex-monorepo); same rule |
| mimo-code | T3 | T0 (differential triage) | rename-clone:opencode; same rule |
| free-code | T2 | T0 | leaked-snapshot:claude-code, no license; study rule: leak subjects get license-risk findings + no code-adjacent portables, not deep review |
| claw-code-agent | T1 | T0 | leaked-snapshot:claude-code, no license; same rule |
| auto-code-rover | T3 | T2 | census language "diff", 2.6M LOC likely an artifact of checked-in diffs/vendored blobs; LOC unreliable, rescope at review; SONAR-Source-Available license noted |

Census caveats reviewers should weigh: shallow clones (shallow=y) hide history -- activity
judgments are evidence-limited for those subjects; contributor counts of 1 on shallow clones
are measurement artifacts, not bus factor. gemini-cli test LOC 4.8k and qwen-code 3.77M LOC
look like measurement artifacts (vendored/generated code); reviewers must sanity-check scope
per subject rather than trusting census LOC blindly.

## Pending decisions
- Boundary re-reviews (within 2 pts of a band) recorded here as they come in Phase 3.

## Phase 1 anchor findings affecting census (2026-09-29)

| subject | issue | correction |
|---|---|---|
| codel | suggested_tier T1, shallow clone hid inactivity | T0: remote `pushed_at 2024-04-29` (dead >12mo, rule (b) noted); module name `github.com/semanser/ai-coder` (rename, benign) |
| crush | test_loc 0 | wrong: 278 `*_test.go` = 61,738 LOC; also lineage note: shares opencode ancestry (`internal/agent/opencode_routing_test.go`) - divergent fork, no-outrank rule not triggered yet, revisit vs opencode's review |
| cline | test_loc 39k | wrong: ~255,600 LOC in 792 `*.test.ts` |
| nanocoder | test_loc 1,057 | wrong: colocated `*.spec.*` ~161k LOC counted as non-test |
| codex | census LOC sane | `*_tests.rs` (439,900 LOC) correctly classified as test |

Band placement vs dispatch expectations: cline expected A, scored 76.5 (B, band ceiling) - no OS sandbox, no cache discipline, evals not in CI; documented here, not silent. All others matched expected bands (codex S 88.5, pi A 78.5, crush B 69.0, nanocoder C 60.5, codel D 23.0). Distribution check: 1/6 scored 80+ - calibration intact.

## Phase 2 differential triage - clones/leaks (2026-09-29)

| subject | census flag | triage verdict | score / band | note |
|---|---|---|---|---|
| codex-infinity | sync-fork:codex | sync-fork CONFIRMED (daily-upstream-sync.sh merges openai/codex; LICENSE/NOTICE identical) + auto-next/goals/arc-monitor patch layer | 83.5 / A | rule (a) respected (83.5 < 88.5); durability docked to 2 (solo maintainer, /home/lee cron); not re-promoted to T3 - delta is a patch series, not an architecture |
| open-interpreter | rename-clone:codex | rename-clone upheld at identity layer (manifest "codex-monorepo"), but evidence says honest documented distribution fork (FORK_BRANDING.md, README.md:47, LICENSE/NOTICE byte-identical) | 78.5 -> **D policy cap** | POLICY DEMOTION per dispatcher rename-clone rule, recorded here not silent; honest weighted would be A; license review found NO Apache-2.0 violation - findings carry residual provenance/trademark risk + census-detector anti-pattern; recommend re-review as divergent-fork if census semantics are revised |
| mimo-code | rename-clone:opencode | CORRECTED to divergent-fork:opencode - README.md:553 declares the fork, LICENSE retains opencode copyright, structural delta (memory-FTS, actor/inbox, workflow runtime, 40+ own migrations) | 60.0 / C (provisional) | rule (a) not checkable until opencode is scored; crush (opencode-lineage) at 69.0 suggests upward move on full review; USE_RESTRICTIONS.md addendum over MIT logged as license-risk nuance |
| free-code | leaked-snapshot:claude-code | LEAK CONFIRMED (self-declared "claude-code-source-snapshot" v2.1.87 + internal Slack channel ID C07VBSHV7EV at prompts.ts:245; no LICENSE; 199 test LOC) | 38.5 / D | protocol rule 3 applied: license-risk findings only, NO code-adjacent portables; guardrail-stripping (README.md:66-72) logged as anti-pattern |
| claw-code-agent | leaked-snapshot:claude-code | CORRECTED to port-from-leaked-source: own Python architecture + 1,233 tests, but in-code "Ported from npm ..." declarations (bash_security.py:3, compact.py:3) + verbatim proprietary prompts (prompt_constants.py:46) | 44.0 / D | same rule-3 treatment (no code-adjacent portables); shallow-clone honored (HEAD = "Merge pull request #43"); borderline: verification 5->6 on boundary re-review would put it at 45.5 = C |

Census-semantics finding for the dispatcher: the rename-clone detector keys on manifest-name equality and therefore cannot distinguish deceptive rebrands (free-code posture) from honest attribution-preserving forks (open-interpreter, mimo-code). Recommend requiring a corroborating signal (attribution stripped / fork denial) before the rename-clone label, and a separate `port-from-leaked-source` flag (claw-code-agent).
Convergence noted: `auto-continuation-goals` (codex-infinity verified code, mimo-code same claim), `unrenamed-manifest-identity` (open-interpreter + mimo-code), `regex-denylist` now x3 with nanocoder.
Distribution check after this batch: 0/5 new subjects at 80+ (codex-infinity 83.5 A is carried-codex under rule (a), flagged separately); no calibration loss.

## Phase 3 T3 giant review - hermes-agent (2026-09-29, integrator)

| item | decision | reason |
|---|---|---|
| tier | T3 upheld | original provenance (scout: origin = census upstream, full history, no fork markers), hyperactive (17,797 commits/30d, head 2026-09-26) - rules (a) and (b) inapplicable, no demotion |
| score | 83.0 / A | first non-anchor subject at 80+; distribution check 3/12 = 25% (at, not over, sanity line) |
| scout-vs-lane override | safety-enforcement 7 accepted over scout's predicted 6-rung | below-yolo floors + fail-closed waits + default-deny authz are code-verified and tested (`approval.py:1052-1057`, `authz_mixin.py:464`); zero shipped kernel enforcement holds it under 8. Recorded, not silent |
| cross-dimension evidence | live-provider cache-hit canaries credited to token-economy only | verification stays at the anchors' 8-ceiling (no fuzzers, no safety evals); the same evidence cannot break two ceilings |
| docs-dx 9 | accepted despite codex 7 / cline 8 | 741 md/mdx + failure-copy engineering + install matrix; codex's 9 was docked by 206-line stub docs, not hermes's problem |
| concept dedup | c4+s12 -> `crash-recovery-posture`; c6+e12 -> `expired-compat-shim-layer`; e4 -> `compaction-tiering`, e7 -> `scheduled-agent-runs` (aliases recorded) | protocol concept discipline; convergence feed: regex-denylist x4, faux-provider-testing x4, permission-policy x5, sandbox-delegation x7 |

## Phase 3 T3 integration - qwen-code (2026-09-29, run 20260929-0502-t3-giant-review-2)

qwen-code scored 80.0 / A (closest anchor codex). No demotion/promotion applied: rule (a) checked and NOT triggered - provenance is divergent-fork:gemini-cli v0.8.2 (README.md:214 "stopped syncing"), and gemini-cli is unscored; archived/dead cap not triggered (remote `archived:false`, `pushed_at 2026-09-29`). Census corrections carried: LOC split inverted (~1.58M prod / ~2.30M test vs claimed 3.77M/213k) and `contributors:1` remote-verified as shallow-clone artifact (133 releases, PR #12532 at head). Distribution check: 4/17 subjects now at 80+ (23.5%) - still under the 25% line; the next 80+ trips it, dispatcher should re-anchor before scoring another giant.

## Boundary + policy decisions (dispatcher, 2026-09-29)

| item | decision | reason |
|---|---|---|
| cline boundary | B FINAL | independent re-review derived 76.5, exact match with anchor score; "tested approval, no OS sandbox" (zero seatbelt/bwrap/landlock/seccomp hits; `"*": {autoApprove: true}` cron default cron-runner.ts:88-93) upheld; A floor not crossed |
| open-interpreter band | D policy cap REVERSED -> 78.5 A, rule (a) ceiling vs codex 88.5 | bands measure code quality; fork-honesty/trademark risk lives in findings (already logged); triage's own analysis said honest weighted is A and license review found no Apache violation; both the demotion and this reversal recorded here, never silent |
| claw-code-agent | 44.0 within 1 pt of C floor -> boundary re-review dispatched (boundary-claw-code-agent) | band rule: within 2 pts of a band gets a fresh independent reviewer before the band is final |
| mimo-code | 60.0 stays provisional | rule (a) check requires opencode's score first; adjust at synthesis if opencode lands low |
| distribution watch | 4/17 (23.5%) at 80+ - NO re-anchor yet | anchors scored strictly; lane dockings observed (hermes safety 7, qwen verification 8-ceiling). IF count crosses 25%: re-score the two subjects nearest 80 with fresh reviewers before scoring another subject |

Census-detector improvement adopted from triage (and itself filed as a study finding): rename-clone
labels require a corroborating attribution-stripped/fork-denied signal; add distinct
`port-from-leaked-source` classification (claw-code-agent pattern).

## Boundary results round 2 (dispatcher, 2026-09-29)

| item | decision | reason |
|---|---|---|
| pi boundary | A FINAL (78.5, independent recount matched Phase 1 exactly) | but flagged as the study's most fragile rung: architecture 9 vs 8 (interactive-mode.ts 6,852 LOC at modes/interactive/interactive-mode.ts:415 + agent-session.ts 4,023 LOC) decides A vs B; both independent scorers kept 9, so 9 stands; errata appended to anchor-excerpts.md, never silent |
| pi side-finding | evals-in-CI gap recurs | real eval harness (packages/evals/package.json:9) wired to no CI workflow - second instance of the `evals-in-ci` gap pattern (after cline); verification-8-ceiling now applies to 3 subjects (codex, cline, pi) |

## opencode result + lineage checks (dispatcher, 2026-09-29)

| item | decision | reason |
|---|---|---|
| opencode 77.0 B | boundary re-review dispatched (boundary-opencode) | 1.0 pt below A floor; integrator itself documented "band sensitivity: top of B, crossing declined" - exactly the case the boundary rule exists for |
| rule (a) mimo-code | CHECKED, no action | divergent-fork 60.0 < opencode 77.0; stays provisional C, may rise on its own merits at synthesis reconciliation |
| rule (a) crush | CHECKED, no action | opencode-ancestry noted by anchor phase; crush 69.0 < opencode 77.0; ancestral relation is divergent - no cap needed, lineage kept as nuance |
| rule (a) kilocode | pending | in-flight T3; if its weighted exceeds 77.0 the integrator must show the divergent-fork delta explicitly before A-range standing (fork may outrank upstream ONLY on demonstrated divergence) |
| schema note | lane_scores object-form | opencode.json uses {score,weight,weighted,lane} objects vs protocol plain numbers; synthesis merge normalizes to {dimension: number} and keeps lane attribution in run-dir copy; no rescore needed |

## openhands T2 (2026-09-29)

- Identity: flagship OpenHands repo has pivoted to Agent Canvas (TS shell, core out-of-tree in
  software-agent-sdk, pinned 1.49.6). Scored as a shell: loop/compaction/action-policy earned
  zero credit, core's flaws zero blame. Do not compare its 72.0/B directly against full-harness
  subjects without that covariate. If a software-agent-sdk sample enters the corpus, review it
  separately; openhands+core composite would land ~A, not carried here.
- Census corrections: test_loc 149,921 → ~195,310 (missed ~45k: `__tests__/` partially or fully
  + colocated `src/**/*.test.*`); non_test_loc 205,856 → ~160k, incl. 42,502-LOC generated
  src/i18n/translation.json (recommend generated-blob exclusion list). contributors/commits=1 =
  shallow artifacts; remote-verified active (pushed 2026-09-29, MIT, 89,472 stars, renamed org
  All-Hands-AI→OpenHands, redirect benign, provenance stays `original`).
- verification 9 is the corpus's first break of the 8-ceiling: workflow file:line evidence of
  model-in-CI in both faux (mock-llm-e2e.yml:90-134) and live (ci.yml:265-276, fork-PR-gated)
  form. They are acceptance smokes with token assertions, not scored evals; alternative reading
  = 8 → total 70.5, band B unchanged. Flagged for synthesis.

## claurst (T2, 2026-09-29)
- Provenance correction: census `divergent-fork:claude-code` is a mislabel - no code lineage; it is a
  behavioral reimplementation from a spec generated off the leaked CC npm sourcemap (README.md:208-218,
  134 TS-mirroring comments). Recommend lineage family `ported-proprietary-source` (claw-code-agent
  precedent), NOT fork-family. Rule (a) uncheckable locally (claude-code not in corpus): evidence-limited.
- No demotion applied; total 65.0 sits exactly on the B/C boundary. safety-enforcement=4 is the swing
  (phantom /sandbox-toggle + project-settings hook/rule self-grant); a reviewer weighting "present but
  misleading" (rung-2) harder would land sub-65 -> C. Flagged for synthesis.
- Census hygiene: commits/contributors=1 are shallow-clone artifacts; remote-verified active
  (pushed_at 2026-09-02, created 2026-03-31, not archived, 10,310 stars, external PR #403 at HEAD).

## gptme (T2) + boundary queue update (dispatcher, 2026-09-29)

- gptme 76.0 B: self-flagged 2.0 under A floor -> boundary-gptme dispatched. Swing points:
  token-economy (3-tier compaction + byte-range master-context recovery), architecture god
  files (api_v2.py 4,055 / shell.py 3,701), verification 8-vs-9 hinges on eval-ci.yml:28
  continue-on-error. RULE REFINEMENT going forward (after boundary verdict confirms): evals in
  CI count toward verification 9 only if the job BLOCKS on failure; continue-on-error evals =
  existence of harness, not enforcement.
- gptme safety-hole noted: no-confirm-hook -> auto-confirm fail-open (confirm.py:228-236) with
  sandbox/guardrails default off/shadow. Pairs with claurst's phantom /sandbox-toggle: new
  candidate concept family around "advertised-but-inert safety controls" - synthesis to check
  whether claurst-safety + gptme-failopen + nanocoder-fail-open merge or stay distinct rungs.
- claurst 65.0 exactly on B/C line, safety 4 named as swing -> boundary-claurst dispatched.
- openhands 72.0 B with partial-product covariate (agent-canvas only; SDK core absent - do NOT
  compare directly against full harnesses; first verification-9 break of the 8-ceiling is
  flagged as acceptance-smoke-with-token-assertions, alt reading 8 => 70.5 B, band unchanged).

## prime-agent (T2, 2026-09-29)
- 72.0 B, divergent-fork:pi scored on own merit; rule (a) moot (below upstream 78.5).
  Divergence census-understated: 713 prime-only + 435 modified files vs 91 hash-identical.
- verification-9 ledger (audit at synthesis): (1) openhands - faux + live acceptance smokes w/
  token assertions, band-safe either reading; (2) prime-agent - label-gated FAIL-CLOSED 28-task
  SWE release gate (behavioral-evals.yml:430). #2 fits the blocks-on-failure rule cleanly;
  #1 stays flagged. Any later verification-9 must cite the workflow line that blocks.
- prime-agent-8: fork predates pi's project-trust gate AND gutted pi's SECURITY.md threat-model
  text - honest-reporting delta, feeds "advertised-but-inert / removed-safety-docs" family watch.

## kimi-code (T2, 2026-09-29)
- 76.5 B self-flagged 1.5 under A floor -> boundary-kimicode dispatched (swing: architecture
  7-vs-8 on v1/v2 dual-engine migration; originality items to survive code verification).
- Provenance nuance logged: VENDORS pi-tui from an earendil-works/pi commit (UPSTREAM.md pin,
  MIT) - library dependency, not product lineage; rule (a) N/A. Boundary reviewer tasked to
  attribute god files (kimi-own vs vendored) before any architecture docking.

## SWE-agent (T1, 2026-09-29)
- 59.0 C. Historical ACI ancestry logged as influence evidence ONLY, not ranking (per dispatch
  instruction) - keeps originality honest at 7. Safety 5 evidence-limited: sandbox/container
  machinery lives in external swerex package (same out-of-tree-covariate family as openhands;
  synthesis covariate list now: openhands, cursor-agent, SWE-agent-partial).

## trae-agent (T1, 2026-09-29)
- 44.0 D self-flagged 1 pt above D/C cut -> boundary-trae-agent dispatched (hinge: verification
  5-vs-6; core loop allegedly untested, Makefile:42 skips 3 provider suites in CI).
- Safety-hole candidate: --docker sandbox covers only 3 tools; MCP + ckg execute on HOST
  (base_agent.py:68). Added to advertised-but-inert family watch as variant "partial-coverage
  sandbox" (family now: phantom-toggle claurst / fail-open gptme+nanocoder / removed-docs
  prime-agent / delegated-never-confirm openhands / partial-coverage trae-agent).

## cursor-agent (T1, 2026-09-29) - covariate resolution
- 33.5 D, mid-band, no boundary. Wrapper question RESOLVED NEGATIVE: standalone MIT harness,
  no Cursor binary invocation; name collision with cursor.com's cursor-agent CLI is branding
  only (finding cursor-agent-1, identity hazard). REMOVED from out-of-tree covariate list
  (openhands stays; SWE-agent partial stays). Scored tree IS the agent: single-API-call core,
  no loop plane - D is on merit, not packaging.

## claw-code-agent boundary RESOLVED (dispatcher, 2026-09-29)
- FINAL: 50.0 C (boundary) supersedes 44.0 D (T0 triage). Rationale: triage was a 10-minute
  half-page pass by protocol ("scored only so the tier list is complete"); the independent
  boundary read is deeper and dimension-level, finding verification 5 stands via genuine
  property tests (test_agent_runtime.py:623/:704/:2842) while other dimensions were under-
  credited at triage. Mean (47.0) rejected: lands within 2 of the SAME cut - boundary rule
  does not recurse; one fresh reviewer decides after the first flip.
- Policy note: T0 scores are treated as band-accurate ONLY when >3 pts from a cut; otherwise
  the boundary reviewer's score is authoritative by default (this precedent applies silently
  to no one - each use logged). Other T0s are currently clear of cuts (max 38.5 vs 45).
- License-risk hardened: 32 "Ported/Mirrors from npm" declarations naming UNRELEASED Claude
  Code TS internals + whole-repo parity checklist + "open-source" badge with no LICENSE and
  GitHub license:null. NO code-adjacent portables from this subject or its lineage.
- dead-or-alive resolved: created 2026-04-01, pushed 2026-06-22, not archived, solo - rule (b)
  not triggered; C stands on activity too.

## verification-9 ledger update (smelt, 2026-09-29)
- (3) smelt: 17 oracled fuzz targets (incl. TUI-loop + cache-prefix invariance) with regression
  seeds replayed in CI (ci.yml:269-294) - CLEAN qualifying under the anchor rule ("what separates
  8 from 10" = evals/fuzzing); strongest of the three 9s so far. Ledger: openhands (flagged
  acceptance-smoke), prime-agent (blocking release gate), smelt (fuzz replay).

## deepagents (T2, 2026-09-29)
- 76.0 B, exactly 2.0 under A floor -> boundary-deepagents dispatched. Extra mandate: its ruling
  on LOOP-IN-LIBRARY subjects (core = langchain create_agent) becomes the convention for the
  covariate family (openhands out-of-process, SWE-agent/swerex, kimi vendored lib) - boundary
  reviewer must state a generalizable ruling, not just a score.
- Census: fixture-inflation now its own concept (test-fixture-loc-inflation, deepagents coined;
  tau2 db.json 205k inside tests dirs). Related ledger: verification-9 attempts must show a
  blocking line; deepagents evals.yml:36 is workflow_dispatch-only = NOT blocking, 8 correct.

## kode-cli lineage (2026-09-29) - claude-code-descendant family grows
- kode-cli 70.5 B; census/lore "cline-lineage" REFUTED (zero cline markers). Actual descent:
  Claude Code internals via anon-kode route - Anthropic-INTERNAL env branches
  (USER_TYPE==='ant', SWE_BENCH at query-executor.ts:22, retry.ts:5) can only come from an
  extracted/internal build. Apache-2.0 self-grant filed as license-risk; portables concept-only.
- Family ledger for synthesis (claude-code-descended subjects): free-code (raw snapshot v2.1.87),
  claw-code-agent (port-from-leak, unreleased internals named), claurst (spec-from-sourcemap,
  clean-room claim refuted), kode-cli (internal-branch markers = earliest/most internal source
  contact). Each distinct pathway; NONE code-portable. If one more appears, this is the study's
  headline finding by instance count.

## boundary-opencode RESOLVED (2026-09-29)
- FINAL: 77.0 B (exact match with provisional; third exact-match boundary of the study).
  Rationale recorded: V1 shipping path scored; arch/verification/originality - each lane
  contested individually - all re-score 8, none survives 9; token-economy 7->8 explicitly
  declined (no projected-context measure vs pi rung). opencode = band ceiling, not near-miss.

## auto-code-rover (2026-09-29, T2) - census blob-inflation + dead at review
- Census said T3 / 2.6M LOC / language "diff": all wrong - 927MB of checked-in results/ run blobs
  (3,161 .diff). Real: ~13.9k Python LOC (app 6,968 / test 6,141 / scripts 814). Filed as
  test-fixture-loc-inflation second instance; census pipeline should exclude results|experiment dirs.
- Activity: shallow clone, head_date 17mo old; remote-verified pushed_at 2025-04-24, archived:false.
  Shallow clone hid NOTHING this time - rule (b) dead-cap B applies regardless of direction.
- Score 45.5 C, boundary-flagged (0.5 over the cut; sensitivity both ways in report). Weakest
  safety_enforcement 2: no approval layer AND README:209 recommends host execution of model-written
  reproducers - the eval path is the anti-pattern, the Docker path is the demo.
- License SONAR-SA v1.0 => concepts-only portables (reviewer-retry-loop nuance, aci-tool-design
  nuance, sbfl-context-seeding unique, retry-blind-backoff anti-pattern new id).
- Provenance: original, no corpus fork relation; rule (a) n/a.

## darce-cli (2026-09-29, T1) - census corrections, no calibration rules invoked
- Census test_loc 0 -> actually 868: root-level `test.ts` (106 behavioral tests) missed by glob.
- Census non_test_loc 5763 inflated: src/ is 2,690 LOC; 5,763 requires counting package-lock.json
  (2,944). True size ~2.7k LOC => T0-range; T1 review run as dispatched, tier flag for synthesis.
- Shallow clone checked remotely: created 2026-03-31, pushed_at 2026-04-01, archived:false. Thin
  activity, but NO dead/archived claim made; rule (b) not applied.
- Score 32.5 D (closest anchor codel, above it on loop reality + real tests). Weakest
  safety_enforcement 1: zero permission/sandbox layer, Bash full-env spawn (BashTool.ts:27-29),
  no path containment; honest by omission (no phantom controls), which is why 1 not 2.
- License manifest-only MIT (package.json:38, no LICENSE, GitHub license:null) => concepts-only,
  third instance of manifest-only-license id.

## gemini-cli T3 integration (2026-09-29, recovered integrator - run 20260929-0502-t3-giant-review-3)
- FINAL: **84.5 A** (original integrator node timed out at 2700s; scout + reviewer-core +
  reviewer-safety + reviewer-econ all succeeded - lanes consumed as-is, totals are the
  integrator's). Closest anchor **codex**: only cohort subject with tested 3-platform kernel
  enforcement and codex-grade interop/subagents/CI; the two systematic gaps vs codex (88.5) are
  enforcement DEFAULTS and the absence of a first-class queue/fork plane. Even the maximal
  single fix - safety 8->10 - lands at 86.5, so S needs both codex gaps closed at once.
- Source lanes: architecture/operability=reviewer-core; safety-enforcement/verification=
  reviewer-safety; token-economy/orchestration/interop/docs-dx=reviewer-econ;
  **originality 9 + durability 9 = integrator**, filed with code evidence per the
  not-marketing rule (probe-verified snapshot pass chatCompressionService.ts:616-638, inflation
  veto client.ts:696-703, policy control-plane hardening policy/config.ts:238-257 +
  integrity.ts:31-56, eval-gate CI eval-pr.yml:87-92, docs-audit.yml:30-40). Durability 9 on
  Google institutional backing + 47 in-tree workflows + remote-verified activity: shallow clone
  handled per the codel lesson - `git ls-remote origin HEAD` reachable at 38700b4, AHEAD of
  snapshot head 2fe7c2d (2026-09-25). Bus factor unverifiable (1-commit clone), same caveat as
  qwen-code 9.
- STRONGEST: verification (9/15). Only subject in the study to legitimately break the ERRATA
  verification-8 ceiling - the named exception (in-CI model evals) satisfied with workflow
  file:line: eval-pr.yml:87-92 (human-approved `eval-gate` env, `pull_request_target` risk
  correctly handled) + evals-nightly.yml:3-5 - on the study's largest property-asserting corps
  (976 files / 387,651 LOC, incl. chained-gating spec shell-safety-regression.test.ts:35-56 and
  whole E2E suite run twice, sandbox off + GEMINI_SANDBOX=docker, package.json:56-63).
- WEAKEST: architecture (8/15), the largest weighted shortfall (-3.0) and reviewer-core's own
  call: 6 independent Scheduler-built entry-path drivers (c2) incl. an ACP path that
  re-implements policy evaluation (acpSession.ts:406,659 vs scheduler/policy.ts:80-94) plus the
  4249-LOC Config hub (config.ts:757). ERRATA >5k-LOC product-file check passes (largest
  text-buffer.ts 4285), so no cross-side docking applies at either 8 or 9.
- verification-9 LEDGER (now four): openhands (flagged acceptance-smoke), prime-agent (blocking
  release gate), smelt (deterministic fuzz replay - still strongest), **gemini-cli (evals-in-CI
  but softest blocker of the four**: eval-gate is a human click, not a build failure, and
  USUALLY_PASSES/USUALLY_FAILS cases skip unless RUN_EVALS, test-helper.ts:380-387). No fuzzing
  anywhere despite a hand-written shell parser being the most fuzzable component in the cohort
  (policy-engine.ts:466) - the 9->10 gate.
- Disagreements resolved: (1) reviewer-core called architecture closest to cline, reviewer-safety
  and reviewer-econ called codex - resolved to codex for the whole subject (cline retained as the
  architecture rung reference). (2) reviewer-core flagged the ACP duplicate-policy path as
  safety-relevant; reviewer-safety never scored it - ruled a divergence/maintenance hazard, not
  an enforcement hole (ACP path is fail-closed on missing engine, acpSession.ts:108-110), so
  docked in architecture only, safety 8 unchanged. (3) interop 9 kept despite unverified SDK
  publication (publish-release/action.yml:12-18 covers cli/core/a2a only) - demotion to 8 would
  give 83.5, band-irrelevant, recorded rather than silent. (4) rewind partial-abort defect
  (rewindFileOps.ts:184-192, comment/code mismatch) counted once, in operability's 8.
- Census correction honored: test_loc 4854 is the third instance of the `*.test.ts` miscount
  (after cline, nanocoder) - real 387,651 LOC. Scoring verification off the census row would have
  inverted the subject. non_test_loc inflated ~2.2x over tracked TS.
- Merge normalization: 36 findings (10 core + 14 safety + 12 econ) -> **34** after two concept
  dedupes - c9 token-ground-truth-feedback into e2 -> `usage-measured-compact-trigger`
  (evidence-stronger lane kept, both evidence sets unioned) and s12 crash-posture into c6 ->
  `crash-recovery-middleware` (nuance folded into one_why). 8 concepts re-canonicalized to
  seeded/established ids with `aliases` recorded: compaction-tiering, loop-detection,
  evals-in-ci, injection-screening, security-posture-docs, sandbox-delegation,
  prompt-cache-warming, tool-output-spillover-files. c2 entry-path-loop-duplication deliberately
  kept NEW rather than merged into per-provider-loop-duplication - different duplication axis
  (entry surface vs provider), cross-noted; convergence counts should treat them as siblings.
  Kinds normalized to schema (lane "strength"/"capability" -> portable/unique per cohort-uniqueness
  claims; safety's `gap` not present this run), originals preserved as `lane_kind`.
- Rules: provenance original per scout (upstream, and the cohort's only *parent* - qwen-code is
  its divergent fork). Rule (a) n/a for gemini-cli itself; lineage ordering checked downstream:
  qwen-code 80.0 < 84.5, consistent with a divergent fork that also drifted down on architecture
  6. Rule (b) n/a - remote-verified active. **No demotions applied.**
- Distribution: 5/82 scored subjects at 80+ = 6.1% (codex 88.5, codex-infinity 83.5, hermes-agent
  83.0, qwen-code 80.0, gemini-cli 84.5) - well under the 25% line, calibration intact.

## Phase 3 T3 giant review - grok-build (2026-09-30, integrator)

| item | decision | reason |
|---|---|---|
| tier | T3 upheld | original provenance (official xAI/SpaceXAI monorepo export, SOURCE_REV pinned, no fork/rename/leak markers, undeclared-identifier grep = 0); head 2026-09-23 alive; rules (a)/(b) inapplicable, no demotion; 82.75 < codex 88.5 so no-outrank moot even if lineage were read loosely |
| score | 82.75 / A | distribution check 7/87 = 8% at 80+, well under the 25% line |
| integrator-owned dims | originality 9, durability 7 | originality: all 4 lane-unique claims re-verified in source (two-pass prefire compaction.rs:59-66; parent-prefixed side calls side_call.rs:102-131; confine-or-refuse config/mod.rs:1442-1563; leader daemon leader/mod.rs:1-40); docked from 10 by codex-lineage shape + corpus prior art on 2 of 4 uniques. durability: xAI backing + Apache-2.0, but zero visible maintenance surface in mirror (vs codex-9's 30+ workflows) and closed governance (no external PRs); shallow clone honored per census caveat |
| disagreement handling | no numeric conflicts (disjoint lanes); 4 tensions resolved in prose in subjects/grok-build.md | leader-daemon credit taken once (operability) not twice (no second dock on safety); manager/mod.rs size ruled neutral evidence (mostly tests from :1510); verification 8 unanimous at ceiling, method = in-repo harness evidence (CI unknown-due-to-export) |
| concept dedup | 43 -> 42 records; c11+s10 -> faux-provider-testing; renames e1->compaction-tiering, e6->fork-cache-alignment (waveloom prior art), e8->workflow-resume-journal, e11->scheduled-agent-runs; aliases recorded | protocol concept discipline |
| convergence feed | faux-provider-testing x34, sandbox-delegation x22, compaction-tiering x29, workflow-resume-journal x13, test-suite-without-ci x11, grammar-parsed-bash-policy x9, confine-or-refuse-startup x3, fork-cache-alignment x2, scheduled-agent-runs x5 | candidate canonicals coined: durability-loss-contract, cursor-addressable-replay, leader-follower-ipc-daemon, two-pass-background-prefire, session-actor-command-loop, monolithic-turn-function (function-scale sibling of god-file-loop, not merged) |
| flagged | leader/hub multi-process trust boundary un-audited; sandbox disables leader mode rather than confining it (pager-bin/main.rs:1358-1372) | deserves its own pass |

## grinta-coding-agent (T2, 2026-09-30)
- 78.5 A EXACTLY on the floor -> boundary-grinta dispatched. Swing: verification 9 rests on
  every-PR mutmut (mutation-testing.yml:38) + LABEL-GATED DeepSWE eval (run-eval.yml:4-10).
  Boundary mandate: is the mutation gate blocking (surviving mutant fails PR?) and what
  fraction of the tree does mutmut cover; label-gate = release property or decoration.
  Mutation testing would be the ledger's first entry of that kind - strongest verification
  evidence class the corpus has produced if it holds.
- New finding shapes: compaction-continuity-gate (continuity_eval.py:20-36) - compaction
  validated for narrative continuity, not just token math; rival-subscription-transport
  (codex_app_server.py:29) - third-party harness riding another vendor's SUBSCRIPTION auth:
  ToS-risk taxonomy question (license-risk vs safety-hole) assigned to boundary ruling.
- OpenHands-lineage vocabulary w/o code residue: lineage evidence gradation noted for synthesis.

## verification-9 ledger update (ouroboros, 2026-09-30)
- (5) ouroboros: nightly live-E2E with $30 cost cap (ci.yml:9,631) + paid-reviewer smoke
  (ci.yml:443). Qualifies: runs unattended, blocks on failure, and COST-BOUNDED - first
  budget-guarded eval gate seen; cost visibility inside verification infra is a hotdog-fit
  portable candidate on its own. Ledger now: smelt fuzz > prime-agent gate > ouroboros nightly >
  gemini-cli human-click > openhands smoke (flagged). ouroboros alt reading (fallback 8) = 74.0,
  band-stable either way.

## jazz (T2, 2026-09-30)
- 76.5 B self-flagged 1.5 under A floor -> boundary-jazz dispatched (hinges: interop 7-vs-6,
  token-economy ladder measure-vs-estimate, evals-harness-not-in-CI keeps verification 8).
- token-economy convergence feed: jazz 4-rung ladder (0.5/0.7/0.8/0.95) + cache-breakpoint
  discipline joins cluster w/ codex cache-key gating, pi projected-trigger, grok-build
  two-pass-prefire, qwen tiers. Synthesis: compaction-tiering vs cache-monotonic-compaction
  split gets a 5th datapoint; ladder-with-cache-breakpoints is a strong hotdog-fit portable
  (no model calls, pure arithmetic, works with user-paid tokens).

## amazon-q-developer-cli (T2, 2026-09-30)
- 65.0 EXACTLY on B floor -> boundary-amazon-q dispatched. Hairline: broken-as-configured CI
  gate (rust.yml:77 excludes absent fig_desktop-fuzz crate); fallback reading 64.25 C.
  Adjudication frame given to reviewer: does a stale exclusion make the gate exist-but-inert
  (docks verification) or is it trivia (keeps B)? Ruling becomes ledger precedent for
  broken-as-configured CI across the corpus.
- Shipped-behavior rule applied: two permission frameworks coexist (unconditional Allow for
  ExecuteCmd at permissions.rs:62-69; read tools checked against WRITE path lists at :47-61) -
  safety scored against what ships by default ONLY; both findings stand regardless.
- Generated-code accounting first formal use: 203k smithy SDK LOC = zero arch credit, zero
  god-file blame (same rule deepagents fixtures, auto-code-rover blobs - now a stated trio).

## tura (T2, 2026-09-30) - inert-safety family tally
- 72.0 B. Family count for "advertised-but-inert": phantom-approval UI tura (TUI renders
  /approve, zero enforcement emitters in Rust) joins claurst phantom-toggle, gptme/nanocoder
  fail-open, trae-agent partial-coverage, prime-agent removed-docs, openhands delegated-
  NeverConfirm. SIX subjects now; two are phantom-CONTROL-SURFACE specifically (claurst, tura).
  Sub-class named: enforcement-less-UI - the UI promises an approval flow the engine lacks.
  Strong convergence candidate; roadmap read: hotdog's permission UI must render ONLY states
  the engine can actually honor (test: every UI permission affordance has a binding hook).

## octomind (T2, 2026-09-30)
- 79.5 A - first T2-pool A claim; 1.5 over floor -> boundary-octomind dispatched (durability/
  architecture notches + token-economy 9 measure-vs-claim). Watch: if boundary pins <78, the
  "T2 pool produces no A" pattern holds and A becomes giants+anchors only so far.
- census again blind to inline Rust *_tests.rs (3,976 claimed vs ~105.5k real): seventh instance;
  census-methodology finding now needs its own findings.jsonl record at synthesis.

## maki (T2, 2026-09-30) - fail-open variant
- 74.5 B, no boundary (3.5 clear). New fail-open variant for family ledger: HOOK-CHAIN TIMEOUT
  fails open (hook.rs:95-100: 30s timeout -> Unchanged -> execution proceeds) - slow hook =
  bypass, attacker-controllable when the hook itself waits on anything external. Distinct from
  missing-emitter (tura) and default-off (gptme): this one converts LATENCY into permission.
  grammar-parsed-bash-policy now x10 across corpus; code-mode (monty) pairs with codex v8.

## codebuff (T2, 2026-09-30)
- 67.0 B exactly 2.0 over B floor -> boundary-codebuff dispatched (swing: verification 5-vs-4
  on 151.8k test LOC with public CI running build+smoke only; token-economy 9 re-check).
- NEW COVARIATE CLASS: public-mirror-of-private-development (squashed, pr-hygiene replay).
  Boundary mandate includes whether durability/verification/operability credit from a mirror
  needs an openhands-style marker ("dev-in-tree presence"). Related family so far: codebuff
  only - watch zeroclaw/continue/community mirrors in T3 wave for same shape.

## memcode (T2, 2026-09-30) - repo-shipped hook auto-run
- 75.5 B, no boundary. SAFETY-HOLE, high impact: .memcode/hooks.json SHIPPED IN THE REPO runs
  at session start (runtime/hooks.go:21) - opening a cloned repo = executing attacker-chosen
  config as the agent. Canonical concept: hook-trust-scoping (seeded id) - first instance;
  pattern: no workspace-trust gate before executing repo-provided hooks. Watch claurst (it had
  project-settings hook injection at core/src/lib.rs:1962) - if same shape, sub-family
  "clone-and-run" with x2+; gemini-cli policy work is the countermeasure reference.
- Vendored-fork mass (internal/forks/vaxis 41.5k) folded under generated/vendored accounting
  rule (zero credit/zero blame) - fourth use.

## kolkrabbi (T2, 2026-09-30) + B-ceiling cluster declaration
- 77.0 boundary dispatched (arch 9-vs-8 hinges CI-blocking ratchets; verification 8-vs-7 on
  seed-only fuzz + never-run pre-registered KolkBench; safety 7-vs-6 tested-but-opt-in jail).
- CLUSTER NOW UNMISTAKABLE (B ceiling): 77.0 opencode(final), 77.0 kolkrabbi*, 76.5 cline(final),
  76.5 kimi-code*, 76.5 jazz*, 76.0 gptme*, 76.0 deepagents* - seven subjects in [76,77.5),
  ALL with tested approvals + no kernel enforcement + no blocking model-verification. tier-list.md
  will carry a "B-ceiling plateau" note; boundaries decide if ANY of them are A on second look.
- census ate docs/site md into non_test (141.6k claimed vs ~55.2k Go) - glob-blindness list grows.

## kolega-code (T2, 2026-09-30)
- 76.5 B, boundary dispatched (safety 6-vs-7 vs 5: COMPOUND-COMMAND rule bypass - bypass-of-
  existing-enforcement scores worse than absence; orchestration resume-journal tested?).
- Census license misparse resolved: BUSL-1.1 both file+manifest; "AGPL" was the Change-License
  line (converts 2030-08-12). License-risk filed regardless; concepts-only enforced.
  Census-blindness list adds LICENSE parsing (Change-License line).
- Plateau now: FOUR subjects at exactly 76.5 (cline final, kimi-code*, jazz*, kolega-code*) plus
  77.0 x2, 76.0 x2. The B-ceiling note in tier-list will quote the shared DNA: tested approval,
  no kernel layer, no blocking model-verification, bus-factor or durability dock.

## forge (T2, 2026-09-30)
- 71.25 B, no boundary. License resolved: Apache-2.0 canonical (ISC = private npm wrapper only)
  - code-adjacent portables NOT blocked (unlike BUSL/AGPL subjects).
- eval-harness-not-in-CI tally now x3 + variants: deepagents (workflow_dispatch-only), forge
  (14 behavioral suites w/ jq asserts, ZERO wiring, and ci.yml carries the eval API KEY anyway),
  jazz (harness unrun). Canonical candidate evals-in-ci nuance: "instrumented but unenforced".
- New inert-safety variant: UNDOCUMENTED-SAFE-OFF - real tested policy engine disabled by
  undeclared restricted=false + first-run **/* allow-all (forge). Distinct from self-labeled-
  weak (kolkrabbi, honest) - hidden-unsafe-default is the aggravated form; feed to family ledger.

## kimi-cli (T2, 2026-09-30) - first archived subject in upper half
- 73.0 B; remote-verified archived:true -> rule (b) cap AT B formally applied (non-binding at
  this score, recorded for the tier-list footnote: highest legitimately-B-due-to-cap subject so
  far; without cap still B, so no ranking distortion).
- kimi-cli <-> kimi-code LINEAGE RESOLVED as predecessor/successor REWRITE (klip-11 rename doc,
  tombstone packages, parity telemetry approval.py:39-43) - not a fork; rule (a) inapplicable;
  both scored independently (73.0 / 76.5*). Study convention: rewrite-lineage pairs are ranked
  separately, with a lineage note; only code-descent triggers rule (a).

## crab-code (T2, 2026-09-30) - claude-code-family candidate #5
- 72.0 B, no boundary. License-risk: "built from scratch" claim vs per-module maps to
  proprietary CC internals (micro_compact.rs:8 etc.), CONFIDENCE MED - unlike claw-code-agent
  (32 in-code port declarations) this could be post-hoc design mapping, not source contact.
  SYNTHESIS TASK: treat as unconfirmed family member; headline finding counts only
  high-confidence descents (currently 4: free-code, claw-code-agent, claurst, kode-cli);
  crab-code gets its own audit pass only if another reviewer independently corroborates.
- Rust inline-test blindness count keeps climbing (2,905 claimed vs ~64k real) - the census
  finding is now statistically dominant for Rust subjects (6/6 wrong, all undercounts).

## claw-code (T2, 2026-09-30) - family #5 (self-declared) + worst runtime lie yet
- 53.5 C, no boundary. Census "original" FALSE: self-declared claude-code rewrite
  (src/__init__.py:1, parity_audit.py 1:1 TS map, reference_data/ surface snapshots).
  FAMILY LEDGER FINAL SHAPE (for headline): 4 high-confidence source-contact descents
  (free-code, claw-code-agent, claurst, kode-cli) + 2 spec-derived derivatives (claw-code
  SELF-DECLARED - admission removes ambiguity about intent, not about source access - and
  claurst's family classification) + 1 unconfirmed (crab-code, audit pending).
  Headline sentence must use: "5 of 48 reviewed subjects carry documented claude-code
  lineage by their own artifacts" - defensible at high confidence.
- SAFETY WORSE THAN INERT: jail fails open to bare sh AND THE RUNTIME TELLS THE USER
  "filesystem isolation is still active" (main.rs:4599). False safety indicator printed at
  fail-open = rung 2 crystallized. Family ledger: "runtime-lie" variant, first instance;
  roadmap: any isolation-degradation message in hotdog must state what was LOST, never
  what remains; test both the fail-closed behavior and the wording.

## san (T2, 2026-09-30) - seeded concepts getting real occupants
- 74.5 B, no boundary. cache-monotonic-compaction (SEEDED id) first clean occupant: compaction
  never moves the cache prefix, provider-anchored measurement. Feed: codex cache-key gating,
  grok-build fork-cache-alignment, octomind keepalive, jazz breakpoints - synthesis should
  check whether these all fold under cache-monotonic-compaction with per-subject rungs.
- LLM-judge polarity pair forming: san fail-CLOSED judge vs gptme fail-open auto-confirm vs
  maki timeout-fail-open. Rung structure: judge exists+fail-closed > exists+fail-open > absent.
  Roadmap-fit: hotdog LLM-approval judge must fail closed w/ budget cap (ouroboros $-guard x
  san polarity).
- library-core covariate: loop engine in sdk-go (deepagents boundary ruling will govern this row).

## boundary-amazon-q-developer-cli RESOLVED (2026-09-30)
- FINAL: 59.5 C (boundary recount) supersedes 65.0 B (provisional). Not the dispatched reason:
  the rust.yml:76 `--exclude fig_desktop-fuzz` (absent crate; zero fig_desktop hits repo-wide)
  is TRIVIA -- cargo warns-and-continues on unmatched --exclude under --workspace, so the
  push-gated job tests all 10 real crates. LEDGER PRECEDENT for broken-as-configured CI: dock
  only if the gate (a) silently skips real tests or (b) runs red-and-ignored; a no-op exclusion
  is neither.
- The B->C move is lane recalibration: interop 6 (no SDK/ACP/LSP/IDE surface), orchestration 4
  (no shipped subagents/queue/loop-detection), durability 6 (remote-verified: ls-remote HEAD ==
  snapshot 15cc8f3, static 5 months; not archived, rule (b) NOT triggered), originality 6.
  Lenient sensitivity bound 63.0, still C.
- Shipped-behavior rule applied: chat_cli is the shipping binary (Cargo.toml:4); its approval
  engine binds and is tested (chat/mod.rs:2390-2406, execute/mod.rs:303,377,415), zero sandbox
  -> safety 6. Both agent-crate permission flaws (permissions.rs:47-61,67-69) are unreachable
  (agent/src/main.rs:1-21 stub) -> findings b1/b2 as pre-ship landmines, no shipped-score dock.
- Generated-code accounting: 203,219 smithy LOC (DO-NOT-EDIT markers) zero-credit/zero-blame;
  hand-written 77,127 with ~11.8k inline tests. Fourth formal use of the rule.

## octomind (T2, 2026-09-30) - boundary DEMOTION 79.5 (A) -> 74.25 (B)
- Independent re-review confirms the original reviewer's fear but locates it on THREE axes, not one:
  (1) safety 5 (not >=6): every enforcement layer ships disabled - sandbox=false
    (config-templates/default.toml:33), LLM authorizer=false (:750-751), guardrails need a
    user-authored file, and NO built-in approval prompt exists in the tool path
    (tool_execution.rs:301-363); nanocoder precedent "real machinery + wrong default = below
    the 6 rung", honest variant (no false safety indicator, unlike the claurst runtime-lie).
  (2) token-economy 8.5 (not 9): fold-economics gate + idle keepalive ARE tested (decision.rs:31-70,143-146;
    amortization_tests.rs:36-172; cache_keepalive_tests.rs:43-224) and DO measure real economics
    (published provider price ratios, usage-checkpoint growth, realized savings display) - but no
    cache-key/prefix-identity control (codex's 9 pillar) and no realized-cache-hit validation;
    sits at jazz's 8.5 rung, equal-to pi's pillar set, not above.
  (3) architecture 7.5: errata god-file check PASSES (largest product file display.rs 4025 < 5k),
    TUI/ACP/websocket all drive one ChatSession core (no nanocoder parallel loops) - but
    main_loop.rs:15 "orchestrates all session operations" (2183 LOC) + ChatSession god-struct
    (core.rs:185) block 8.
- Verification 8 CORRECT per errata ceiling: 3943 test fns / 110k test LOC, 3-OS CI matrix +
  coverage + cargo-audit; live model matrix #[ignore] (authorizer_live_tests.rs:21), SWE-bench-Live
  bench/ harness is box-run not CI, no fuzzing. bench/README.md is the cheapest wire to a 9 claim.
- Durability 5 stands (solo, no SECURITY.md; offset by org CI + audit + release cadence); shallow
  clone noted. A floor held: NO. No calibration rule (fork/dead) applies - straight merit demotion.
- Convergence feeds: priced-fold-economics (new id, no anchor parallel) joins cache economics
  synthesis; validated-finish-gate: octomind gate.rs is now strongest occupant (evidence hierarchy
  + readback + learning-label polarity); prompt-cache-warming gains TTL-ping-with-provider-refusal
  nuance; safety ledger: honest-opt-in variant added next to nanocoder fail-open and claurst runtime-lie.

## Dispatcher acceptances + waveloom (2026-09-30)
- ACCEPTED boundary demotions (first of study): amazon-q 65.0 B -> 59.5 C (mean 62.25 also C,
  band robust; boundary authoritative per claw-code-agent precedent - no recursion), octomind
  79.5 A -> 74.25 B (straight merit demotion; "T2 pool produces no unchallenged A" holds).
  Both provisional reviewers had self-flagged the exact fragility; boundary found the mechanism.
- PRECEDENT SET (broken-as-configured CI): dock verification only if the gate (a) silently
  skips real tests or (b) runs red-and-ignored. No-op exclusions = trivia. Saves the corpus
  from cargo-cult docking of cosmetic workflow rot.
- PRECEDENT REINFORCED (enforcement defaults): octomind docked 6->5 because EVERY layer ships
  disabled (sandbox=false default.toml:33, authorizer off, no built-in approval prompt) -
  nanocoder's "real machinery + wrong default = below the 6 rung" now applied twice
  (octomind honest variant, forge hidden variant). Safety scoring doctrine = shipped default
  posture, full stop. Remaining 40+ reviews can stop debating this.
- waveloom 72.5 B: CLONE-AND-RUN hooks x2 now (memcode + waveloom main.go:739-789; claurst
  settings-injection variant x3). Sub-family solid; hotdog audit item upgraded from "check" to
  "must-have trust gate + test" for anything workspace-sourced.
- token-economy 9 outside anchors keeps multiplying (waveloom monotonic decision set +
  measured injection economics 26% of cache-miss tokens, loop.go:1110): cache economics is the
  dimension where solo projects MATCH institutions - contrast safety, where they can't afford
  kernel work... except forge-norvialabs did. convergence.md gets a paragraph on exactly which
  dimensions are accessible to small teams.

## nausicaa-harness (T2, 2026-09-30)
- 73.5 B, no boundary. fencing-token leases + recovery journals (tested) = session-turn-lease
  candidate now x2 (hermes-agent + nausicaa) - promotion to canonical at synthesis if any T3
  adds a third. Crash-recovery-posture cluster feeds also x2.
- Attribution hygiene datapoint: commit-pinned MIT attributions to Pi/Prime Agent/DeepSeek code
  - the POSITIVE pole of the attribution-spectrum findings (vs claw-code denial, claurst
  refuted clean-room claim, kode-cli silence). Spectrum: pinned-attribution > declared-fork >
  silent-fork > denied-derivation. Cheap screening signal, strong predictive value so far.

## agentty (T2, 2026-09-30)
- 76.5 B, 1.5 under A floor -> boundary-agentty dispatched. NEW RUBRIC QUESTION on its back:
  bounded fuzz + ASan/UBSan/TSan matrix - does it meet smelt's bar (oracled + seed persistence
  + replay) for verification 9? Ruling sets the systems-programming verification standard for
  the remaining reviews (crab-code, codewhale, vtcode, jcode T3 lanes will cite it).
- Second non-giant with enforced-by-default OS sandbox (forge-norvialabs was first) - if safety
  7 confirmed, the "only giants afford kernel enforcement" claim is officially dead: two solo
  C++/Rust projects did it. Roadmap: this is hotdog's single most defensible investment.

## zot (T1, 2026-09-30)
- 67.0 B at exactly +2.0 over floor -> boundary-zot dispatched (inclusive-2 rule; codebuff
  precedent). Swing: safety 6-vs-5 (yolo default args.go:95-108 + substring deny-list - the
  substring matcher is bypassable BY CONSTRUCTION, quoting/concat, worse than regex).
- zotfile permission-manifest packaging (zot-1): repo-declared capability manifest concept
  joins roadmap portable pool (pairs with sandbox-delegation cluster).
- B-ceiling-adjacent density continues: 67.0 x2 (zot, codebuff), plateau band [65,68] forming
  UNDER the famous 76-77 one - tier-list may need two annotated clusters.

## tau (T1, 2026-09-30) - declared design-lineage to an ANCHOR
- 56.5 C, no boundary. loop.py:1 self-declares "Pi-style Python port" - attribution rung 1-2
  (declared), rule (a) n/a (no code descent, verified). Notable for study framing: tau is a
  direct-to-ANCHOR design port - the field is now demonstrably copying pi's shape, not just
  converging on it; convergence.md can cite tau as the field's first explicit "port of the
  anchor" datapoint. Its score (pi 78.5 - 22) quantifies what "same shape, lower discipline"
  costs: no cache warming, no orchestration, thin interop - the loop was the easy 20%.

## neovate-code (T1, 2026-09-30)
- 62.0 C. BOUNDARY DECLINED: reviewer said "within 2 of 65" - arithmetic wrong (65-62=3.0).
  Outside rule; no dispatch. Dispatcher arithmetic check now explicit: verify claimed distance
  to cut before dispatching (this is the first declined flag; flags are honored at face value
  otherwise).
- SAFETY-HOLE, strong new variant: APPROVAL-SCOPE AMNESIA - ACP always-allow cached at
  toolName:category granularity and replayed without re-checking per-command risk
  (acp/session.ts:113-137). Approve once for a CATEGORY, future specific dangerous commands
  ride the cache. Distinct from fail-open (no judgment happened) and inert UI (no engine):
  judgment happened, on the wrong key. Rung: 5 with the hole; hotdog rule: approval caches
  must key on COMMAND-EQUIVALENCE CLASS with explicit scope narrowing, never category.
- turn-cap deflation (turnsCount -= approvedToolUses.length, loop.ts:737): budget accounting
  that credits itself - anti-pattern candidate "self-crediting budget".

## plandex (T1, 2026-09-30) - rule (b) by-a-hair edge
- 55.0 C. pushed_at 2025-10-03 = 361 days: dead-rule misses by FOUR DAYS because we happened
  to review on 2026-09-29. Cap moot (C < B anyway) but synthesis convention: rule (b) is
  evaluated as "would a reviewer in a normal window call this dead" - a 361-day stale repo
  with cloud wind-down commits at HEAD IS dead-in-practice; plandex gets a footnote
  "dead-but-uncapped by timing accident" rather than pretending the line means something at
  n=4 days. Same convention pre-applied to any subject within 30 days of the 12mo cut.
- Concept pool additions: tree-sitter-anchor-edits (edits anchored to parsed syntax nodes -
  pairs grammar-parsed-bash-policy x10 as "grammar-parsed editing"), overflow-summary-graft,
  auto-debug-repair-loop (cross-check vs auto-continuation-goals x2 when synthesizing).

## keen-code (T1, 2026-09-30)
- 63.0 C at exactly -2.0 under B floor -> boundary-keen-code dispatched (inclusive-2; contrast
  neovate 62.0 at -3.0 declined). 63-65 zone now has THREE boundary calls in various states -
  synthesis will compare the three for consistency (codebuff +2 over, zot +2 over, keen -2 under).
- Go colocated-test blindness x4 now (crush, memcode, kolkrabbi, keen). Rust x6. TS x5. Census
  methodology appendix has its own sample size problem: it is the study's most replicated finding.

## mocode (T1, 2026-09-30) - non-kernel safety ceiling moves up
- 70.5 B, no boundary. SAFETY 7 CONDITIONAL = strongest non-kernel enforcement observed
  (fingerprint grants, fail-closed off-TTY, hash-gated skill trust, realpath jail, ALL tested).
  Non-kernel safety ladder now: 7 mocode > 6.5 maki > 6 crush/pi/cline-rung > 5 deny-list/
  scope-amnesia > 4 phantom/hidden-off. If boundary-free, tier-list intro needs this line:
  "the best non-kernel safety in the corpus is a T1 solo project" - same story as token-economy.
- fingerprint-grants + hash-gated-skill-trust join portable pool; skill-trust hashing pairs
  directly with hotdog's extension/skill loading audit item (mocode-? evidence at synthesis).
- Identity discipline held in the mode-cluster: no cross-import between mocode/mini-kode/
  minicode/nanocoder - appendix doing its job.

## devon (T1, 2026-09-30) - census overcount + literal "test copy/"
- 33.5 D, no boundary. Census test LOC off ~17x INFLATED direction: glob ate vendored swe-bench
  experimental incl. a directory literally named "test copy/" - the corpus's funniest census
  artifact; blindness list now spans under- AND over-count (devon, auto-code-rover, deepagents
  inflate; TS/Rust/Go globs deflate). Methodology note: glob census errors are bidirectional,
  which is worse for naive tiering than any consistent bias would be.
- devon-9: README badge says Apache-2.0, LICENSE says AGPL - fourth license-posture finding
  (kolega misparse, darce manifest-only, crab... ) => license verification is a required
  screening step, badges are claims (docs are not evidence - original protocol rule, vindicated).

## ra-aid (T1, 2026-09-30)
- 49.5 C, no boundary. Bypass taxonomy addition: APPROVAL-BYPASS-VIA-INTERPRETER-EVAL -
  shell approval tested+real (shell.py:89-109) but eval() w/ default builtins in the agent's
  own code path (ciayn_agent.py:345,533,691) reaches os.system without approval, AND docstring
  claims a nonexistent sandbox (:122). Pair with neovate scope-amnesia: bypass-by-reuse (wrong
  cache key) vs bypass-by-escape-hatch (eval). Rule for hotdog: no eval/exec in agent runtime;
  tool layer is the only side-effect door - test by grepping own tree for eval/exec at CI.
- shallow-HEAD AGAIN understated activity (remote 2026-01-30 vs snapshot 2025-06-16) - third
  inactivity-false-positive avoided; rule (b) discipline continues to pay.

## dvalincode (T1, 2026-09-30) - monotone-perm idea
- 71.0 B, no boundary. Originality pool: NARROWING-MONOTONE POLICY with un-raisable hard
  blocks (session permissions can only tighten; hard blocks can never be widened mid-session)
  - safety-side twin of cache-monotonic-compaction: an invariant that makes a whole attack
  class unrepresentable. Joins top-of-pool candidates w/ fingerprint-grants + hash-gated
  skill trust. Roadmap-fit excellent: pure invariants, zero-dep, no kernel needed.
- Also novel: "trust self-report" + re-derivable fix records (audit trail reconstructible
  from policy+transcripts, not stored verdicts). Feed security-posture-docs ledger at synth.

## zap-coding-agent (T1, 2026-09-30)
- 56.0 C, no boundary. hook-trust-scoping hits x3+ (memcode, waveloom, zap; claurst variant) -
  sub-family CONFIRMED canonical at synthesis. test-suite-without-ci now its own recurring
  anti-pattern (zap 378 semantic tests NEVER executed - worst-case instance: tests exist,
  zero CI). Manifest-only-license x3 (zap, darce, g3) => license-risk screening formalized.
- skill-token projection into compaction trigger (zap) = lazy-skill-loading x token-economy
  crossover; if T3 lanes show same, cluster "cost-aware lazy loading" forms.

## minicode (T1, 2026-09-30) - inverse-nanocoder
- 72.5 B, no boundary. NOTABLE SAFETY SHAPE: jail+bash-guard bind EVEN UNDER allow-all config;
  fail-closed explicit sandbox = exact inverse of nanocoder (real machinery, wrong default).
  minicode: machinery that config cannot disable = floor-invariant, pairs with dvalincode
  narrowing-monotone in the invariant-enforcement pool. Possible ledger item pending agentty
  fuzz ruling ("nightly seeded fuzz" - seed persistence? replay in CI?). If agentty ruling is
  strict, minicode stays 8; if lenient, minicode AND agentty move together - check consistency.
- Frozen vendored kernel w/ CI-pinned seams: interesting supply-chain posture (vendored dep +
  pinned seam tests) - feed supply-chain ledger (hotdog's own README carries supply-chain doc).

## g3 (T1, 2026-09-30) - minor row, folded
- 55.0 C, no boundary. Cache-economy cluster +1 (thin→compact→ACD dehydration tiers, rolling
  cache-breakpoint budget); manifest-only-license +1 (x4: zap, darce, g3, +). No decisions.
  NOTE going forward: minor clean rows fold into synthesis cluster feeds in batch instead of
  individual ledger entries - notes stay readable, evidence stays in findings/*.jsonl.

## codemachine-cli (T1, 2026-09-30) - orchestration-by-bypass
- 42.5 D, no boundary (2.5 under cut; needs verification>0 anyway - zero test files, verified).
- ANTI-PATTERN (new, strong): ORCHESTRATION-BY-BYPASS - all 7 rival-CLI adapters launch the
  engines with --dangerously-skip-permissions / --bypass-approvals-and-sandbox / --auto-approve
  with NO re-enable path. Its whole value prop = running other harnesses with their safety off.
  Distinct from delegated-confirm (openhands waits for someone else) and inert-local: codemachine
  actively DISABLES enforcement that WOULD otherwise bind in the subprocess. Roadmap hard line:
  hotdog spawning external engines must never pass bypass flags; test = grep spawn argv.
- Note for interop dimension: rival-adapter matrices (x2 now w/ aeon) score well on interop but
  this row shows interop credit can hide a safety sub-floor - synthesis cross-check both dims.

## hax (T1, 2026-09-30) - fifteenth boundary, doctrine-critical
- 65.0 EXACTLY on B floor -> boundary-hax dispatched. Its safety question (prompt-only +
  honest labeling: codel-2 or above?) plus kolkrabbi's honesty-credit question = THE two
  rulings that unify "does honesty about absent enforcement move rungs?" for final synthesis.
  Interim working doctrine for later reviews: honesty mitigates DECEPTION penalties (rung-2
  "misleading") but never substitutes for ENFORCEMENT (cannot lift absent mechanics above 3);
  hax ruling will confirm or amend this.
- 65-line population now: hax 65.0*, claurst 65.0*, amazon-q settled 59.5 C. Boundary picks.

## ipsupport-code (T1, 2026-09-30) - folded, two pool entries
- 71.75 B, no boundary. Pool candidates: (1) shadow-mode risk classifier with REGRESSION-GATED
  RETRAIN in CI (permission-policy meets ML-ops at T1 scale - watch for analogs in giants;
  if none, unique-class finding); (2) pitfalls-into-tool-errors (curated failure advice IN tool
  error text at the point of failure) - hotdog-fit S effort, pairs w/ docs-dx findings.
- Go glob-miss continues its streak (x5). sandbox-under-CI (sandbox tests execute in CI) is
  third instance - candidate concept "enforcement-tests-in-ci" separating 6 from 7 rungs.

## boundary-keen-code RESOLVED (reviewer, 2026-09-30)
- FINAL: 63.75 C (boundary recount; provisional 63.0 C). B floor NOT crossed.
- Token-economy claim verified in code but rung-placed cline-plus/pi-minus (7.5): trigger measures the serialized outbound request per tool turn (anthropic.go:593-596), boundary-tested exactly (auto_compaction_test.go:105), wire-tested dual breakpoints (anthropic_test.go:984+), TurnMemory retained-output projection real - but len/3 token heuristic, zero usage-feedback, no warming, no realized-cache-hit validation, no tiering. Below the san/jazz/waveloom measured-economics cluster, so the "only path to B" fails; generosity bound (verif 8 + token 8) = 65.0 exactly on the line without mechanism upgrade - band held.
- Verification 7.5: all 35.9k test LOC block (go.yml:29-30 go test -race on push/PR + govulncheck + CodeQL); property-asserting wire-capture idiom (b6); zero fuzz, zero evals, single OS, coverage non-blocking = ERRATA 8 unreachable, crush-7 outperformed on test shape only.
- Safety 6-rung CONFIRMED, no dock: enforcement ships ON (build default), and keen is the corpus's clean counterexample to neovate approval-scope amnesia - session grants never honored for elevated risk (requester.go:73), classifier independent of model flag (bash.go:151, bash_test.go:477). Sub-floor: subagent AutoApprover (tool_factory.go:13) filed as bounded safety-hole (b4). No hook-trust-scoping hole (MCP config user-level; subagent/skill defs markdown-only, no repo-shipped executables).
- Caveat disclosed: pre-writing concept dedup grep across findings/ incidentally surfaced one original-review line (keen-code-1, turn-memory-projection) before independence was fully fenced; boundary findings avoid that idea, none duplicated; six new records b1-b6 with reuse-first concept discipline (usage-measured-compact-trigger, prompt-cache-marking, permission-policy, subagent-permission-inheritance, regex-denylist, faux-provider-testing).

- hax BOUNDARY FINAL 66.0 B (floor HELD). RULING: honesty-credit = exactly one rung, upward only
  out of the misleading band (codel-2 -> hax-3); absent mechanics cap at 3 regardless of disclosure;
  enforcement always outranks honesty (kolkrabbi-7 stands on machinery). Interim doctrine at :698-700
  CONFIRMED verbatim; corpus rule for synthesis: rung2 = policy-as-prompt AND misleading, rung3 =
  policy-as-prompt honestly labeled, rung4+ = binding machinery, honest or not. Verification pick
  settled 8: ASan/TSan + BSD-QEMU + real-binary-mock-e2e all BLOCK on push (ci.yml:3-5,33-38,74-86;
  check.sh:100-107); 8-ceiling held (zero fuzz targets, no in-CI evals).

## ferrum (T1, 2026-09-30)
- 63.0 C, 1.5 under floor -> boundary-ferrum dispatched, explicitly bound to the now-settled
  doctrines (honesty-credit/hax, shipped-default, blocking-only verification, amazon-q CI rule)
  - later boundaries run on precedent, not improvisation.
- keen-code FINAL 63.75 C confirmed; hax FINAL 66.0 B; honesty doctrine CONFIRMED VERBATIM:
  rung2 = policy-as-prompt AND misleading; rung3 = honestly labeled; rung4+ = binding machinery.
  keen bonus datapoint: clean counterexample to neovate scope-amnesia (grants never honored for
  elevated risk, classifier independent of model flag) - approval-scope correctness now has a
  positive exemplar for the roadmap, not just violations.

## 3code (T1, 2026-09-30)
- 65.75 B within 2 of floor -> boundary-3code dispatched. Key questions: safety 7 with
  EXTERNAL fence engine (sandwall) + default-OPEN network - completeness borrowed from an
  out-of-tree binary + egress leak may dock to 6 (codex contrast: separate egress approval).
  anti-approval-kernel-fence thesis: implemented or manifesto? (design-position census: corpus
  so far has approval-first (codex), fence-first (3code?), and fence+approval (minicode floor).)
- dmail conversation-revert + byte-reproducible CI builds folded for supply-chain ledger.
- B-floor zone now ALSO has 65.75*/66.0(hax final)/65.0*(claurst) + 63.x trio - synthesis
  consistency sweep covers [63,68] as one adjudication band.

## molt (T1, 2026-09-30)
- 62.0 C. BOUNDARY DECLINED (neovate arithmetic rule): reviewer said "boundary watch" but
  65-62 = 3.0, outside inclusive-2. Consistent with neovate decline; no dispatch.
- Pool: anti-vacuous-test-gate (built-in check that tests aren't vacuously passing - pairs
  nicely w/ enforcement-tests-in-ci; both are "tests that verify the tests" meta-cluster).
- Identity hygiene win again: desktop-app manifest turned out to BE a real CLI harness; full
  rubric applied, no shell-covariate needed (contrast openhands).

## ob-1 (T1, 2026-09-30) - an observable LINEAGE CHAIN
- 67.5 B, no boundary. RARE: claude-code -> claw-code (self-declared rewrite) -> ob-1 (11
  "parity with claw-code's X" headers) = a two-hop DESIGN-GENEALOGY CHAIN observable in-repo.
  For convergence.md diffusion-vs-discovery analysis this is the first chain with both hops
  evidenced; concept spread no longer needs inference when headers name the parent.
- safety-hole: autopilot+no-sandbox defaults behind single /trust flip (config.ts:706-708) -
  shipped-default doctrine applies (enforcement one slash away from off = docked posture);
  adds to hidden/weak-default sub-family (forge hidden, octomind honest-off, ob-1 one-tap-off).
- pool: keyless free-model bandit router (cost-tiered routing w/o API keys) + tool-repair
  verified-failure escalation - both hotdog-fit candidates, check giants for analogs.

## agentless (T1->T0, 2026-09-30)
- 42.0 D; dead remote-verified (pushed_at 2024-12-22 all branches), rule (b) applied, moot.
  T0 criterion retro-met (dead >12mo + <10k real LOC); T1 spend accepted as sunk, tier-list
  footnote. Census: 32,032-line CSV miscounted as source (blob/fixture/lockfile/doc/COUNT=5 now).
- SAFETY-HOLE classic: eval() directly on MODEL OUTPUT (repair.py:188, postprocess_data.py:849)
  - not eval-in-runtime but eval-of-model-text: prompt injection becomes code exec by design.
  Joins bypass taxonomy as its own class: EXEC-OF-MODEL-TEXT. hotdog rule already covers
  (no eval) - this row is the citation.
- Originality 8 w/o machinery = the study's cleanest evidence for originality being its own
  dimension (funnel design influence > implementation); keep scoring originality separately
  from architecture exactly as rubric says.

## qqcode (T1, 2026-09-30) - impersonation family thickens
- 60.0 C, no boundary. Divergent-fork:mistral-vibe CONFIRMED with numbers (21/145 shared files
  byte-identical, 70 own files); rule (a) synthesis check pending mistral-vibe's T3 score.
- IMPERSONATION SUB-FAMILY now x2+: qqcode injects verbatim "You are Claude Code" prompt prefix
  to pass Anthropic OAUTH ENTITLEMENT (anthropic_sdk.py:29,58-61,339-344); claurst = deeper
  stealth stack. Pattern name for synthesis: vendor-protocol-impersonation (prompt-level and
  client-level variants) - distinct from lineage (they're not claude-code DESCENDANTS here,
  they're claude-code IMPERSONATORS at the wire). Combined w/ grinta rival-subscription-transport
  = "subscription laundering" concept cluster candidate; ToS-risk class question now has 3
  occupants - synthesis to define the kind (license-risk? new kind tos-risk?).

## binharic-cli (T1, 2026-09-30) - anti-pattern showcase row
- 40.0 D, no boundary. Verification-dishonesty sub-family established, three new canonical
  candidates: REPLICA-TESTS (37/88 test files never import src - tautological harness),
  TEST-ONLY-AGENT-LOOP (whole "advanced" agent layer reachable only from tests), and phantom
  safety systems with ZERO call sites (PermissionsManager, checkpoint registry). Distinct from
  test-suite-without-ci (those tests are REAL but unrun): binharic's tests run but test a
  parallel imaginary product. Verification 3 correct.
- Screening rule for synthesis + methodology appendix: import-graph check - % of test files
  importing production code; <50% = replica suspicion. Cheap, structural, catches what CI
  greenness cannot.
- Safety rung 2 exemplar (phantom controls, no runtime lie) sits between hax-3 (honest
  prompt-only) and claurst-2 (phantom + misleading docs) - rung 2's boundary against rung 3
  is documentation posture, exactly per confirmed doctrine.

## boundary-ferrum RESOLVED (2026-09-30)
- FINAL: 62.75 C (boundary recount) supersedes 63.0 C (provisional). B floor NOT crossed
  (2.25 short). Both dispatched 1-pt swings GRANTED and still misses: verification 5->6 =
  top of the zero-CI cap (claw-code precedent; corpus is genuinely property-shaped: ~17.3k
  inline + 1,752 integration LOC, tier matrices, fail-closed resource tests, compaction
  invariants, binary-spawning ACP tests, faux provider, 21-task bench vs Pi+OpenCode),
  safety 5->6 = shipped-default posture (grammar policy binds at default medium,
  fail-closed on every parse path, tier-independent catastrophic rejects; nanocoder-5 is
  the opt-in/fail-open shape, ferrum is not).
- Zero CI VERIFIED: 0 yml/yaml repo-wide, no .github/.forgejo/.woodpecker. The B claim fails
  on the rest of the board (durability 4.5, orchestration 5, operability 6.5), not on the
  contested lanes. enforcement-tests-in-ci (keen 6-vs-7 gate) marked unreachable without
  CI; CI absence charged once (verification), no double-dock in safety (gemini-cli ruling).
- Architecture 6 per pi-errata docking (agent/mod.rs 10,178 total/~7,076 non-test, loop+TUI
  fused; sink trait separates, file does not). Positives for synthesis: repo-config-
  monotone-narrowing (hook-trust-scoping positive exemplar #2 after keen's approval-scope),
  security-table-contract-test (new), fail-closed post-compaction send-block.
- Shallow clone honored: no activity claims either way; rules (a)/(b) n/a.

## coro-code + groq-code-cli (T1, 2026-09-30) - folded, D-band pattern set
- coro-code 42.0 D, no boundary (3.0 under 45). "TEMPORARILY DISABLE the access permission
  system" IS the HEAD commit - permanent-temporary disabling, sub-family w/ forge hidden-off;
  compaction trigger bound to COMPLETION budget not context window (core.rs:72) = miswired
  trigger class; --trajectory-file claims saved, never saves = PHANTOM-FEATURE reporting
  (ties to binharic phantom-controls, runtime-claim w/o behavior, lighter than claw-code lie).
- groq-code-cli 36.5 D, no boundary. NEW anti-pattern: RUNNER-BLIND TESTS - ava config glob
  orphans 3/4 of its own test files (package.json:53-56); distinct from test-suite-without-ci
  (here npm test ITSELF never sees them) - screening rule: test-file count vs runner-report
  count mismatch. startswith delete-guard bypass x3 now (claii, picocode-contrast, groq).
- ferrum FINAL 62.75 C (B not crossed; both contested lanes granted and still short - good
  precedent: boundaries grant merits and STILL hold bands when the rest of the board says C).
- D-band coherence: safety rung-2 variants (phantom/never-saved/temporarily-disabled) +
  trigger-miswire + runner-blind tests = anti-pattern section nearly complete from D rows.

## mini-kode (T1, 2026-09-30)
- 46.5 C, 1.5 over floor -> boundary-mini-kode dispatched (18th). Swing: verification 5-vs-4
  zero-CI cap check + runner-blind check.
- Path-security sub-family starting: SIBLING-PREFIX FS GRANT (/foo grants /foobar,
  pathChecker.ts:53 - contradicts own doc example) + BLACKLIST-LAUNDERING-VIA-AUTO-EGRESS
  (denied local file access reachable through auto-approved fetch tool). hotdog rule: path
  grants compare normalized path SEGMENTS never string prefixes; egress tools never
  auto-approved when local reads are gated (deny-set must be closed under reachable effects).
- mode cluster fully disambiguated now (4x4 checked, zero contamination).

## kilocode (T3, 2026-09-30) - 0.5 under A + DOCTRINE COLLISION flagged
- 77.5 B provisional; rule (a) differential properly documented (net +0.5 vs opencode 77.0,
  +safety +orchestration -architecture; sync-fork-reclass would cap). 0.5 under A floor ->
  boundary-kilocode dispatched WITH A SECOND MANDATE: resolve safety-7-with-default-allow vs
  nanocoder/octomind shipped-default doctrine (below-6 precedent). If doctrine wins, the
  outranks-opencode result REVERSES (the +1 safety differential dies). General rule needed
  for remaining giants showing strong-opt-in/weak-default shape (gemini-cli already noted
  "enforcement DEFAULTS" gap; several T3 pending).
- T3 merge quality excellent: 30 findings, cross-lane dedup, 6 concepts realigned to corpus ids.

## boundary-gptme RESOLVED (2026-09-30) - four rulings, exact-match band
- FINAL 76.0 B (exact match, 4th decimal-level boundary confirmation). Rulings logged:
  1) continue-on-error eval gate = TELEMETRY not enforcement (branch protection can't save it)
     - blocks-on-failure rule now CONFIRMED by direct inspection, not just inference.
  2) SAFETY NUANCE vs kilocode collision: gptme's no-hook auto-confirm stays at 6 because it
     is a DOCUMENTED OPT-IN HEADLESS CONTRACT (SECURITY.md:37) and hook crashes fail CLOSED.
     Emerging unified doctrine (finalize with kilocode ruling): rung keyed to default posture
     IN THE PRIMARY INTERACTIVE MODE + honest disclosure of headless opt-outs; silent flips
     docked (gptme's non-TTY->no_confirm filed as safety-hole, cli/main.py:1082-1088).
  3) ERRATA phrasing sharpened: ">5k product file" is a CEILING marker, not a <5k amnesty for
     4k fused routes+logic files - docking is by shape, not just size.
  4) token-economy 9 joins cluster: measured-shrink retry gate (chat.py:865-872) - compaction
     result verified against actual shrink before accepting. Cost-aware ledger grows.

## boundary-claurst RESOLVED (dispatcher accepted, 2026-09-30)
- FINAL: 54.0 C (boundary) supersedes 65.0 B (T2 first pass). Band flip B->C; adopted per
  no-recursion precedent (boundary authoritative after first flip). Mechanism: safety 4->2 -
  phantom /sandbox-toggle ("Functional" in docs, zero consumers) + repo settings self-grant
  ONE LINE below the sibling SECURITY guards (core/lib.rs:2003 vs :1992-2001 - the fix was
  adjacent code, not a boundary) + ungated sh -c hooks. Approvals bind+tested but phantom
  protection is the codel-rung tell per doctrine (binharic rung-2 exemplar consistent).
- verification 6 (not provisional's 4-propping-65... : provisional had verification higher;
  boundary: 34k LOC, 3-OS CI, real property tests, but no faux-loop tests + smoke existence
  checks cap at 6). Net -11.
- Independence-disclosure norm now x2 (keen, claurst): whole-corpus greps before scoring can
  surface original finding titles; both reviewers fenced and continued cleanly. Methodology
  appendix: boundary reviewers should scope dedup greps AFTER independent scoring, or by
  concept-id only.
- [63,68] adjudication band loses a member (claurst ->54 C); exactly-on-the-line population
  shrinks to hax-66-final + pending (zot/codebuff/3code).

## boundary-kimicode RESOLVED (2026-09-30)
- FINAL 77.0 B (boundary 77.0 vs provisional 76.5, same band; boundary adopted as the
  adjudicated row). A floor NOT crossed - kimi-code now SITS on the B ceiling w/ opencode 77.0.
- CORRECTION recorded: provisional's architecture dock (v1/v2 dual-engine) is STALE - no live
  v1 at HEAD (legacy flag read by zero sources, phantom CI job only). Errata lesson: docking
  reasons must be re-verified at HEAD, migration debt that FINISHED doesn't dock twice.
- Vendored-pi ruling: library dependency w/ documented pin + sync governance; NO god-file
  inheritance (pi's 6.8k product files were not vendored). Covariate question closed.
- Originality 8 CONFIRMED all-four-real: tree-sitter command policy w/ wasm DIFFERENTIAL ORACLE
  + known-diff registry + fail-closed-on-unanalyzable = current BEST occupant of
  grammar-parsed-bash-policy (x11) - roadmap: this is the reference implementation to study.
- New anti-pattern: CI-EXCLUDED TEST FILES - minidb WAL's 33 test files sit in vitest.config
  exclusion (zero CI jobs) - runner-blind sibling: tests visible to runner but runner
  excluded from CI. finding kimi-code-b1.

## boundary-trae-agent RESOLVED (2026-09-30)
- FINAL 36.0 D (boundary; provisional 44.0 D, same band, no flip - boundary rule did its job
  at the BOTTOM of the board too). CI runs 7/10 test files, Makefile hard-skips 33% of test
  corpus (rotted: patches nonexistent module), core loop zero test refs -> verification 4 per
  amazon-q branch (a): silently-skipped REAL tests DO dock. Blind 10x retry of context
  overflows -> token-economy 2. Docker sandbox = phantom-coverage confirmed (3 tools in,
  MCP/ckg to host), safety 3.
- Byte-band note: trae-agent 36.0 now sits beside cursor-agent 33.5 / groq 36.5 / darce 32.5:
  the "thin real harness with phantom controls" D cluster is coherent w/o rule (b).

## boundary-deepagents RESOLVED (2026-09-30) - LIBRARY-CORE CONVENTION SET
- FINAL 76.5 B (boundary +0.5 over provisional, same band).
- CONVENTION (covariate family doctrine, governs openhands/SWE-agent/san/library cores):
  score what the tree ENFORCES around the upstream loop; no docking for the loop's absence,
  no credit for loop internals; ARCHITECTURE CAPPED AT 8 - separation auditable only to the
  dependency boundary (graph.py:961). Symmetric, stated generally, adopted by dispatcher.
- Corpus-worst god file confirmed: app.py 31,203 LOC (4.4x next source) - errata symmetry
  bars arch-8-anyway; A unreachable on structure alone.
- Verification 8 re-confirmed the continue-on-error lesson a 2nd time (ALL eval workflows
  manual-trigger AND continue-on-error) - gptme+kilocode+deepagents now x3: manual-trigger or
  non-blocking evals = telemetry. Rule is beyond dispute.

## code (T3, 2026-09-30) - folded
- 68.5 B, closest crush. Provenance shape NEW: self-declared divergent-fork:codex w/ CRON
  30-MIN upstream ingest + tree ships BOTH a live upstream mirror (codex-rs/, never release-
  compiled) and the migrated product (code-rs/) mid rename-migration (MIGRATION_GUIDE). 
  Rule (a) moot (68.5 << 88.5). Corpus lesson: mirror-in-tree means census LOC double-counts
  the upstream - reviewer scoped to code-rs/ correctly (311k real vs 618k mirror).
- Architecture 5: chatwidget.rs 44,921 LOC (new corpus worst single-file, beats deepagents
  app.py 31k), 14.9k-LOC streaming.rs loop w/ 1,235-line entry, and THREE loop/state designs
  one of which never compiles (c1-c3). Weight-15 dimension decisive, as designed.

## aider (T3, 2026-09-30) - headline-grade "the field outgrew the pioneer" row
- 60.5 C (exactly nanocoder's score - the study's touchstone lands in mid-C). Famous pioneer
  with 199 contributors scores C because: 473 tests (12.4k LOC) after census correction (its
  "76k test LOC" = WEBSITE DOC TREE - census blindness variant: docs-as-tests), zero MCP/ACP,
  safety 5 (auto-approve + injection failure modes s1/s3/s5/s6), repo-map/originality 7 -
  "signature ideas absorbed into band defaults" (repomap pagerank now table stakes corpus-wide).
  docs-dx 9 with mass caveat (blog prose vs pi's engineering docs).
- Durability 6: 96% commits one author (3 aliases), H1-2026 cadence down ~62%.
- For tier-list narrative: name-value vs rubric-value divergence is largest here; publish
  aider's vector as the cautionary exhibit that rubrics measure 2026 artifact quality, not
  historical influence (which we track separately in lineage notes).

## oh-my-pi (T3, 2026-09-30) - rule (a) exemplary case, A earned
- 82.0 A. Divergent fork of PI itself (+3.5 over anchor). Integrator's rule (a) handling is
  the reference pattern: cap NOT applied because all excess credit sits in subsystems pi
  deliberately lacks (interop 9v6, orchestration 9v7, token 9v8); every shared-ancestry
  dimension matches or trails pi (arch 7<9) - "no upstream-synced code credited above where
  pi itself scored". Gray edge self-recorded: ongoing upstream merge playbook; if ever
  reclassified sync-fork, cap binds 78.4 and findings e1-e9 named as affected credit.
- A-cohort now SEVEN: 88.5/84.5/83.5c/83.0/82.75/82.0/80.0 - distribution 7/~91 = 7.7%, holding.
- token-economy pool additions of note: snapcompact (image-billed NO-LLM archive tier -
  compaction tier costing zero model calls), armed SPECULATIVE compaction w/ 927-LOC test,
  stored-history floor so compression extensions cannot deceive the trigger (trigger integrity
  against own extension system = hotdog-fit: our extension could lie about token counts).
- Architecture caution: 12,181-LOC AgentSession (328 methods) = fused-hub anti-pattern at
  A-band scale - A band tolerates arch 7; tier-list should say so out loud.

## bitfun (T3, 2026-09-30) - folded
- 74.5 B, closest cline-profile; weakest durability (solo personal account, 8-month repo at
  1.9M claimed LOC - scout LOC recount pending in report; watch LOC-vs-team plausibility note
  at synthesis). Design inheritance from codex by nomenclature, NOT code (adapters wrap
  codex/claude-code/opencode/pi as EXTERNAL harnesses = multi-harness-orchestrator shape,
  x3 now w/ codemachine+aeon - interop-credit-vs-safety-floor cross-check applies).
- Name collision functorz/BitFun checked+cleared (identity control working); remote verified
  active day-of-run per codel lesson.

## boundary-workground2 RESOLVED (dispatcher, 2026-09-30)
- FINAL: 78.25 A (boundary) exact match with the resumed T3 integrator score -- fourth exact-match boundary of the study (after cline, pi, opencode). All ten dimensions independently reproduced with own file:line paths; closest anchor codex upheld (lineage confirmed independently in the committed session log).
- Hinge check: operability 8.5 -> 8 would give 77.75 = B; operability 8.5 stands (fenced saves save.go:441-475, CAS leases session_lease.go:14-31, conflict tests save_test.go:76,125 -- strictly above the crush/cline 8 rung). Verification cannot rise (zero fuzz targets re-verified; evals manual-trigger/report-only per the standing 9-gate errata). A is final, no further review.
- ATTRIBUTION CORRECTION for synthesis lineage pass: the T3 integrator assumed `cache-impact.yml` (cache-impact CI gate) was lineage-inherited from deepseek-reasonix; the boundary reviewer grepped the reasonix tree and found ZERO cache-impact hits -- wg2 is the ORIGIN. Recorded as workground2-b2 (nuance). Originality 7.5 unchanged (the correction moves attribution between two scored subjects, it does not add new capability), but the T3 report's inheritance-audit table is wrong on this row; synthesis must not propagate "cache-impact gate inherited" into convergence.md lineage notes.
- workground2-b1 (leaked-proprietary-snapshot, med): boundary re-read of the committed Codex session log classified as a license-risk-adjacent finding; consistent with the T3 "provenance nuance, no rule-3 treatment" call -- kept as hygiene flag, no cap.

## boundary-open-interpreter lane result (boundary reviewer, 2026-09-30)
- From-scratch full T2/T3-depth pass (no anchors' reports read beyond the frozen excerpts): **83.0 / A**, band A stands, stored "D" string confirmed stale (policy cap demoted then reversed; this lane agrees with the reversal on the merits). Lane deltas vs the 78.5 triage: originality 2->6 (the ~20k-LOC rival-harness wire-emulation plane `core/src/harness/` is a corpus-unique tested mechanism, not config-plane), docs-dx 7->8; zero demolitions; verification held at the 8 ceiling (no in-CI evals, no fuzz targets).
- Adjudications: (1) divergent-fork:codex with honest attribution-preserving rebrand - NOT sync-fork (no upstream-sync automation; pinned rust-v0.154.0 baseline + manual rebase per RELEASE_NOTES.md:3-4; ~30k own Rust LOC, 6 own crates); rule (a) inapplicable and satisfied (83.0 < 88.5). (2) A/B: A, deciding dimension verification (377k carried test LOC + 5-OS nextest CI + fork delta tested in codex idiom; arch+verif = 25.5/30 locks the 78 floor).
- **Distribution check: this is the 5th subject at 80+ (with codex 88.5, codex-infinity 83.5, hermes 83.0, qwen-code 80.0) - ratio now past/at the 25% line. Dispatcher should re-anchor before scoring another giant, per the standing watch note.**
- New findings only (open-interpreter-b1..b5): rival-harness-wire-emulation (unique), verbatim-proprietary-prompt convergence x2 w/ claw-code-agent-2, oauth-client-impersonation nuance x3 w/ claurst/qqcode, scheduled-agent-runs x5, rival-harness-session-import x2 w/ cline-b1. Old provenance findings (open-interpreter-1..4) stay in findings/open-interpreter.jsonl unchanged.

## boundary-zeroclaw (boundary reviewer, 2026-09-30) - DEMOTION recorded
- Independent T2-depth re-score of the 78.25/A T3 integrator total: **77.5 / B**. Nine of ten dimensions
  independently reproduced at the main scores with own file:line; single delta = verification 8.5 -> 8.
- Hinge adjudication: the merge-gating CI eval (regression_suite.rs:21 via ci.yml:814 into required gate
  :1305) replays canned traces through a stubbed ModelProvider (replay.rs:1-3) - the same scripted-mock
  design that pins frozen codex at 8; no in-CI model evals, fuzz targets unwired (0 workflow hits, 3/5
  fuzz serde). Crediting 8.5 would place zeroclaw above codex on verification on a mechanism codex
  already has; the errata's ceiling applies verbatim. Boundary score authoritative: **zeroclaw final 77.5 B**.
  Safety 7.5 / token-economy 7.5 / docs-dx 9 all re-verified and held - had verification alone not moved,
  A would stand; no other dimension needs a second look.
- Census errata filed (zeroclaw-b3): Rust inline #[cfg(test)] blindness - test_loc 34,710 vs 543,372
  measured, non_test 1,063,392 vs 591,948. Provenance: original confirmed (remote == Cargo.toml
  repository, full history); rule (a) n/a.

## Wave 5: hotdog self-review FINAL (2026-09-30)
- 68.5 / B, closest anchor pi, all ten dimensions evidence-backed in subjects/hotdog.md; 15 findings (findings/hotdog.jsonl). No boundary flag (3.5 above 65, 9.5 below 78). S gate n/a.
- Self-review doctrine held: frozen anchors applied unmodified (pi god-file errata checked - max file 1660 LOC, none; verification-8 ceiling checked and NOT reached: single ubuntu CI job, coverage run-not-gated, no in-CI evals/fuzzing, no subprocess/PTY e2e). Absent machinery scored absent, no simplicity substitute.
- The score is unadjusted and un-advocated: safety-enforcement 5 = one below the pi/cline/crush 6 rung via the nanocoder "real machinery + wrong default" precedent (approvals off by default; best-tested gate in the corpus but shipped disabled; honesty exemplary and tested workspace containment credited - hence 5 not lower). Durability 4 weakest (solo, ~4 months, 19 stars). Originality 8 strongest, six mechanisms verified code-real and corpus-unmatched (marker-alias-mangling, render-anti-spoofing, cross-process lane ledger, cache-warm fleet placement, workflow run-dir ownership/resume, leaked-call-repair).
- 3 minor doc-vs-code drifts filed against ourselves (supply-chain 'no published artifact' vs publishConfig.provenance; CONTEXT.md log-property seam; compaction message-role wording). None misleading; docs-dx 8 held.
- For the roadmap: hotdog-8 (approvals off by default) is the top self-candidate under the roadmap rule (kind=anti-pattern excluded, but the underlying gap maps to the permission-policy default-posture cluster; roadmap writer should treat hotdog-8 as the anchor of the non-moves-vs-moves analysis, not as a portable).
- Distribution after hotdog: 8/103 at 80+ (7.8%) plus two pending boundary adjudications that could add at most open-interpreter (triage row pending; boundary says 83.0 A) -> worst case 9/103 = 8.7%. The "past the 25% line" alarm raised inside the open-interpreter boundary run was against the PARTIAL corpus mid-study; the full-corpus audit (wave6/score-inventory.md) is authoritative and the sanity rule passes.

## boundary-kilocode RESOLVED (dispatcher, 2026-09-30) + SAFETY DOCTRINE AMENDMENT (study-wide)
- FINAL: 78.5 A (boundary) supersedes 77.5 B (T3 provisional). First boundary band-flip that moved UP; boundary reviewer itself flagged it for the synthesis B-ceiling sweep. Sensitivity disclosed: the A turn rides entirely on verification 9 (release-blocking smoke gate qualifying under the corpus's blocking-only eval rule; precedent class prime-agent/openhands/gemini-cli). verif->8 gives 77.0 exact-tie opencode. SYNTHESIS MUST RE-AUDIT that single 9 against the gptme/deepagents x3 telemetry ruling before the A band is final -- this is the one outstanding verification-9 in the corpus below the A-anchors and it decides a band.
- Rule (a): kilocode (divergent fork of opencode) OUTRANKS upstream 77.0 by +1.5, divergence demonstrated (default bash-ask policy absent upstream, kilo-sandbox jail, prompt-queue/fork/board, fork governance; no upstream-synced code credited above opencode's own dimension scores; no sync markers). ADOPTED per divergent-fork-on-merit rule.
- DOCTRINE AMENDMENT (mandate 2) ADOPTED STUDY-WIDE: rung = rung earned by the binding DEFAULT posture, PLUS at most ONE rung for an opt-in enforcement layer passing three mechanical gates: (i) fail-closed (refuse-to-enable without backend; refuse-to-run, never fall back to unprotected, when the enabled path breaks); (ii) enforcement tests in CI in a blocking job; (iii) non-deceptive (no claim the default is protected; stale UNDER-disclosure of an existing jail = docs finding, not rung collapse; runtime false-safety indicator stays a rung-2 tell). Caps regardless: silent fail-open opt-in -> below 6 (nanocoder); zero binding default -> 5 (octomind). "Shipped default posture, full stop" survives as the floor rule; the +1 is auditable, not vibes. Reference application: kilocode 6 (bash-ask default, tested) +1 = 7. Applies retroactively to boundary rows already final only if a re-audit changes nothing; applies to all pending boundary reviews (orca-agent, opensquilla, codewhale) and any synthesis safety-disagreement sweep.

## Boundary band-flips ACCEPTED (dispatcher, 2026-09-30) -- completing the reconciliation ledger
- grinta-coding-agent FINAL: 74.0 B (boundary) supersedes 78.5 A. Verification 9 did not survive re-audit (trigger pipeline exists but is not a blocking eval gate per the confirmed blocking-only rule); the A floor cannot be reconstructed. Consistent with the kilocode flag: grinta and kilocode were the two verification-9 claims at the A floor; grinta's fell, kilocode's is queued for the same re-audit before synthesis finalizes.
- auto-code-rover FINAL: 41.5 D (boundary) supersedes 45.5 C. Dead + SONAR-Source-Available; boundary authoritative per amazon-q/octomind/claw-code-agent precedent.
- Reconciliation ledger now closed for all 7 audit-flagged band flips: amazon-q 59.5 C, octomind 74.25 B, claurst 54.0 C, claw-code-agent 50.0 C, grinta 74.0 B, auto-code-rover 41.5 D, kilocode 78.5 A (upward, verification-9 re-audit still owed by synthesis). Post-flip distribution shift: A 15->~13-14, B/C/D shift accordingly; wave6/score-inventory.md section updated at synthesis with final rows.

## boundary-codewhale RESOLVED (dispatcher, 2026-09-30)
- FINAL: 78.0 A (boundary) -- fifth exact-match boundary of the study; all ten dimensions independently reproduced at 78.0 (safety 7: kernel sandbox + live macOS denial tests clear the 6-rung despite Linux bwrap being opt-in -- survives the new kilocode doctrine's default-posture test on the denial tests; arch 6 boundary-gated engine; token-economy 8.5 survival-contract refusal last_round.rs:349-402). A stands; sitting exactly on the floor is survivable when every hinge independently re-earns its rung.
- PROVENANCE CORRECTION for the lineage pass: main review said original; boundary re-read finds rename-clone of deepseek-tui (LICENSE:3, app-server/src/lib.rs:37-42, npm/deepseek-tui package). Rule (a) NOT applicable in practice -- upstream not in the corpus, nothing to cap against. Synthesis: record lineage note in tier-list, keep 78.0 (renaming without upstream present changes no merit), but convergence/originality readers must discount the "original" label: any corpus-unique claims in codewhale findings may belong to deepseek-tui.
- Census carried: test_loc 61,627 -> ~261k+ (sibling tests.rs 222,850 + tests/ 38,134) -- census test-glob blindness keeps compounding (6th instance class); contributors=1 shallow artifact.
- Boundary reviewer filed ZERO new findings deliberately (all observations map to the 34 canonical T3 records) -- first boundary pass to add nothing; good conservation of convergence counts, precedent: don't manufacture boundary findings.

## boundary-open-interpreter RESOLVED (dispatcher, 2026-09-30)
- FINAL: 83.0 A (full boundary review) supersedes 78.5 triage row with stale "D" band. Adjudications adopted: divergent-fork:codex, honest attribution-preserving rebrand (pinned rust-v0.154.0 baseline, ~30k own Rust LOC, 6 own crates, no sync automation); rule (a) inapplicable and numerically satisfied. Deciding dimension verification (377k carried test LOC, 5-OS nextest CI, fork delta tested in codex idiom); triage's core miss was originality 2->6 -- the ~20k-LOC core/src/harness/ rival-harness wire-emulation plane is corpus-unique and tested. Verification held at the 8 ceiling (no in-CI model evals, no fuzz).
- DISPOSITION of the reviewer's "5th at 80+, re-anchor" flag: DECLINED as partial-corpus arithmetic (the same mid-study alarm class as the earlier queue note). Full-corpus audit: 80+ = 9/103 = 8.7% including this row -- sanity rule (>25%) passes with wide margin. wave6/score-inventory.md is the authoritative distribution; no re-anchoring.
- Convergence updates to fold at merge (post-audit, not in concept-ledger.md): open-interpreter-b1 rival-harness-wire-emulation (unique, new concept); b2 converges verbatim-proprietary-prompt with claw-code-agent-2; b3 oauth-client-impersonation x3 (with claurst, qqcode); b4 scheduled-agent-runs x5; b5 rival-harness-session-import x2 with cline-b1. Also open-interpreter-b2 is a license-risk-adjacent record on an A-band subject -- roadmap must apply rule 3 (concepts-only copying) for anything sourced from its verbatim-prompt surface.
- Boundary ledger status: open/decided -- remaining in flight: opensquilla, orca-agent. Owed synthesis task: kilocode verification-9 re-audit (blocking-only rule, grinta precedent).
## boundary-opensquilla RESOLVED (dispatcher, 2026-09-30)
- FINAL: 77.5 B (boundary) exact match with the main review on all ten dimensions -- sixth exact-match boundary. Band final.
- Hinge check under the amended doctrine: safety-enforcement 5.5 was the single path to A (5.5->6 = exactly 78) and FAILS the amended doctrine's own logic -- the shipped config self-declares an out-of-box bypass (opensquilla.toml.example:512-513,532; run_mode.py:28 collapses bypass->FULL) while docs/sandbox-security.md:10 claims a Safe default whose refusal path is unreachable (dispatch.py:419-441, zero origin_trace producers). That is the deceptive-default shape the +1 opt-in rule never rewards; 5.5 stands. Second hinge token-economy 9->9.5 also fails (router-accuracy marker excluded from every CI pytest run, zero tests).
- PROVENANCE: original confirmed (repo id 1231170332, fork:false; census upstream URL is org-migration legacy, not lineage). Rule (a) N/A. Census correction filed (opensquilla-b3): test LOC 723k vs census 647k (generated-code inflation, under- not over-count this time).
- opensquilla-b1..b5 filed under canonical concepts (injection-screening, permissive-default-vs-docs, generated-code-loc-census, loop-detection, evals-in-ci) -- fold into concept ledger at merge; permissive-default-vs-docs joins the rung-2/deceptive-default anti-pattern family next to claurst phantom-protection and forge hidden-off.

## boundary-orca-agent RESOLVED (dispatcher, 2026-09-30) -- boundary phase CLOSED
- FINAL: 78.75 A (boundary) exact match -- seventh exact-match boundary; all ten rungs independently reproduced including both 9s (orchestration, operability), which the main review flagged for scrutiny and this pass attacked directly: the 9-rung exhibits survived first-hand verification (runner.rs:3584 all-terminal repair property test; budget_controller.rs:22-29 lease pool; subagent_recovery_contract.rs:543 foreign-session rejection; execution_journal.rs:169 armed fault-injection). s4 silent-skip hole (macOS contract tests never run in CI -- runtime_lifecycle_contract.rs:2389-2400) confirmed real but books as band-bottom deduction, not a below-band signal. Neither hypothetical discount (orch 9->8.5 = 78.25; verif 8->7.5 = 78.0) was found warranted.
- Provenance: original, fork:false, bus-factor-1 re-confirmed via API; blade->orca self-rename residue noted (Cargo.toml:59, SECURITY.md:28) -- rename-RESIDUE is distinct from rename-CLONE (codewhale/open-interpreter class): own codebase, own name-change trail.
- Zero new findings filed (second conservation pass after codewhale).
- BOUNDARY LEDGER CLOSED: 21 boundary pairs adjudicated + 7 exact matches; the ONLY owed adjudication in the study is kilocode verification-9 (independent re-audit dispatched separately). All 103 corpus rows + hotdog self-row are otherwise FINAL for the tier list.

## kilocode verification-9 RE-AUDIT RESOLVED (dispatcher, 2026-09-30) -- LAST outstanding score question closed
- FINAL: verification 9 STANDS, kilocode 78.5 A is FINAL (first and only upward band flip confirmed, not reversed). Mechanical basis (kilocode-verification9-reaudit.md): live claude-sonnet-4.6 Harbor evals against draft-release binaries (smoke-test.yml:112-131), needs-edge gating (publish.yml:606), zero continue-on-error in either file -- passes the gptme blocking rule MORE cleanly than two 9s the corpus already kept.
- Precedent reconciliation done properly: grinta's 9 fell on the MISSING needs-edge (distinguishes, doesn't contradict); prime-agent's clean 9 independently re-verified fail-closed (behavioral-evals.yml:428-430). Main review's "release-gated not PR-gated" withhold rationale CONTRADICTS prime-agent/ouroboros precedents and is retired as a criterion.
- Retained caveat (flag, not demotion): pass-threshold scripts live in private kilo-bench, snapshot-unverifiable -- same trust class as openhands' kept 9. Documented, never silent. Boundary's other nine dims netted exactly 0 vs main, no re-exam.
- FOLLOW-UP DISPATCHED (consistency sweep, band-safe): the corpus verification-9 population is gemini-cli (human-click eval), openhands (flagged smoke), prime-agent (re-verified clean), kilocode (this pass), + any others -- audit the two soft ones (gemini-cli, openhands) against the now-mechanical three-part rule (live model behavior, needs-edge/blocking, zero continue-on-error). No bands ride on it; it protects convergence claims about what verification-9 means.

## verification-9 SWEEP RESOLVED (dispatcher, 2026-10-01) -- two adjudicated demotions, zero band changes
- Population from files: 9 = kilocode (final, re-audited), prime-agent 72.0, ouroboros 75.5, smelt 74.5, agentty 77.5, openhands 72.0, gemini-cli 84.5, deepseek-reasonix 86.0. ANCHOR CORRECTION: codex's verification is 8, not 9 (so is pi/grok-build/oh-my-pi/hermes-agent/open-interpreter/orca-agent) -- any downstream text claiming "codex verification 9" is wrong; anchors errata govern.
- DEMOTED (adjudicated corrections; score files intentionally unedited, the tier list applies these rows): gemini-cli 9->8 (eval gate exits 0 on CONFIRMED regression, scripts/run_eval_regression.js:93-101; nightly evals continue-on-error) -> weighted 84.5 -> 83.0, stays A. deepseek-reasonix 9->8 (live e2e comment-triggered only, e2e-bot.yml:9-32; fuzz targets never execute in CI -- the same machinery wg2 inherited and got scored 8 for, symmetric application) -> 86.0 -> 84.5, stays A.
- STAND: openhands 9 (real blocking verdict ci.yml:490-492, private-threshold flag retained), smelt + agentty 9 (fuzz-exception class), prime-agent (re-verified), kilocode (final).
- DEFINITION FOR PUBLICATION (tier list quotes this): verification-9 = a live-model (or CI-executed oracle'd-fuzz) gate on a path that cannot stay green when behavior regresses; tolerant steps plus a hard verdict count; continue-on-error, exit-0-on-regression, and manual-trigger gates are TELEMETRY; thresholds snapshot-auditable or an explicit retained flag.
- Post-sweep distribution: 80+ = 9/103 = 8.7% (gemini-cli 83.0, reasonix 84.5 both remain 80+); sanity rule passes. A-cohort final order shifts: reasonix 84.5 above gemini-cli 83.0.

## Ledger closure: unledgered boundary recounts ACCEPTED (dispatcher, 2026-10-01)
- Tier-list audit flagged 7 boundary pairs with boundary files but no dispatcher acceptance entry. ACCEPTED en bloc, boundary authoritative per precedent; ALL EIGHT were delta-small or same-band, no band flips -- ledger gap was clerical, not substantive: 3code 68.0 B (was 65.75 B), agentty 77.5 B (was 76.5 B; adopted, sits 0.5 under A floor but was already boundary-reviewed -> band final), codebuff 66.0 B (was 67.0 B), jazz 74.75 B (was 76.5 B), kolega-code 72.75 B (was 76.5 B, largest of the seven at -3.75, stays B), mini-kode 45.25 C (was 46.5 C), zot 66.0 B (was 67.0 B). kolkrabbi's unledgered "CONFIRMED" also accepted (exact match, no change).
- goose ARITHMETIC ADJUDICATED: stated weighted_total 73 is a sum error; lanes recompute to 74.5. FINAL goose row = 74.5 B (lanes authoritative, band unchanged). Score file left as written; tier-list row carries 74.5 with this entry as the warrant.
- Exact-match tally for drift notes: the "4th/7th exact-match" narrative was counting dispatcher-adjudicated RESOLVED entries; delta-zero boundary totals across the ledger are 9 (cline, pi, opencode, workground2, codewhale, opensquilla, orca-agent, + gptme decimal-level, kolkrabbi). tier-list drift note standardized on 9 with this footnote; per-entry ordinals in earlier notes kept as written history.
- Tier list band census post-everything: S1 / A14 / B49 / C22 / D17 = 103; 80+ = 9/103 = 8.7% (sanity passes); top row codex 88.5 unchanged. TIER LIST FINAL.

## Methodology footnote: harness-level failure observed during synthesis (2026-10-01)
- The roadmap deliverable worker died TWICE identically: assembled the full report and emitted it as one giant write tool-call payload; generation stalled mid-payload, stream idle-timeout killed the request (0 output tokens at the timeout line), retry looped, nothing reached disk -- and the task then reported "completed" with no artifact ("green empty success"). Observed directly on our own harness while reviewing 103 others for exactly this class of gap.
- Maps to the study's own taxonomy: no machine-checked artifact gate on the delegated-task path (the workflow engine has verdict files + output-file freshness for precisely this; plain delegate_task has none) => the delegated path is the harness's weak orchestration seam. Corpus analogues: announced-intent-no-artifact rows and the phantom-execution family.
- Fix applied operationally: re-dispatch with chunked-write protocol (intermediate ranking artifact first; sequential appends <=55 lines; read-back verification required before task end). Recorded as a candidate finding for the roadmap's T4 argument (machine-checked orchestration) -- our-thesis fit proven by live failure rather than argument.
