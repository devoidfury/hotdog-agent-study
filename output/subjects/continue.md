# subjects-continue: integrated T3 review (run 20260929-0849-t3-giant-review-3, node integrator)

Subject: Continue (`/data/samples/agents/continue`, upstream original continuedev/continue, head `5522c6f44` 2026-07-20, README-declared read-only).
Lanes integrated: reviewer-core (10 findings, arch 5.5 / operability 6), reviewer-safety (12 findings, safety 6 / verification 7), reviewer-econ (10 findings, token 6 / orch 5 / interop 7 / docs-dx 7).
Merged findings: `merged-findings.jsonl` -- 30 records from 32 inputs, deduped by concept (see Dedup log).

## Verdict

Continue is a large, genuinely tested, institutionally-backed coding agent whose engineering gravity sits in the wrong place: the agent loop lives in the VS Code GUI's Redux thunks with a full parallel re-implementation on the CLI, the one recursion guard is compiled out of production, the flagship headless "serve" plane is a hidden, undocumented, unauthenticated bind-all control server, and the subagent primitive works by globally switching permissions to allow-all. Against that: a 105,973-LOC behavior-asserting test corpus (census says 19k; census is wrong), the strongest non-kernel shell classifier in the corpus after codex, a genuinely good typed messenger boundary, and honest CLI permission docs. Weighted total **59.75 -> band C**. Closest anchor: **nanocoder**. The project is archived (README "no longer actively maintained", Final 2.0.0), so rule (b) caps at B -- the cap cannot bind at a C total; recorded, not silent, and charged to durability 3.

## Anchor placement (resolved across lanes)

Lanes disagreed and the disagreement is real but per-dimension: core placed architecture at the **nanocoder 5-rung** (loop inside a product UI state tree + parallel second implementation), safety placed enforcement in the **6-rung cluster (cline/pi/crush)** ("tested approval, nothing underneath", the serve hole a blemish not a rung-drop), econ placed the product shape nearest **cline** while pushing token-economy and orchestration toward crush/nanocoder. Integrated stance: **nanocoder** -- the score vector {5.5,7,6,6,5,7,6,6,3,7} sits L1-distance 4.5 from nanocoder's {5,7,5,6,6,7,6,7,4,7} and far from cline (76.5) or crush (69.0), whose strengths (architecture, orchestration, operability) are exactly continue's gaps. Cline stays as the shape note (MCP + multi-IDE + mintlify + cost fields); crush as the safety-6-rung contrast.

## Score reconciliation (explicit, per disagreement)

1. **The serve control plane, weighed twice in two currencies: safety (continue-s3) vs econ (continue-e8).** Same mechanism -- `serve.ts:388` `app.listen(port)` with no host and no auth, `POST /message` prompt injection, `POST /permission` self-approval, headless defaults `{Bash: allow, *: allow}` (`defaultPolicies.ts:31-34`), hidden command (`index.ts:312`). Safety kept enforcement at 6 (interactive defaults are ask-everywhere, approval semantics seam-tested); econ gave the identical fact zero interop credit (codel-2-shaped isolation theater). Both calls adopted -- they are not contradictory, they are different dimensions; the records were merged (canonical concept `undocumented-bind-all-control-plane`) with the cross-scoring note kept inside the record.
2. **The test-only depth guard: core continue-c3 vs safety continue-s9.** One mechanism (`streamNormalInput.ts:85`, `NODE_ENV === "test" && depth > 50`), two lane ids (phantom-safety-control vs loop-detection). Merged under the corpus concept `phantom-safety-control`, core record kept as base (its full recursion chain :354,:378,:390 -> callToolById.ts:143 -> streamResponseAfterToolCall.ts:83 is the stronger evidence), safety's poisoned-context impact and crush `loop_detection.go` contrast unioned. Both lanes rated impact high; no conflict.
3. **Census test LOC 19,409 vs measured 105,973.** Resolved to the **recount** (421 files, `.vitest.ts` suffix missed by census), per the crush/cline/nanocoder errata; verification scoring and every corpus claim use the recount.
4. **Docs-dx dock vs SECURITY.md silence.** Econ docked docs-dx 7 for e9 (docs assert ask-tools-are-excluded headless; code ships allow-all); safety's s12 flags SECURITY.md's bare reporting-only posture. Different facts (docs asserting a false guarantee vs a security doc omitting the true ceiling), so both stand with no double-dock and no double-credit.
5. **Safety's verification notation "7/15".** Read as score 7 on the 0-10 rung scale at weight 15, normalized in `lane_scores`.
6. **Same code, two dimensions -- kept, not merged:** `history.ts:81,131,178` non-atomic JSON underpins core c7 (session durability -> operability 6) and econ e6 (undurable orchestration state -> orchestration 5); the dual-stack fact underpins core c2 (architecture) and econ e1 (token-economy contract split). Each dimension docks the fact once; recorded to keep the overlap visible rather than silent.

## Scores (rung scale 0-10 each; weight in parens)

| dimension | score | weight | weighted | basis |
|---|---|---|---|---|
| architecture | 5.5 | 15 | 8.25 | core: nanocoder 5-rung shape (loop in Redux thunks c1, parallel CLI stack c2, test-only guard c3, 1,460-LOC chokepoint c4) with the half-credit for the typed messenger (c5) and 4.4k LOC faux-messenger loop tests (c6) |
| verification | 7 | 15 | 10.5 | safety: 421 files / 105,973 LOC measured, bypass-shaped classifier tests, seam-tested approval; 8 frozen out -- no fuzzing, no evals, live keys in unit CI, no coverage gate (s10) |
| safety-enforcement | 6 | 10 | 6.0 | safety: 6-rung "tested approval, nothing underneath" (s1, s2, s6); unauth serve plane (s3), GUI edit bypass (s4), preference-wins precedence (s7) hold it at 6 |
| token-economy | 6 | 10 | 6.0 | econ: solid CLI auto-compaction (e2) + cost fields (e4), but flagship surface summarizes only on click and silently drops oldest (e1), caching opt-in with per-dispatch prefix churn (e3) |
| orchestration | 5 | 10 | 5.0 | econ: subagent = global permission negation + service monkey-patching (e5), undurable queue/sessions/crash posture (e6); beats codel 4, below nanocoder 6 |
| interop | 7 | 10 | 7.0 | econ: MCP+OAuth, 67 provider adapters, 3 IDE hosts + binary, real headless `-p`/`--format json` (e7); no published agent SDK, serve plane earns nothing (e8 merged into s3) |
| operability | 6 | 10 | 6.0 | core: shared session manager + resume + CLI undo/rewind, but silent-corruption data-loss path (c7), no work checkpoints (c8), split crash posture (c9) |
| originality | 6 | 10 | 6.0 | integrator, verified in code (below) |
| durability | 3 | 5 | 1.5 | integrator, from census + git metadata; archived project (below) |
| docs-dx | 7 | 5 | 3.5 | econ: real 153-file tree mostly matching code (e10), docked for the security-relevant false guarantee (e9) |
| **weighted_total** | | 100 | **59.75** | **band C (45-64)** |

## My two dimensions, scored with code evidence

### Originality: 6/10 (weight 10)

Idea-vs-marketing check, done by direct read at `5522c6f44` (marketing claims unused):
- **Shell classifier is the real thing.** `packages/terminal-security/src/evaluateTerminalCommandSecurity.ts`: parse failure fails closed to `allowedWithPermission` (`:99` "be conservative and require permission"); variable-expansion dual evaluation -- empty-token detection (`:134-143`), both interpretations evaluated and combined with forced `allowedWithPermission` floor (`:146-168`); recursive command-substitution analysis that re-enters itself and refuses to stay auto-allowed (`:247-262`); pinned by 1,867 LOC of bypass-shaped tests (wc-measured). This is the strongest non-kernel command gate in the corpus after codex's execpolicy -- real mechanism, not prose.
- **Typed tuple-protocol messenger** -- `core/protocol/webview.ts:11+` maps each message to a `[payload, return]` tuple; one `IMessenger` interface over IPC, TCP and in-process transports (c5). Verified real; a small-interface/deep-implementation boundary worth porting.
- Docking from 7-8: every flagship mechanism has prior art already in this corpus (compaction, subagents, permission modes, MCP), and the good halves live on **one surface only** -- auto-compaction CLI-only, subagents CLI-only, the GUI ships a manual button plus silent oldest-drop (e1); the orchestration primitive itself is `{tool:"*", permission:"allow"}` (`extensions/cli/src/subagent/executor.ts:81-88`, comment "allow all tools for now"), which is the absence of a design idea; the historically pioneering product shape (dual-pane IDE agent + Hub) earns no mechanism credit at this band. 6.

### Durability: 3/10 (weight 5)

From census + `git` metadata on the snapshot:
- **Status: archived by self-declaration.** `README.md:19-21`: "The continuedev/continue repository is no longer actively maintained and is read-only for all users", plus a "Final 2.0.0 Release" section. Head 2026-07-20, ~2 months pre-run, and the README note plus read-only-for-all wording means the freeze is policy, not silence.
- **Cadence collapse measured**: 9,167 commits 2024-07..2025-07 -> 6,014 2025-07..2026-01 -> 849 2026-01..head -> 24 since 2026-06-01.
- **Historicals were strong**: 21,569 commits, **537 contributors** (largest real contributor base in the run), Apache-2.0 file + manifest, institutional backing (continuedev org, remote exact match, release/marketplace plumbing). That history plus a healthy fork surface is why this is 3 and not 1-2 -- the artifact survives, the maintenance does not.
- Consequence for the corpus: the serve-plane hole (s3/e8), the GUI edit bypass (s4) and the false headless-permissions doc (e9) will not receive security patches on this line. Rule (b) B-cap is checked-and-true but cannot bind at a 59.75 C total; recorded per the never-silent clause.

## Strongest / weakest (report requirement)

- **Strongest: verification (7/10, 70% of the heaviest weight).** Evidence: recounted corpus of 421 test files / 105,973 LOC asserting behavior, not existence -- bypass-shaped classifier tests (`terminalCommandSecurity.test.ts:137` sudo-from-variable, `:880` backslash-split, `:978` nested substitution, `:1158` newline-injection); loop-level approval semantics asserted through the real GUI thunk flow (`streamResponse_toolCalls.test.ts:1882-1983`); CLI precedence algebra pinned (`permissionChecker.test.ts:571-700`); 860-LOC behavioral suite over the real spawn surface (`runTerminalCommand.vitest.ts:128-666`); CI runs on 3 OS x 4 Node (`cli-pr-checks.yml:60-61,:113`). Held at the 7-rung, not 8, by the frozen errata: zero fuzzing, zero evals (`eval/` holds only `.gitignore`), unit CI keyed on live provider secrets gated by `IGNORE_API_KEY_TESTS` (`pr-checks.yaml:78-88`), coverage scripts that no workflow runs, and no tests on the GUI policy thunk or serve HTTP surface. (Tied numerically with interop 7 and docs-dx 7; verification named for measured, behavior-asserting evidence and weight.)
- **Weakest engineering dimension: orchestration (5/10).** Evidence: launching a subagent replaces the shared `TOOL_PERMISSIONS` service with `[{tool:"*", permission:"allow"}]` (`extensions/cli/src/subagent/executor.ts:81-88`), monkey-patches `systemMessage` and disables `ChatHistoryService` for the child (`:101-112`); the child inherits the subagent tool itself (`tools/index.tsx:132`) so nesting is model-bounded only; no spend/depth budgets anywhere in the 213-LOC executor; queue is an in-memory FIFO (`messageQueue.ts:20-56`); sessions are whole-file non-atomic JSON (`history.ts:81,131,178`); crash handlers defer a non-zero exit while execution continues (`index.ts:120-164`); zero loop detectors (c3/s9). Durability 3 is the lower raw number but is a maintenance-status judgment, recorded as such rather than laundered into the design column.

## Dedup log

- `continue-c3` (core, `phantom-safety-control`) + `continue-s9` (safety, `loop-detection`) -> one record at id `continue-c3`, concept `phantom-safety-control`, alias `continue-s9`; core evidence base kept (full recursion chain), safety's crush contrast + poisoned-context impact unioned.
- `continue-s3` (safety, filed under pool id `permission-policy`) + `continue-e8` (econ, `undocumented-bind-all-control-plane`) -> one record at id `continue-s3`, concept realigned to econ's precise coinage, alias `continue-e8` recorded; hidden/undocumented evidence folded into the safety record.
- Kept distinct on purpose (same citation, different scoring dimension, no silent double-count): `continue-c7` (session durability -> operability) vs `continue-e6` (undurable orchestration state -> orchestration), both citing `core/util/history.ts:81,131,178`; `continue-c2` (entry-path duplication -> architecture) vs `continue-e1` (compaction contract split -> token-economy); `continue-c9` (split crash posture, operability) vs `continue-e6`'s crash-handler citation (queue durability, orchestration); `continue-s4` (GUI edit bypass) vs `continue-s8` (triple-stack divergence) -- instance vs structure.

## Calibration notes

1. Provenance: **upstream original** (scout section 2 + census `provenance_flag: original`); the sync-fork-must-not-outrank-upstream rule cannot fire; no demotion taken.
2. **Rule (b) archived/dead: TRUE, cap non-binding.** README self-declares read-only + Final 2.0.0; cadence 9,167 -> 6,014 -> 849 -> 24 (since June). B-cap cannot demote a computed C (59.75); recorded explicitly, never silent, and the archive is charged to durability 3 rather than hidden in prose.
3. Census artifact corrected study-wide: `test_loc 19,409` rejected; 105,973 LOC / 421 files recount used (matches the crush/cline/nanocoder undercount pattern).
4. Weighted-total formula: score/10 x weight, weights sum 100; 59.75 lands mid-band C. S-band conditions moot (>=88 required; also safety 6 / verification 7 clear the 5-floor regardless).
5. Anchor split resolved to **nanocoder** on vector distance and gap-shape congruence (see Anchor placement); per-dimension placements (nanocoder-5 arch, 6-rung safety cluster, cline shape) retained inside the lane tables.
6. Band sensitivity: reaching B (65) would need, e.g., architecture +1 and orchestration +1.5 -- both refused: the named docks (dual stack c1/c2, permission-negation subagents e5) are structural, and per-lane verdicts were all `pass` with no lane overridden on evidence.
7. Read-only throughout: the subject was never executed; integrator code inspection limited to static reads of the cited files at head `5522c6f44` plus `git log` cadence counts.
