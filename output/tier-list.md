# Calibrated tier list -- coding-agent field study (final, post-sweep)

## 1. Preamble

**Method.** 103 corpus subjects reviewed read-only against the ten-dimension rubric (weighted sum = 100; architecture and verification at 15, safety-enforcement, token-economy, orchestration, interop, operability, originality at 10, durability and docs-dx at 5). Six anchors were scored strictly against the rung definitions in Phase 1 and **frozen** (codex 88.5 S, pi 78.5 A, cline 76.5 B-at-ceiling, crush 69.0 B, nanocoder 60.5 C, codel 23.0 D); every subsequent reviewer first answered "which anchor is this closer to, and why." Review effort was tiered from the census (T0 triage / T1 standard / T2 deep / T3 three-reviewer graph). **Boundary protocol:** any subject within 2 points of a band floor got a fresh independent reviewer before the band was final; after one flip the boundary score is authoritative (no recursion). Bands: S >= 88 (with no safety-enforcement or verification below 5), A 78-87, B 65-77, C 45-64, D < 45; rubric total decides tier, subject to rule (a) (a sync-fork derivative must not outrank its upstream; a divergent fork may, on demonstrated merit) and rule (b) (archived/dead caps at B, never silent).

**Distribution sanity.** Post-sweep, 9 of 103 corpus rows score 80+ = **8.7%**, well under the 25% re-anchor line; the sanity rule passes (wave6/score-inventory.md is the authoritative distribution; the two mid-study "past the line" alarms were partial-corpus arithmetic, dispositioned in boundary-open-interpreter RESOLVED).

**verification-9, as defined for publication (quoted from wave6/verification9-sweep.md):**

> **verification 9** = the subject's snapshot contains a model-behavioral verification gate that (1) drives LIVE provider calls and grades behavior (or, on the parallel fuzz road, executes oracled fuzzing of the shipped code paths in CI), (2) runs AUTOMATICALLY on a CI-required or release path and CANNOT be green when the behavior regresses -- a tolerant-steps-plus-hard-verdict job counts (prime-agent, openhands), but continue-on-error, report-only scripts that exit 0 on regression (gemini-cli), and comment-/dispatch-only manual triggers (deepagents, gptme, workground2, reasonix) are telemetry, not verification; and (3) has pass-threshold semantics auditable from the snapshot, with out-of-tree thresholds (kilocode's kilo-bench, openhands' required-check status) retained as explicit flags rather than silent passes or automatic demotions.

**Honest caveats.** ~94% of corpus rows rest on reviewer scores of **read-only snapshots**: no subject was ever executed, so every "binds" claim is code-and-CI evidence, not runtime observation. **Shallow clones truncate history** -- contributors=1 rows and HEAD-date inactivity claims were remote-verified before use (and several false inactivity calls were avoided: ra-aid, gemini-cli, claw-code-agent); where remote checks were impossible, activity judgments are marked evidence-limited. Private-harness thresholds (kilocode's kilo-bench, openhands' required-check status) are **retained as flags, not silent passes**. Census-derived LOC/contributor fields were corrected throughout (see errata classes in section 4).

## 2. The tier list

Rows are the final adjudicated totals: score files, then boundary RESOLVED entries (boundary authoritative), then the verification-9 sweep demotions (gemini-cli 84.5->83.0, deepseek-reasonix 86.0->84.5 -- score files intentionally unedited, applied here), then rule (b) caps/reversals as recorded. Provenance shown only where non-original. &dagger; = see aider footnote.

### S (>= 88)
| rank | subject | final total | band | tier | closest anchor | why here | provenance |
|---|---|---|---|---|---|---|---|
| 1 | codex | 88.5 | S | T2 | ANCHOR | kernel enforcement on all platforms, execpolicy DSL, 440k-LOC suite; the S anchor itself [ANCHOR] | original |

### A (78-87)
| rank | subject | final total | band | tier | closest anchor | why here | provenance |
|---|---|---|---|---|---|---|---|
| 2 | deepseek-reasonix | 84.5 | A | T3 | codex | top A after the verification-9 sweep: live e2e-bot is comment-triggered only, 86.0->84.5 | original |
| 3 | codex-infinity | 83.5 | A | T0 | codex | codex tree + patch layer (auto-next/goals/arc-monitor); sync-fork ceiling respected at 83.5 < 88.5 | sync-fork:codex |
| 4 | gemini-cli | 83.0 | A | T3 | codex | Google backing, largest property-asserting suite (387k LOC); verification 8 post-sweep (gate exits 0 on CONFIRMED regression) | original |
| 4 | hermes-agent | 83.0 | A | T3 | codex | first non-anchor 80+: below-yolo floors, fail-closed waits, live-provider cache-hit canaries | original |
| 4 | open-interpreter | 83.0 | A | T0 | codex | honest attribution-preserving codex rebrand; corpus-unique tested rival-harness wire-emulation plane; demote-then-reverse chain lands at 83.0 A | divergent-fork:codex |
| 7 | grok-build | 82.75 | A | T3 | codex | two-pass prefire compaction, parent-prefixed side calls, leader daemon; closed governance caps durability at 7 | original |
| 8 | oh-my-pi | 82.0 | A | T3 | codex | divergent pi fork +3.5 earned entirely in subsystems pi deliberately lacks; 12,181-LOC AgentSession pins architecture 7 | divergent-fork:pi |
| 9 | qwen-code | 80.0 | A | T3 | codex | divergent gemini-cli fork that drifted down on architecture; measured cache discipline, 133 releases | divergent-fork:gemini-cli |
| 10 | jcode | 79.25 | A | T3 | codex | late T3 entrant, codex-profile; weakest lane safety-enforcement | original |
| 11 | orca-agent | 78.75 | A | T3 | codex | exact-match boundary; both 9s (lease-pool budgets, fault-injected journals) survived first-hand attack | original |
| 12 | kilocode | 78.5 | A | T3 | cline | the one upward flip: live release-blocking Harbor evals keep verification 9; safety +1 under the opt-in three-gate amendment | divergent-fork:opencode |
| 12 | pi | 78.5 | A | T2 | ANCHOR | the A anchor; study-fragilest rung (arch 9 over a 6,852-LOC interactive-mode file) held twice independently [ANCHOR] | original |
| 14 | workground2 | 78.25 | A | T3 | codex | exact-match boundary; fenced saves + CAS leases; cache-impact CI gate origin corrected to wg2 (not inherited) | divergent-fork:deepseek-reasonix |
| 15 | codewhale | 78.0 | A | T3 | codex | exact-match boundary on the floor; kernel sandbox with live macOS denial tests; provenance corrected to rename-clone of deepseek-tui | rename-clone:deepseek-tui |

### B (65-77)
| rank | subject | final total | band | tier | closest anchor | why here | provenance |
|---|---|---|---|---|---|---|---|
| 16 | agentty | 77.5 | B | T2 | pi | boundary 77.5 B FINAL: oracled sanitizer fuzz in a blocking gate keeps verification 9; durability/orchestration bind the plateau | original |
| 16 | opensquilla | 77.5 | B | T3 | codex | exact-match boundary; token-economy 9 over the deepest tested kernel-jail stack in the corpus, shipped switched off (safety 5.5) | original |
| 16 | zeroclaw | 77.5 | B | T3 | codex | boundary demotion A->B: merge-gating eval replays canned traces through a stubbed provider (verification 8.5->8) | original |
| 19 | kimi-code | 77.0 | B | T2 | cline | boundary 77.0: stale v1/v2 dock retracted at HEAD; corpus-best grammar-parsed bash policy (differential oracle) | original |
| 19 | kolkrabbi | 77.0 | B | T2 | pi | B-ceiling plateau member; boundary "77.0 B CONFIRMED" (report verdict, no dispatcher ledger entry - see discrepancies) | original |
| 19 | opencode | 77.0 | B | T3 | cline | exact-match boundary: band ceiling not near-miss - no contested lane survives a 9 | original |
| 22 | cline | 76.5 | B | T2 | ANCHOR | B anchor, band ceiling: tested approval, zero OS-sandbox hits, auto-approve cron default [ANCHOR] | original |
| 22 | deepagents | 76.5 | B | T2 | pi | library-core convention row: architecture capped 8 at the dependency boundary; app.py 31,203 LOC (god-file errata symmetry) | original |
| 24 | gptme | 76.0 | B | T2 | pi | exact-match boundary; origin of the continue-on-error-equals-telemetry ruling | original |
| 24 | vtcode | 76.0 | B | T3 | codex | late T3 entrant at the B ceiling; token-economy strong, durability weak | original |
| 26 | memcode | 75.5 | B | T2 | cline | property-asserting test culture; clone-and-run hook hole (repo-shipped hooks.json executes at session start) | original |
| 26 | ouroboros | 75.5 | B | T2 | cline | verification-9 ledger #3: nightly live-E2E with a $30 cost cap - first budget-guarded eval gate | original |
| 28 | letta-code | 75.25 | B | T3 | cline | live-model scenario evals gate PR CI (8.5, held under 9 by zero fuzzing); shipped default auto-allows all tools | original |
| 29 | forge-norvialabs | 75.0 | B | T2 | codex | second non-giant with enforced-by-default OS sandbox; interop is the gap | original |
| 30 | jazz | 74.75 | B | T2 | pi | measured 4-rung compaction ladder with cache breakpoints; boundary recount adopted (76.5 -> 74.75, ledger closure) | original |
| 31 | bitfun | 74.5 | B | T3 | cline | multi-harness orchestrator (codex/claude-code/opencode/pi as externals); solo 8-month repo against 1.9M claimed LOC | original |
| 31 | maki | 74.5 | B | T2 | pi | grammar-parsed bash policy; hook-chain-timeout-fails-open variant (latency becomes permission) | original |
| 31 | san | 74.5 | B | T2 | pi | first clean cache-monotonic-compaction occupant; fail-CLOSED LLM permission judge (polarity exemplar) | original |
| 31 | smelt | 74.5 | B | T2 | pi | strongest verification-9 in the corpus: 17 oracled fuzz targets, committed seeds replayed in blocking CI | original |
| 31 | goose | 74.5 | B | T3 | cline | stated-total sum error (73) adjudicated FINAL 74.5 from its own lanes (calibration-notes ledger closure) | original |
| 36 | atomic-agent | 74.25 | B | T2 | pi | one separable turn loop behind thin hosts; durability the anchor-distant lane | original |
| 36 | octomind | 74.25 | B | T2 | pi | boundary demotion A->B: every enforcement layer ships disabled (sandbox=false, no built-in approval prompt in the tool path) | original |
| 38 | grinta-coding-agent | 74.0 | B | T2 | cline | verification-9 fell at boundary (trigger pipeline, no consuming needs-edge): 78.5 A -> 74.0 B | original |
| 39 | dexto | 73.5 | B | T3 | cline | validated serializable turn-state core; auto-approve schema default contradicts its own docs | original |
| 39 | mistral-vibe | 73.5 | B | T3 | cline | full ACP server + corpus-unique rival-plugin import layer; fail-closed tree-sitter shell gating | original |
| 39 | nausicaa-harness | 73.5 | B | T2 | cline | fencing-token leases + tested recovery journals (session-turn-lease x2 with hermes-agent) | original |
| 42 | kimi-cli | 73.0 | B | T2 | cline | first archived subject in the upper half; rule (b) cap at B recorded, non-binding; rewrite lineage to kimi-code | rewrite-lineage:kimi-code |
| 43 | kolega-code | 72.75 | B | T2 | cline | compound-command bypass of its own existing enforcement; boundary recount adopted (76.5 -> 72.75, ledger closure) | original |
| 44 | minicode | 72.5 | B | T1 | crush | jail + bash guards bind even under allow-all config - the exact inverse of nanocoder (floor invariant) | original |
| 44 | waveloom | 72.5 | B | T2 | crush | monotonic fold-decision set + measured injection economics (26% of cache-miss tokens); clone-and-run hooks x2 | original |
| 46 | crab-code | 72.0 | B | T2 | crush | claude-code-family candidate #5 at confidence MED, audit pending - excluded from the headline lineage count | claude-code-family (unconfirmed) |
| 46 | openhands | 72.0 | B | T2 | cline | agent-canvas shell covariate (core out-of-tree); verification 9 stands at the ledger bottom, fork-coverage flag retained | original |
| 46 | prime-agent | 72.0 | B | T2 | pi | divergent pi fork below upstream; label-gated fail-closed 28-task release gate = a clean verification 9 | divergent-fork:pi |
| 46 | tura | 72.0 | B | T2 | crush | enforcement-less-UI variant: TUI renders /approve with zero enforcement emitters in the engine | original |
| 50 | ipsupport-code | 71.75 | B | T1 | crush | shadow-mode risk classifier with regression-gated retrain in CI; pitfalls-into-tool-errors advice | original |
| 51 | forge | 71.25 | B | T2 | crush | real tested policy engine disabled by undeclared default + first-run allow-all (hidden-unsafe aggravated shape) | original |
| 52 | dvalincode | 71.0 | B | T1 | crush | narrowing-monotone policy with un-raisable hard blocks - an invariant making a whole attack class unrepresentable | original |
| 53 | kode-cli | 70.5 | B | T2 | cline | claude-code descendant via the anon-kode route (Anthropic-internal env branches); license-risk, concepts-only | claude-code-descended |
| 53 | mocode | 70.5 | B | T1 | crush | strongest non-kernel enforcement observed: fingerprint grants, fail-closed off-TTY, hash-gated skill trust, all tested | original |
| 55 | openharness | 69.5 | B | T1 | crush | broad modern feature surface; durability the band-limiting lane | original |
| 55 | roo-code | 69.5 | B | T3 | cline | divergent cline fork (roo-cline); archived cap at B recorded, durability 4 holds the total at low B | divergent-fork:cline |
| 57 | aeon | 69.0 | B | T1 | crush | defensive-ops culture, rival import tooling; interop credit checked against safety floor | original |
| 57 | crush | 69.0 | B | T2 | ANCHOR | B anchor: loop-detection breaker, client/server + sqlc state; opencode ancestry logged as lineage nuance [ANCHOR] | opencode-lineage (divergent) |
| 59 | code | 68.5 | B | T3 | crush | self-declared codex fork mid rename-migration; chatwidget.rs 44,921 LOC corpus-worst file, arch 5 on weight 15 | divergent-fork:codex |
| - | **hotdog** | 68.5 | B | T1 | pi | self-row: originality 8 (six code-verified corpus-unmatched mechanisms), safety 5 via the nanocoder precedent (best-tested gate shipped disabled); scored last, anchors-seen, self-profile as input not truth | original | **SELF** |
| 60 | 3code | 68.0 | B | T1 | crush | external fence engine (sandwall) + default-open network; boundary recount adopted (65.75 -> 68.0, ledger closure) | original |
| 61 | ob-1 | 67.5 | B | T1 | crush | first observable two-hop design chain (claude-code -> claw-code -> ob-1 headers); autopilot one /trust flip from off | design-lineage:claw-code |
| 62 | codebuff | 66.0 | B | T2 | cline | public-mirror-of-private-development covariate class founder; 151.8k test LOC, public CI build+smoke only | original |
| 62 | zot | 66.0 | B | T1 | crush | substring deny-list bypassable by construction; zotfile repo-declared capability manifest | original |
| 62 | hax | 66.0 | B | T1 | crush | boundary 66.0, B floor held; the honesty-credit ruling exhibit (one rung, upward only out of the misleading band) | original |

**hotdog SELF footnote:** hotdog's 68.5 B is a Wave-5 self-review by reviewers who had seen the frozen anchors, with its self-profile as input, not as truth (protocol rule 7). It is listed inline for context but **excluded from rank numbering and from all band counts** -- self-review, anchors-seen, not corpus-ranked. Its safety-enforcement 5 comes from the nanocoder precedent (best-tested gate in the corpus, shipped disabled by default); originality 8 on six code-verified corpus-unmatched mechanisms.

**aider &dagger; footnote (name-value vs rubric-value):** the famous pioneer of the field -- 199 contributors -- lands in mid-C because the rubric measures **2026 artifact quality, not historical influence**, which the study tracks separately in lineage notes (calibration-notes, aider T3 entry): 473 tests after the docs-as-tests census correction, zero MCP/ACP, safety 5, signature ideas (repo-map PageRank) now absorbed into corpus-wide band defaults. Publish aider's vector as the cautionary exhibit: rank follows the artifact.

### C (45-64)
| rank | subject | final total | band | tier | closest anchor | why here | provenance |
|---|---|---|---|---|---|---|---|
| 65 | keen-code | 63.75 | C | T1 | crush |  | original |
| 66 | ferrum | 62.75 | C | T1 | crush |  | original |
| 67 | molt | 62.0 | C | T1 | crush |  | original |
| 67 | neovate-code | 62.0 | C | T1 | cline |  | original |
| 69 | aider &dagger; | 60.5 | C | T3 | pi |  | original |
| 69 | nanocoder | 60.5 | C | T2 | ANCHOR |  | original |
| 71 | mimo-code | 60.0 | C | T0 | crush |  | divergent-fork:opencode |
| 71 | qqcode | 60.0 | C | T1 | nanocoder |  | divergent-fork:mistral-vibe |
| 73 | continue | 59.75 | C | T3 | nanocoder |  | original |
| 74 | amazon-q-developer-cli | 59.5 | C | T2 | crush |  | original |
| 75 | SWE-agent | 59.0 | C | T1 | crush |  | original |
| 76 | grok-cli | 58.5 | C | T1 | nanocoder |  | original |
| 77 | tau | 56.5 | C | T1 | pi |  | design-port:pi (declared) |
| 78 | zap-coding-agent | 56.0 | C | T1 | nanocoder |  | original |
| 79 | g3 | 55.0 | C | T1 | nanocoder |  | original |
| 79 | plandex | 55.0 | C | T1 | crush |  | original |
| 81 | claurst | 54.0 | C | T2 | crush |  | ported-proprietary-source |
| 82 | claw-code | 53.5 | C | T2 | nanocoder |  | port-derived:claude-code (self-declared) |
| 83 | openlumara | 51.0 | C | T1 | crush |  | original |
| 84 | claw-code-agent | 50.0 | C | T0 | crush |  | port-from-leaked-source:claude-code |
| 85 | ra-aid | 49.5 | C | T1 | nanocoder |  | original |
| 86 | mini-kode | 45.25 | C | T1 | nanocoder | boundary recount adopted (46.5 -> 45.25, ledger closure); 1.5 over the D cut | original |

### D (< 45)
| rank | subject | final total | band | tier | closest anchor | why here | provenance |
|---|---|---|---|---|---|---|---|
| 87 | codemachine-cli | 42.5 | D | T1 | codel |  | original |
| 88 | agentless | 42.0 | D | T1 | codel |  | original |
| 88 | coro-code | 42.0 | D | T1 | codel |  | original |
| 90 | auto-code-rover | 41.5 | D | T2 | crush |  | original |
| 91 | binharic-cli | 40.0 | D | T1 | nanocoder |  | original |
| 92 | free-code | 38.5 | D | T0 | codex |  | leaked-snapshot:claude-code |
| 93 | groq-code-cli | 36.5 | D | T1 | nanocoder |  | original |
| 94 | open-codex | 36.0 | D | T1 | nanocoder |  | divergent-fork:codex |
| 94 | trae-agent | 36.0 | D | T1 | crush |  | original |
| 96 | picocode | 34.0 | D | T0 | pi |  | original |
| 97 | cursor-agent | 33.5 | D | T1 | codel |  | original |
| 97 | devon | 33.5 | D | T1 | codel |  | original |
| 99 | darce-cli | 32.5 | D | T1 | codel |  | original |
| 100 | claude-engineer | 25.5 | D | T0 | codel |  | original |
| 101 | claii | 24.0 | D | T0 | codel |  | original |
| 102 | codel | 23.0 | D | T0 | ANCHOR |  | original |
| 103 | developer | 21.0 | D | T0 | codel |  | original |

## 3. Calibration rules applied

### Rule (a): fork-vs-upstream ordering (every check, with outcome)

- **kilocode > opencode: DIVERGENCE DEMONSTRATED, permitted.** Divergent fork outranks its upstream 78.5 vs 77.0 only after the integrator showed the delta explicitly: default bash-ask policy absent upstream, kilo-sandbox jail, prompt-queue/fork/board, fork governance; no upstream-synced code credited above opencode's own dimension scores; no sync markers (boundary-kilocode RESOLVED).
- **codex-infinity 83.5 < codex 88.5: sync-fork respected.** Daily-upstream-sync confirmed (byte-identical LICENSE/NOTICE); durability docked to 2; never re-promoted to T3 -- the delta is a patch series, not an architecture (Phase 2 differential triage).
- **workground2 78.25 < deepseek-reasonix 84.5: divergent, check moot.** Self-declared divergent fork (README.md:155, 84 hash-identical files); ordering consistent anyway (boundary-workground2 RESOLVED).
- **open-interpreter 83.0 < codex 88.5: satisfied.** Reclassified rename-clone -> honest divergent fork (no sync automation, pinned rust-v0.154.0 baseline, ~30k own Rust LOC); rule inapplicable and numerically satisfied regardless (boundary-open-interpreter RESOLVED).
- **oh-my-pi 82.0 > pi 78.5: divergent, permitted, gray edge self-recorded.** All excess credit sits in subsystems pi deliberately lacks; shared-ancestry dimensions match or trail pi; the note records that a sync-fork reclassification would cap it at 78.4 and names findings e1-e9 as the affected credit (oh-my-pi T3 entry).
- **qwen-code 80.0 < gemini-cli 84.5: consistent** post-sweep ordering for a divergent fork that also drifted down on architecture 6 (gemini-cli T3 integration).
- **prime-agent 72.0 < pi 78.5: moot** (below upstream; divergence understated at census -- 713 prime-only files).
- **roo-code 69.5 < cline 76.5: divergent fork, outrank was permissible, not exercised** (score-file calibration notes).
- **mimo-code 60.0 < opencode 77.0: clean** after rename-clone -> divergent-fork correction; "provisional C" may rise only on its own merits (rule (a) check, no action).
- **crush 69.0 < opencode 77.0: divergent ancestry, no cap needed**; lineage kept as nuance (opencode result + lineage checks).
- **qqcode 60.0 < mistral-vibe 73.5: consistent** -- divergent-fork:mistral-vibe confirmed by file-level delta (21/145 shared byte-identical, 70 own files).
- **code 68.5 << codex 88.5: moot** (self-declared divergent fork).
- **kimi-cli <-> kimi-code: rule (a) inapplicable** -- predecessor/successor REWRITE, not code descent; convention set: rewrite-lineage pairs rank separately with a lineage note (kimi-cli T2 entry).
- **codewhale rename-clone:deepseek-tui: rule (a) NOT applicable in practice** -- upstream not in the corpus, nothing to cap against; 78.0 stands, but the "original" label must be discounted by convergence/originality readers (boundary-codewhale RESOLVED).
- **claurst:** rule (a) uncheckable locally -- claude-code is not in the corpus; lineage reclassified fork -> ported-proprietary-source (evidence-limited, logged).

### Rule (b): archived/dead caps, demotions, reversals (never silent)

- **roo-code: archived cap at B, durability-driven.** GitHub archived=true; the raw 69.5 was already B so the cap did not bind numerically, but it drove durability to 4 (an active org-backed profile would earn 7-8), which is what holds the total at the low end of B (score-file calibration notes).
- **kimi-cli: first archived subject in the upper half; rule (b) cap at B formally applied, non-binding** (without the cap still B; recorded for the footnote -- highest legitimately-B-due-to-cap subject).
- **auto-code-rover: dead cap at B applies regardless of direction**; boundary recount 41.5 D sits under it anyway (2026-09-29 entry + Boundary band-flips ACCEPTED).
- **codel: dead >12mo** (remote pushed_at 2024-04-29) -- T0 + rule (b) noted at anchor phase (23.0 D, cap moot).
- **agentless: dead remote-verified, rule (b) applied, moot** (42.0 D; T0 criterion retro-met).
- **claude-engineer (dead 21mo) and developer (dead 29mo): dead, already D, no cap needed** -- activity status recorded in each score file's calibration_notes so the D is understood as dead-D, not merely low.
- **picocode and claii: below the dead bar** -- remote checks show 8mo and ~10mo quiet respectively; rule (b) explicitly NOT applied (no death claim; evidence-limited beyond that).
- **plandex: 361 days stale = dead-in-practice by a four-day timing accident**; convention adopted: rule (b) is evaluated as "would a reviewer in a normal window call this dead" -- footnote, not cap (C < B anyway).
- **open-interpreter: the demotion-then-reversal chain, the study's never-silent exemplar.** Phase-2 POLICY DEMOTION to D (rename-clone rule) -> dispatcher REVERSAL to 78.5 A (bands measure code quality; trademark risk lives in findings) -> full boundary review lands 83.0 A; the reversal stands, both directions logged in calibration-notes.

## 4. Drift notes (how calibration moved across the study)

- **Boundary program:** 21 boundary pairs adjudicated and the ledger closed (boundary-orca-agent RESOLVED); the boundary score is authoritative after one flip (claw-code-agent precedent: the mean was rejected because the boundary rule does not recurse).
- **Seven exact decimal matches:** cline, pi, opencode, workground2, codewhale, opensquilla, orca-agent reproduced all ten rungs independently -- independent scoring agreeing to the dime is the study's strongest reliability signal (a gptme/workground2 numbering collision and kolkrabbi's identical-but-unledgered recount are bookkeeping errata, see UNLOGGED-DISCREPANCIES).
- **Seven band flips: six down** (amazon-q-developer-cli 65.0->59.5 C, octomind 79.5->74.25 B, claurst 65.0->54.0 C, grinta 78.5->74.0 B, auto-code-rover 45.5->41.5 D, zeroclaw 78.25->77.5 B) **and one up** (kilocode 77.5->78.5 A), which survived a hostile verification-9 re-audit (Boundary band-flips ACCEPTED + kilocode verification-9 RE-AUDIT RESOLVED); ferrum is the mirror-image precedent: both contested lanes GRANTED at boundary and the band still held.
- **Three verification-9 demotions total:** grinta (boundary: trigger pipeline with no consuming needs-edge), gemini-cli (exit-0 even on a CONFIRMED regression), deepseek-reasonix (comment-triggered-only e2e bot, fuzz targets never executed); the last two are band-safe 1.5-point adjudications applied in this table, and the sweep retired "release-gated not PR-gated" as a withhold criterion (wave6/verification9-sweep.md).
- **Census-errata classes found en route:** test-glob blindness x6+ (Rust inline `#[cfg(test)]`: zeroclaw 34,710 claimed vs 543,372 real; Go colocated x5; TS x5), inline-cfg(test) blindness, docs-as-tests (aider's "76k test LOC" was the website doc tree), blob inflation (auto-code-rover: 927MB of checked-in results read as 2.6M-LOC "diff"), mirror-in-tree double-count (code: 618k claimed, 311k real once the never-compiled upstream mirror was scoped out), test-fixture-loc inflation (deepagents' 205k tau2 db.json), generated-blob overcount the other direction (devon ~17x), and LICENSE parse blindness (kolega's BUSL "Change-License" line misread as AGPL) -- glob census errors are bidirectional, which is worse for naive tiering than any consistent bias (multiple Phase-3 entries).
- **Renamed-vs-original provenance corrections:** codewhale reclassified from "original" to rename-clone of deepseek-tui (upstream outside corpus; discount its "corpus-unique" claims); workground2's lineage resolved as declared-divergent-fork of deepseek-reasonix AND its cache-impact CI gate re-attributed as origin rather than inheritance (the T3 inheritance-audit table was wrong; workground2-b2); mimo-code rename-clone -> divergent-fork; claurst fork label -> ported-proprietary-source.
- **Two study-wide doctrine outputs:** (1) the safety default-posture rule with the kilocode opt-in three-gate amendment -- rung = rung earned by the binding DEFAULT posture, plus at most one rung for an opt-in layer passing fail-closed / blocking-CI-tests / non-deceptive gates; and its sibling honesty-credit ruling (hax/kolkrabbi): honesty mitigates deception penalties but never substitutes for enforcement (Boundary-kilocode + hax FINAL entries); (2) the publication definition of verification-9 quoted in the preamble (wave6/verification9-sweep.md).

## 5. Method footnotes

- **Import-graph screening rule (binharic-cli entry):** compute the share of test files that import production code; <50% = replica-test suspicion. Cheap, structural, and catches what CI greenness cannot (binharic's 37/88 test files tested a parallel imaginary product).
- **Independence-disclosure norm (keen-code, claurst disclosures):** boundary reviewers must scope concept-dedup greps AFTER independent scoring (or by concept-id only) -- whole-corpus greps before scoring can surface original-review finding titles; both disclosed reviewers fenced and continued cleanly.
- **Runner-blind test families:** screening rule = test-file count vs runner-report count mismatch. Three shapes found: runner-blind (groq-code-cli: ava glob orphans 3/4 of its own tests from `npm test`), CI-excluded (kimi-code: 33 WAL test files in a vitest exclusion, zero CI jobs), and Makefile-skipped (trae-agent: 33% of the corpus hard-skipped by rotted patch targets).

## UNLOGGED-DISCREPANCIES

All discrepancies were logged and adjudicated in calibration-notes ("Ledger closure: unledgered boundary recounts ACCEPTED", 2026-10-01); none changes a band.

1. **Seven boundary re-reviews lacked dispatcher RESOLVED entries when this list was drafted** (3code 68.0 B, agentty 77.5 B, codebuff 66.0 B, jazz 74.75 B, kolega-code 72.75 B, mini-kode 45.25 C, zot 66.0 B). ALL ACCEPTED en bloc in the ledger-closure entry, boundary authoritative per precedent; the boundary rows above are the adopted finals.
2. **agentty adopted at 77.5 B** from the boundary report -- now ledgered in the same closure entry (the missing ledger line this draft flagged exists).
3. **goose arithmetic RESOLVED:** stated weighted_total 73 was a sum error; dispatcher adjudicated FINAL 74.5 B from its own lanes (calibration-notes ledger-closure entry, 2026-10-01). Full lane recompute across all 135 score files (main+boundary) found this as the ONLY arithmetic drift.
4. **kolkrabbi:** exact-match boundary confirmation -- accepted and ledgered in the closure entry; totals identical, table unaffected.
5. **Exact-match count bookkeeping:** standardized in the ledger to 9 delta-zero boundary totals (cline, pi, opencode, workground2, codewhale, opensquilla, orca-agent, gptme, kolkrabbi); per-entry ordinals in earlier notes kept as written history.

## Sanity check

- Final corpus rows: 103 = S 1 + A 14 + B 49 + C 22 + D 17 (hotdog self-row excluded from all counts).
- 80+ rows: 9 (codex 88.5, reasonix 84.5, codex-infinity 83.5, gemini-cli 83.0, hermes-agent 83.0, open-interpreter 83.0, grok-build 82.75, oh-my-pi 82.0, qwen-code 80.0) = 8.7% -- sanity rule passes.
- Lane-level recompute of every final row: consistent (goose sum-error drift adjudicated to 74.5 per the ledger-closure entry; rows above reflect finals).
