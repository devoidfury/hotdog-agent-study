# codewhale — T3 integration (run 20260929-0855-t3-giant-review)

Integrator merge of lanes reviewer-core / reviewer-safety / reviewer-econ over `/data/samples/agents/codewhale` @ 94130d9 (v0.10.0/v0.10.1-era, 27 crates, ~1.25M Rust LOC); read-only throughout, subject never executed. Merged findings: `merged-findings.jsonl` (34 records from 35 lane findings, 1 cross-lane dedup: core c7 == safety s12 -> `crash-recovery-middleware` with aliases recorded).

**Closest anchor: codex (S 88.5).** All three lanes independently converge on codex proximity in their own bands: codex-shaped typed execpolicy with segment-aware denial and live kernel denial tests (safety), codex 9-rung orchestration rebuilt plus ACP and rival-import surfaces codex lacks (econ), codex-class faux-provider test corps and debug-request-preview surface (core). The system-level read: codewhale is what codex's mechanism set looks like rebuilt by a solo-core community at 1.25M LOC — the mechanisms are real, wired, and property-tested, but the loop is fused at ~7x crush's scale inside an 87%-of-tree crate, and kernel enforcement is automatic on one platform only.

## Score disagreements resolved

Lanes scored disjoint dimensions, so no numeric reconciliation was required; five qualitative tensions were resolved explicitly:

1. **Dual tree: econ docked it (e13), core credited its absence (c3).** I verified on disk: `context_budget`, `prompt_zones`, `session_tree`, `tool_history_repair` each exist at exactly one path (`crates/runtime/src`), and the alias block (`crates/tui/src/lib.rs:133-142`) is one `use` statement per the extraction plan. The "Copied verbatim from the TUI crate root" comment (`crates/runtime/src/lib.rs:12-13`) refers to the crate-level `#![allow(clippy::uninlined_format_args)]` attribute, not duplicated modules. Resolution: both records stand — architecture credit (c3, migration shape worth copying, machine-checked runtime->UI ban) and docs-dx dock (e13, the comment phrasing + alias indirection + LEGACY_PATHS are a genuine confusion surface for readers) are the same seam scored from opposite sides; no double-count either way. cline's 8-rung dock was for *actual* duplicate trees; codewhale's dock is for the paper trail, which is why docs-dx sits at 8, not lower.
2. **Verification 8 despite in-CI "eval" steps.** The scout flagged the `-- eval` legs as a possible verification-9 breaker; safety ruled them offline prompt-composition smoke that the repo itself labels a smoke test (`eval.rs:1-4`, `eval_smoke.feature:3`), and additionally gated on trusted/self-hosted legs (`ci.yml:906,942`). Under the refined rule from gptme (evals count toward 9 only when the job blocks AND observes model behavior), these legs qualify on neither prong; there is no fuzzing anywhere. 8 stands, at the ceiling — same rung as codex, cline, pi, qwen.
3. **Econ's open question e8 (are retired-but-executable coordination tools re-gated?) vs safety's s5.** Safety's `shell_denial_acceptance.rs:22-32` proves denied bash cannot be reached through indirect task search/start end-to-end — that answers the denial-path half of e8 for the shell case. The residual (all six retired verbs re-passing *approval*, not just denial, gates) stays an open safety-hole finding with med confidence; it is not scored down further because the tested indirect-reach property covers the dangerous composition.
4. **Scout-map corrections both lanes found independently (panic hook, module copies).** Both are load-bearing for scoring: the global panic hook exists (`crates/tui/src/lib.rs:1815`, verified again this pass) which is why operability is 8 not 7-adjacent, and there is no live dual tree which is why architecture is 6 not 5. The scout's §3.8 "no global panic hook" risk item is retracted in the merged record (m1 aliases c7+s12), not silently.
5. **Provenance correction vs scores: rename-clone does not change the score.** Census said `original`; scout proved an honest self-rebrand of DeepSeek-TUI (LICENSE retains "DeepSeek-TUI Contributors", `app-server/src/lib.rs:38-39` says so in-code, updater migrates pre-rebrand binaries). Per the open-interpreter reversal precedent (bands measure code quality; provenance/honesty lives in findings), no policy demotion: recorded here, never silent. The self-rebrand is attribution-preserving, the opposite of the deceptive-rebrand pattern the census detector was fixed for.

## Integrator-owned dimensions (my scores, evidence)

### originality: 8/10

Task: verify the ideas are real in code, not marketing. Verified directly this pass:
- **Survival contract (real, corpus-unique, one nuance):** `SURVIVAL_CONTRACT.md` + versioned schema (`survival_contract.rs`, SURVIVAL_CONTRACT_VERSION=2, rule enum Always/LastRoundVerbatim/LastRoundBounded/…) + `.mjs` validator for future TS strategies. My check adds a nuance econ did not state: the Rust schema table is `cfg(test)`-gated documentation, and the *runtime* refusal lives in `last_round.rs:349-402` (`survives()` — the latest user round must survive verbatim or bounded or the compaction pass is refused). Enforcement is real; the honesty of the module doc about where it lives is itself on-brand. No anchor has a language-crossing compaction contract.
- **Compile-enforced protocol parity (real):** `core/protocol_parity.rs:1-8` exhaustive no-wildcard projections kept out of test cfg so `cargo build` cannot pass with an unmapped variant — rustc as the drift detector; no corpus peer.
- **FleetDenialGuard anti-gaming layer (real):** `dispatch.rs:74-98` keeps reads as SHA-256 pairs so an unchanged re-read cannot reset denial counters, with conservative eviction and no raw bytes retained — the strongest loop-breaker observed (crush's was first, this is most developed).
- **Prefix-cache drift attribution (real but attributed):** `prefix_cache.rs:1` self-declares "inspired by Reasonix's Pillar 1" — and deepseek-reasonix is itself in the sample queue; base-idea credit goes there if it scores. The keep-counting-the-miss discipline is a genuine refinement.
- Not 9: substantial design DNA is attributed-derived — COMPACT_PROMPT ported from Codex templates (`compaction.rs:174-186`), frame_rate_limiter from openai/codex, device_code from pi-mono (THIRD_PARTY_NOTICES), compaction shape codex-shaped throughout. Genuinely original mechanisms exist and ship tested, but the base is reassembly, matching the corpus 8-rung ("defended AND tested, partially derived") that opencode, qwen, smelt and hermes occupy.

### durability: 7/10

Census row was unusable on two fields; remote check on `github.com/Hmbown/CodeWhale` closes them:
- **Cadence (9-rung):** `pushed_at 2026-09-30T10:51Z` — same day as this review; CHANGELOG shows ~10-day release cadence (0.9.11 2026-08-22 -> 0.9.12 09-03 -> 0.9.13 09-13 -> 0.10.0 09-22, HEAD commit tags v0.10.1 fix); 6,600+ issue/PR numbers (#6651 at head) against a repo `created_at 2026-01-19` — extreme throughput; 31 CI workflows incl. security-audit cron, codeql, cargo-deny.
- **Contributors:** census `contributors: 1` is a shallow-clone artifact as with every other subject; remote contributors list shows a real long tail (dependabot 126, then cyq1017 112, nightt5879 97, and dozens more in the 20-65 band).
- **Adoption:** 41,042 stars / 3,565 forks / 192 open issues / homepage codewhale.net — top-of-corpus adoption, above every anchor except codex.
- **License:** MIT file present, upstream DeepSeek-TUI attribution retained — clean legal footing, no license-risk finding.
- **Docked to 7 on two counts the metadata cannot hide:** (1) institutional backing is absent — personal account (owner type: User), and the owner holds ~6,751 of ~7,300 listed contributions (≈90% commit dominance): a genuine bus factor, unlike qwen's Alibaba or codex's OpenAI floor; (2) mid-life brand pivot (DeepSeek-TUI -> CodeWhale, ~8.5 months old at this repo path) is continuity risk of its own, even though honestly handled. 7 = hyperactive, adopted, cleanly licensed, solo-core.

## Totals

| dimension | score | weight | weighted |
|---|---|---|---|
| architecture | 6 | 15 | 9.0 |
| verification | 8 | 15 | 12.0 |
| safety-enforcement | 7 | 10 | 7.0 |
| token-economy | 8.5 | 10 | 8.5 |
| orchestration | 9 | 10 | 9.0 |
| interop | 9 | 10 | 9.0 |
| operability | 8 | 10 | 8.0 |
| originality | 8 | 10 | 8.0 |
| durability | 7 | 5 | 3.5 |
| docs-dx | 8 | 5 | 4.0 |
| **weighted_total** | | 100 | **78.0** |

**Band: A (78-87) — exactly on the floor.** S-gate: numerics (78 < 88) rule S out regardless; safety-enforcement 7 and verification 8 both clear the ≥5 floor. Why not higher: architecture 6 is load-bearing — a 2,407-line `run_turn` (`turn_loop.rs:700-3106`) inside an 87%-of-tree crate with product-UI calls behind a config flag is crush's fusion dock (7) at 7x scale; safety 7 is held under 8 because the OS sandbox is automatic on macOS only (Linux bwrap opt-in, seccomp dormant, Windows contract-only) and a demanded sandbox degrades to unsandboxed with an honest receipt instead of a refusal.

**Strongest dimension: interop 9** (tied numerically with orchestration 9; interop chosen on evidence depth): MCP client + server on one shared ToolRegistry, a full ACP stdio adapter with session/load and mid-stream cancel that codex does not have, HTTP/SSE runtime API with web/mobile clients, release-CI-published typed SDK, official VS Code extension, four chat bridges, and `/import-claude` as the corpus's best rival-import story (bounded, consent-gated, secret-safe, rollback-capable — `import_claude.rs:1-26`). Only the SDK's Fleet-only scope keeps this at 9.
**Weakest dimension: architecture 6** — evidence above and in r-core.md: fused ~17k-LOC loop+policy+wiring pair, UI coupling behind `terminal_chrome_enabled` (`turn_loop.rs:728-729`), 21,206-LOC crate root, several non-test files >5k (errata rule docks both sides); credited but insufficient: alias-block extraction shape, cargo-graph boundary gate, compile-enforced parity.

## Calibration notes

- **Rule (a) uncheckable:** provenance is rename-clone (self-rebrand of DeepSeek-TUI, same author); DeepSeek-TUI is not in the corpus and is unscored, so the no-outrank ceiling cannot bind. Per the open-interpreter reversal precedent, no policy demotion for the rename itself; provenance lives in this file and the census correction below. If DeepSeek-TUI ever enters the corpus, re-check.
- **Rule (b) not triggered:** remote `archived:false`, `pushed_at` = review day; hyperactive. No cap.
- **Census corrections carried (never silent):** `provenance_flag: original` wrong -> rename-clone (honest self-rebrand; LICENSE:3, app-server/src/lib.rs:38-39, update.rs:1048); `test_loc 61,627` ~4.5x low -> ~278.7k test-like LOC (inline `mod tests` + sibling `tests.rs`, same pattern as crush/cline/nanocoder errata); `contributors: 1` remote-verified as shallow artifact (real long tail, but owner ≈90% dominance).
- **Band sensitivity — flagged for dispatcher:** 78.0 sits exactly on the A floor, within 2 pts of the B cut. The swing dimension is my own durability 7: at 6 the total is 77.5 (B), at 8 it is 78.5. Token-economy 8.5 vs 9 is the secondary swing (±0.5, can cross alone too: 78.5 at both). Per the boundary rule this subject should get an independent boundary read before A is final (precedents: gptme 76.0, kimi-code 76.5, trae-agent 44.0).
- **Distribution check:** after codewhale, 4/18 scored subjects at 80+ (22.2%) — under the 25% line; codewhale itself is 78.0, not an 80+, so no trip.
- **Dedup/concept discipline:** 35 lane findings -> 34; one cross-lane merge (c7+s12 -> crash-recovery-middleware, seeded id from crush-6). Reused seeded/corpus canonical ids: god-file-loop, ui-coupled-loop, session-tree-branching, checkpoint-revert, permission-policy, opt-in-hard-enforcement, e2e-runs-against-real-sandbox, documented-omissions, faux-provider-testing, no-fuzzing-advisory-evals, compaction-tiering, prompt-cache-marking, workflow-resume-journal, turn-budget-accounting, loop-detection, interop-matrix, headless-rpc-protocol. Convergence feed: permission-policy now x47, faux-provider-testing x34, loop-detection x26, compaction-tiering x29; fail-open family gains the mildest rung (sandbox-request-failopen: degradation with an honest receipt, vs nanocoder/gptme deception-adjacent variants).
