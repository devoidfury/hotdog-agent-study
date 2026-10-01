# Final coherence audit (2026-10-01)

Read-only verification pass over the completed study. One fix slot allowed (tier-list total-line
arithmetic only); it was NOT needed and nothing was edited.

## 1. Deliverable completeness checklist

| # | Requirement | Result | Evidence |
|---|---|---|---|
| 1 | subjects/<name>.md for all 103 census subjects | PASS | 104 files = 103 census + hotdog; census↔files set-diff = 0 both ways |
| 2 | scores/<name>.json for all 103 | PASS | 104 files = 103 census + hotdog self-row; set-diff = 0 |
| 3 | census cross-check, expect exactly 0 missing | PASS | census.json `subjects`: 103 entries; no missing/extras |
| 4 | findings.jsonl exists and parses | PASS | 1,666 lines, 1,666 parsed, 0 bad; 104 distinct subjects; portable+safety-hole pool = 754 (matches roadmap's "754") |
| 5 | tier-list.md / convergence.md / roadmap-for-hotdog.md / census.json exist | PASS | all present |
| 6 | calibration-notes.md with adjudication ledger | PASS | 1,059 lines; boundary RESOLVED entries, ledger-closure section ("21 boundary pairs adjudicated"), goose adjudication, verification-9 SWEEP RESOLVED |
| 7 | Phase 0 census fields: git / license / manifest-name / provenance | PASS | per-row `commits`, `head_date`, `shallow`, `upstream_remote`; `license`; `manifest_name`+`manifest_file` (rename-clone detection); `provenance_flag`+`provenance_note`+`rename_clone_suspect` |
| 8 | Anchor phase: anchors/ = 6 reports + 6 scores + excerpts | PASS | cline, codel, codex, crush, nanocoder, pi (.md) + scores-*.json x6 + anchor-excerpts.md (incl. ERRATA) |
| 9 | T3 graph shape used: merged-findings.jsonl count 24 in t3 run dirs | PASS | exactly 24 `merged-findings.jsonl` under /etc/hotdog/workflows/runs/*t3-giant-review*; 774 raw recs = merge-log's "24 runs / 774 recs" |
| 10 | Boundary rule: every BOUNDARY flag in wave6/score-inventory.md has a boundary pass or logged exception | PASS | 15 flagged; 14 have boundary/<name>.md passes (3code, amazon-q-developer-cli, auto-code-rover, claurst, codewhale, grinta-coding-agent, hax, kilocode, open-interpreter, opensquilla, orca-agent, pi, workground2, zeroclaw). codex: anchor exception -- frozen S anchor, src=A in the inventory, anchors-vs-main "no reconciliation needed" (note: exception is implicit in anchor status, not an explicit one-line waiver in calibration-notes). agentty: not flagged (76.5/77.5 ok); its boundary recount is the ledger-closure "already boundary-reviewed -> band final" accepted entry, consistent. |

## 2. Citation resolution

Extraction: regex over roadmap-for-hotdog.md, tier-list.md, convergence.md; resolved against the
1,666 finding ids. 1,134 distinct citations resolve to findings.jsonl. All hotdog ids cited
(1,2,3,4,6,7,8,9,10,13,14,15) present.

UNRESOLVED literal ids (9, all in convergence.md; none in roadmap or tier-list):

| literal | intended | intended id exists? |
|---|---|---|
| amazon-q-12 | amazon-q-developer-cli-12 (quality-gates-unwired) | yes |
| amazon-q-6 | amazon-q-developer-cli-6 (checkpoint-revert) | yes |
| binharic-2 | binharic-cli-2 (phantom-safety-control) | yes |
| grinta-4 | grinta-coding-agent-4 (sandbox-delegation) | yes |
| grinta-9 | grinta-coding-agent-9 | yes |
| grinta-b2 | grinta-coding-agent-b2 | yes |
| grinta-b4 | grinta-coding-agent-b4 | yes |
| grinta-b5 | grinta-coding-agent-b5 | yes |
| s14-b4 (from "zeroclaw-s14/s14-b4") | zeroclaw-s14 + zeroclaw-b4 (both security-posture-docs) | both exist; the compound literal does not |

Editorial shorthand (subject-name truncations / slash re-cites), zero dangling references; 8/9 map
unambiguously, the 9th decomposes into two existing ids. Everything else the extractor flagged
(verification-9, exit-0, Phase-2, AGPL-3, e1-e9 range notation, goose-v1-25-0, ob-1 as a subject
name, etc.) is prose, not a citation.

### Random-10 evidence spot-check (seed 42) -- 10/10

| id | evidence | check |
|---|---|---|
| amazon-q-developer-cli-b5 | .github/workflows/terminal-bench.yaml:6-13 | file exists (84 lines); :6-13 shows `workflow_dispatch`-only trigger -- claim "never blocks" real |
| codewhale-e8 | crates/tui/src/tools/subagent/mod.rs:8-12 | exists, 20,578 LOC matches claim; :8-12 literally says retired tools "still registered and executable by name" |
| continue-e6 | extensions/cli/src/stream/messageQueue.ts:20-56 | exists; :20 `class MessageQueue extends EventEmitter` + plain array -- matches |
| crush-7 | internal/agent/channel.go:5-15 | exists (18 lines); WithChannel/ChannelFromContext context-tagging exactly as claimed |
| dexto-c12 | packages/core/src/session/session-manager.ts:318-394 | exists (1,389 lines); :318 `forkSession(parentSessionId...)` matches |
| groq-code-cli-2 | src/tools/tools.ts:123,185,222,264 | exists; the four cited lines are exactly the path.resolve calls + the startsWith delete guard |
| hotdog-14 | src/extensions/loop-detect/detector.ts (+index.ts, tests) | hotdog has no /data snapshot (it is the host harness); evidence verified against /workspace: detector.ts exists (109 lines), cited ranges plausible |
| kilocode-b4 | .github/workflows/check-opencode-annotations.yml:1-40 | exists (66 lines); PR-triggered annotation check matches "CI-blocking ... every push" |
| openharness-14 | src/commands/skills.ts:136-145 | exists; --accept-license=<SPDX> refusal copy exactly as claimed |
| zeroclaw-e14 | crates/zeroclaw-channels/src/orchestrator/mod.rs (52,741 LOC); acp.md:213 | mod.rs exists at exactly 52,741 LOC (claimed number exact); acp.md:213 is real ACP-surface prose |

## 3. Numerical coherence

- **Tier-list band census (rows recomputed):** 104 rows incl. SELF; corpus rows = S 1 / A 14 / B 49 /
  C 22 / D 17 = 103. Matches the file's own sanity line exactly. No total-line typo; **no fix applied**
  (fix slot unused). No duplicate names, hotdog correctly excluded from counts.
- **Sweep demotions / boundary rows visible:** gemini-cli 83.0 A ✓, deepseek-reasonix 84.5 A ✓,
  grinta-coding-agent 74.0 B ✓, zeroclaw 77.5 B ✓, open-interpreter 83.0 A ✓.
- **80+ tally:** 9/103 = 8.7% -- recount of the listed rows agrees.
- **Roadmap ranking formula, all 16 items recomputed against findings.jsonl concept counts:**
  printed conv x printed fit = printed score for all 16; AND printed conv == actual distinct-subject
  count for all 16 (faux-provider-testing 47, compaction-tiering 43, permission-policy 61,
  evals-in-ci 29, loop-detection 36, prompt-cache-marking 29, workflow-resume-journal 19,
  hook-trust-scoping 25, usage-measured-compact-trigger 23, cache-monotonic-compaction 19,
  grammar-parsed-bash-policy 16, security-posture-docs 24, phantom-safety-control 15,
  architecture-contract-test 11, headless-contract 7, turn-budget-accounting 5).
  **0 items disagree.**
- **Three-move arithmetic:** 68.5 + 1.0 (safety rung x w10) = 69.5; + 1.5 (verification rung x w15)
  = 71.0; + 1.0 (token-economy rung x w10) = 72.0. Consistent with scores/hotdog.json weights;
  final "6.0 under the A floor" = 78 - 72 ✓.
- **FLAGGED disagreement (not in the allowed-fix scope): goose.** calibration-notes
  "Ledger closure (2026-10-01)" adjudicates goose FINAL = 74.5 B and states "tier-list row carries
  74.5 with this entry as the warrant"; tier-list still prints 73 with "unresolved arithmetic flag"
  and its UNLOGGED-DISCREPANCIES #3 says "no adjudication ... stated 73 retained". Same band either
  way (no census impact), but the two documents contradict each other on both the value and on
  whether an adjudication exists. Stale-row drift; left untouched per audit rules.

## 4. Cross-document hotdog position

| document | total | band | dims asserted | agree? |
|---|---|---|---|---|
| scores/hotdog.json | 68.5 | B | arch 8, verif 7, safety 5, token 7, orch 8, interop 5, orig 8, oper 7, dur 4, docs 8 | baseline |
| tier-list.md SELF row | 68.5 | B | safety 5, originality 8 (six corpus-unmatched mechanisms) | ✓ |
| convergence.md | 68.5 | B | "safety 5 / interop 5 / verification 7" (line 2253); safety clause "rung 5 ... we scored ourselves 5" | ✓ |
| roadmap "Where we are" | 68.5 | B | safety 5, verif 7, interop 5, token 7, orch 8, oper 7, arch 8, docs 8, orig 8 (strongest), dur 4 (weakest) | ✓ |

No drift. All four name 68.5 / B and identical dimension values.

## 5. Fix applied

None. The tier-list total line agrees with its own recomputed rows, so the single allowed
arithmetic fix was not triggered. No other file touched.

STUDY STATUS: PUBLISHABLE
