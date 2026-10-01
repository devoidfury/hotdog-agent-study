# qwen-code — T3 integration (run 20260929-0502-t3-giant-review-2)

Integrator merge of lanes reviewer-core / reviewer-safety / reviewer-econ over `/data/samples/agents/qwen-code` @ 9bfc4b9; read-only throughout, subject never executed. Merged findings: `merged-findings.jsonl` (32 records: 6 unique, 12 portable, 7 nuance, 4 anti-pattern, 3 safety-hole).

**Closest anchor: codex (S 88.5).** All three lanes independently converge on codex proximity in their own bands (enforcement layers below the bypass switch, session journals, three-tier compaction, crash-reclaim journals, multi-language SDKs); architecture shape is the one exception where core reads it closer to cline. The system-level read: qwen-code is codex's feature furniture reinstalled on gemini-cli bones, defended by tests, minus codex's default-on all-platform kernel enforcement and architectural discipline.

## Score disagreements resolved

No lane scored a dimension twice, so no numeric reconciliation was required; four qualitative tensions were resolved explicitly:

1. **Architecture anchor proximity vs the 6.** Core called the shape "closer to cline" (arch 8) yet scored 6 — correct, not a contradiction: the proximity identifies the family (core-first loop, hosts on top), while the score docks below both cline-8 and crush-7 for a *second full tool engine* in the ACP host that mirrors CoreToolScheduler by comment only (Session.ts:12198,12212,13530 vs coreToolScheduler.ts:1597), a 2070-line `sendMessageStream` (client.ts:3055-5125), and while-loop turn policy resident in all four hosts (nonInteractiveCli.ts:2459, use-llm-stream.ts:4070, Session.ts:5030, opentui/live-session.ts:853). That is cline's dual-tree docking problem plus crush's fusion problem in one tree. 6 stands.
2. **Verification 8 despite a partially breached 8-vs-10 ceiling.** Safety built the eval rig no anchor except codex had wired at all (SWE-bench Verified 500 instances + Terminal-Bench 2.0, dsw-swe-verified-release.yml) — but it fires on release, not per-PR (:3-4), there is zero fuzzing across packages/.github/scripts despite three enforcement-critical parsers, and bwrap confinement is asserted at argv level under mocked fs, never executed in CI (bwrap-execution.test.ts:153-154 — nanocoder's CI actually installs bubblewrap). The anchor ceiling is *in-CI* evals + fuzzing; qwen has neither. 8 stands, at the ceiling.
3. **Interop 9 vs core's ACP host-divergence finding.** Econ scored contract-surface breadth (ACP + MCP + TS/Python/Java SDKs + versioned daemon REST + 9 channels — codex's 9 cell) and explicitly declined to re-penalize Session.ts host divergence; core already charged that in architecture (c1/s6 are the same seam from two sides). No double-count either way: interop 9 and architecture 6 coexist by construction.
4. **Operability 8 under a codex-9 proximity claim.** Journaling machinery is genuinely at codex-9 level (JSONL + ledger sidecar sessionService.ts:849-852,1030-1055; fsync file+dir :595-740; 1318-LOC corruption suite; writer lease absent from every anchor, session-writer-lease.ts:19-21,51-80) but docked for no debug-introspection analogue, a 4.9k-LOC SessionService god class, undocumented journal format vs pi's docs/sessions.md, and rewind's admitted shell-edit gap (fileHistoryService.ts:547-548). 8 stands.

## Integrator-owned dimensions (my scores, evidence)

### originality: 8/10

Requirement was to verify the idea is real in code, not marketing. Verified directly this pass:
- **session-writer-lease (unique in corpus, confirmed):** cross-process transcript lease fenced by `readLocalBootId`/`readPidNamespaceId` (session-writer-lease.ts:18-19), O_NOFOLLOW lock-dir with dev/ino revalidation, versioned LOCK_SCHEMA_VERSION 1/2/3 (:22-24), transcript snapshot-hash fencing. No anchor has any writer-lease concept — this is the single most original mechanism found and it ships with two test files.
- **Fail-closed LLM permission classifier with deterministic floors beneath it (confirmed):** classifier.ts:17-18 states the fail-closed contract and code honors it (:263/382); deterministic destructive-command guard runs *before* the classifier (autoMode.ts:817-846) and external writes can never be classifier-approved (:855-867). Distinctive in the corpus — nanocoder's jail fail-opens, codex's execpolicy is a DSL not a classifier; qwen's layered classifier-with-floors is the most developed LLM-in-the-loop enforcement design observed.
- **Cache-safe subagent forks (confirmed):** forkedAgent.ts:10-26 makes cache-safety an explicit fork parameter with a session-mismatch guard and a dedicated suite (forkedAgent.cache.test.ts) — no anchor peer.
- Not 9: much of the surface is elaboration of corpus-known concepts (compaction tiers → codex, loop-detection → crush's first implementation now an order of magnitude larger, workflow journals → cline's resume journal, durable cron → cline's sqlite cron), on top of inherited gemini-cli DNA. Original where it exists, not field-defining across the board. 8 = "defended: properties enforced AND tested", below codex/pi's 9.

### durability: 9/10

Census row was unusable (contributors=1, commits=1 — shallow-clone artifacts; scout measured LOC inversion independently). Remote check on `github.com/QwenLM/qwen-code` closes the scout's owed check:
- **Institutional backing:** QwenLM org (Alibaba's Qwen team) — the same class as codex/OpenAI and above crush/Charm. `fork:false`, Apache-2.0 (LICENSE, GitHub API `spdx_id: Apache-2.0`) with upstream Google attribution retained — clean legal footing, no license-risk finding.
- **Cadence:** created 2025-06-26, `pushed_at 2026-09-29T07:49Z` (same day as this review); 133 releases in CHANGELOG.md, latest v0.24.6 dated 2026-09-26 — better than weekly; PR #12532 at head; 60 CI workflows incl. daily CVE hard gate, monthly OpenSSF Scorecard, nightly CodeQL (s8).
- **Community scale:** 28.2k stars, 3.1k forks, 1.5k open issues — adoption comparable to or above every anchor except codex.
- Docked from a 10-grade only by repo youth (15 months) and unassessable bus factor inside the corporation (shallow clone; census `contributors:1` artifact). 9, matching codex/cline's rung: institutional backing + heavy CI/release infrastructure.

## Totals

| dimension | score | weight | weighted |
|---|---|---|---|
| architecture | 6 | 15 | 9.0 |
| verification | 8 | 15 | 12.0 |
| safety-enforcement | 7 | 10 | 7.0 |
| token-economy | 9 | 10 | 9.0 |
| orchestration | 9 | 10 | 9.0 |
| interop | 9 | 10 | 9.0 |
| operability | 8 | 10 | 8.0 |
| originality | 8 | 10 | 8.0 |
| durability | 9 | 5 | 4.5 |
| docs-dx | 9 | 5 | 4.5 |
| **weighted_total** | | 100 | **80.0** |

**Band: A (78-87).** S-gate numerics: safety-enforcement 7 and verification 8 both clear the >=5 floor, but the 88 bar is unreachable — architecture 6 and safety 7 are the load-bearing gaps: a jail the flagship hosts refuse (runtime-shell-policy.ts:74) and never run in CI, and a tool engine duplicated per host, are exactly what codex has and qwen lacks.

**Strongest dimension: token-economy 9** (tied numerically with orchestration/interop/docs-dx; token-economy chosen on evidence depth): three-tier compaction ladder with failure circuit breaker (chatCompressionService.ts:109-142) + 966-LOC no-LLM microcompaction evictor (microcompact.ts:14,41-61) + per-provider cache-breakpoint discipline with live-measured global-scope/1h-TTL gating (anthropicContentGenerator.ts:538-686) + cache-invalidated-after-compression (llm-chat.ts:2752) — the strongest cache story in the corpus, above codex's excerpt on measurement honesty (trigger gate measures the pending prompt, pi-style, :286-296).
**Weakest dimension: architecture 6** — evidence in disagreement-resolution 1 (dual tool engine, 2070-line method, four-host turn loops, 12.2k-LOC god-config).

## Calibration notes

- **Rule (a) not triggered:** provenance is *divergent-fork:gemini-cli v0.8.2* (README.md:214 "stopped syncing"; README:214, dependabot.yml:13, Google/Qwen license-header strata, geminiChat.ts:7 deprecated shim). Divergent forks may outrank; gemini-cli is unscored anyway. No demotion applied, recorded here so it is not silent.
- **Archived/dead rule not triggered:** remote `archived:false`, `pushed_at` today.
- **Census corrections carried:** LOC split inverted (measured ~1.58M prod / ~2.30M test vs census 3.77M/213k); contributors=1 is a shallow artifact — remote-verified this pass.
- **Distribution check:** after qwen-code, 4/17 scored subjects at 80+ (23.5%) — under the 25% loss-of-calibration line, but one more 80+ subject trips it; flag for dispatcher.
- **Kind normalization at merge:** econ's `capability` findings mapped to schema kinds (portable; e4 cache-safe-fork unique, e9 interop-matrix nuance per its positioning-only nature); safety's `gap` findings mapped to `nuance` per codex-10 precedent. Original kind preserved as `lane_kind`. Five econ concepts re-canonicalized to seeded corpus ids with the lane-coined ids recorded as aliases (compaction-tier-ladder→compaction-tiering, workflow-crash-journal→workflow-resume-journal, loop-detection-breaker→loop-detection, durable-cron→scheduled-agent-runs, cache-warming-absent→prompt-cache-warming). No cross-lane concept collisions, so no findings were dropped.
- **Known med-confidence residuals carried downstream:** two-TUI production share (c8), host approval-divergence not behaviorally demonstrated (s6), no-fuzzing absence claim (s9), SDK publish-CI unverified (e9/e12).
