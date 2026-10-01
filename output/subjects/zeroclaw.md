# zeroclaw -- T3 subject report (run 20260929-0854-t3-giant-review, integrator)

Subject: **zeroclaw** (Rust workspace, 21 crates + 3 apps, v0.8.5, edition 2024). Provenance: **original** -- upstream is itself (`github.com/zeroclaw-labs/zeroclaw`, full non-shallow history, no import-squash or rename-clone markers); one declared spec-derived module (`src/verifiable_intent/`, based on the agent-intent/verifiable-intent spec, NOTICE-disclosed). Scout, three reviewer lanes all pass; 36 merged findings (38 lane findings, 2 concept dedupes) in `merged-findings.jsonl`.

**Closest anchor: codex.** All three lanes independently converge: closer to codex than to cline/pi/crush on architecture-adjacent shape (turn-step decomposition, rollout journal), on safety (a real enforcement layer beneath the approval gate that the 6-rung trio lacks), and on economy (byte-stability cache engineering, journal-grade durability) -- but earning none of codex's own rungs: conditional sandbox, drop-only compaction, no MCP server/SDK/queue, and god-file mass that breaks the 8-9 separation story.

## Scores

| dimension | weight | score | lane | basis |
|---|---|---|---|---|
| architecture | 15 | 7 | core | crush rung: real turn decomposition (c1) vs 50.8k-LOC orchestrator (c2), 1,585-line loop fn (c4), 11-deep constructor telescope (c3), 19.2k composition root (c10) |
| verification | 15 | 8.5 | safety | ~529k inline test LOC (s12), behavior-asserting enforcement tests (s11), replay evals merge-gating CI (s9); held under 9: evals are scripted replay with stubbed model, fuzzing unwired and 3/5 targets fuzz serde (s10) |
| safety-enforcement | 10 | 7.5 | safety | shell-parsing exec policy (s5), fail-closed provenance-aware approval (s6), live-sandbox denial tests (s8); docked: fail-open NoopSandbox default (s1), landlock/bwrap compiled out of default build (s2), no Windows boundary (s4), injection defense SOP-only and warn-by-default (s3) |
| token-economy | 10 | 7.5 | econ | best cache discipline in the study (e1), honest accounting (e3); zero LLM summarization anywhere (e2), proactive budgets opt-in behind legacy 32k default (e4) |
| orchestration | 10 | 8 | econ | SOP restore-claim ledger (e5), policy-clamped subagents (containment record, alias s7), idempotent outbox (e7); no queue and admittedly unenforced delegated cost ceiling (e8) |
| interop | 10 | 8 | econ | spec-conformant ACP v1 server with durable sessions (e9), full MCP client (e10), rival-harness adapters (e11); capped: no MCP server, no SDK, breadth outrunning depth (e14) |
| operability | 10 | 8 | core | JSONL journal + provenance sidecars (c6), generation-fenced rehydrate-after-reap with regression test (c7), panic containment (crash-recovery record, alias s13); no fork/rewind/checkpoint (c8) |
| originality | 10 | 8 | **integrator** | verified in code, see below |
| durability | 5 | 7 | **integrator** | census/git metadata, see below |
| docs-dx | 5 | 9 | econ | 217-page mdbook + generated-documentation pipeline, two drift-free spot-checks (e12), actionable errors and onboarding ladder (e13) |

**Weighted total: 78.25 / 100 -- band A (78-87).** S is not in play (needs 88+); the S-band floor condition (safety-enforcement and verification both >= 5) is comfortably met at 7.5 and 8.5.

## Originality (integrator, 8/10)

The mandate is to verify the ideas are real in code, not marketing. I re-verified the candidate novel mechanisms directly in the tree; every one is implemented, not aspirational:

- **Generation-fenced rehydrate-after-reap** -- `crates/zeroclaw-infra/src/session_queue.rs:16-21` carries the per-session transcript generation with the explicit no-idle-eviction rationale ("dropping an entry would let a stale holder match again"); `rpc/dispatch.rs:4228` `rehydrate_reaped_session` holds the admission permit for the whole incarnation. The lane found no anchor subject with this mechanism at any rung (c7).
- **Trim-breadcrumb provenance sidecar** -- `session_store.rs:64-75`: whether a transcript starts with the synthetic breadcrumb is stored as "a canonical fact next to the transcript so restore never has to infer provenance from message text". Real code, real comment.
- **Rolling cache breakpoint with image placeholder** -- `anthropic.rs:44-47`: an image-only turn gets a text placeholder precisely so the cache marker never rolls backward. This is the study's best cache engineering (e1), past pi's warming, nearest codex's cache-key reuse.
- **Rival-harness adapters** -- `coding_cli.rs:9-21` shared `SAFE_ENV_VARS` allowlist over env-cleared subprocesses for Claude Code/Codex CLI/Gemini CLI/OpenCode/Grok; no anchor subject wraps competing harnesses as tools at all (e11).
- Supporting originals: architecture contract tests that mechanically enforce single-source-of-truth rules (c5), the SOP run-ledger crash semantics (e5), and a spec-derived SD-JWT verifiable-intent module that exists in-tree (`src/verifiable_intent/`).

The cap from 9: the core shape is reproduction, not invention. Turn-step decomposition is explicitly the codex StepContext pattern "reproduced independently" (c1), the journal is the codex rollout design, the exec policy is "a curated parser rather than a user-facing policy language" (s5). The novel parts are genuine and tested, but they are refinements on a codex-shaped chassis, and the loudest breadth claims (firmware, 30+ channels) add surface, not ideas -- e14's breadth/depth trap is the marketing-adjacent caution. 8/10: several study-first mechanisms, all verified real; no new paradigm.

## Durability (integrator, 7/10)

From census + git metadata measured in the tree (read-only):

- **Cadence:** head `2026-09-26`, four days before run date -- alive, not archived, so the dead-cap-at-B rule does not fire. But monthly commits are decaying off the viral launch: 756 (June) -> 469 (July) -> 392 (August) -> 383 (through Sep 26). Maturation or cooling; indistinguishable this side of a longer window, so it caps enthusiasm.
- **Contributors:** 497, with the top author at 632 of 5,412 commits (~12%) and a long tail -- not a single-maintainer bus factor, unusually broad for a 7.5-month-old repo.
- **License:** MIT/Apache-2.0 dual, LICENSE files present; the census "nonstandard license names" note is a packaging artifact. Clean for adoption and fork resilience.
- **Institutional backing:** none demonstrated. `zeroclaw-labs` is a community org; the growth profile is viral (initial release Feb 2026). Mitigating governance signals: NOTICE declares the one derived component, `docs/maintainers/excision-v0.8.0-incidents.md` documents deletions and incidents, 32-workflow CI fleet with SLSA provenance docs. Nothing resembling corporate or foundation stewardship underwrites longevity.
- Risk kept visible: velocity decay + youth + no institutional backstop is exactly the profile that flattens in 12 months; the broad contributor base and healthy governance are why this says 7 rather than lower.

## Score disagreements and resolution

No two lanes scored the same dimension, so there is no numeric head-to-head to break; the real disagreements were cross-lane claims bearing on one another, resolved here explicitly:

1. **Verification credit for the CI eval gate.** The core lane flagged "inline-test quality at codex's 8-rung pattern" and explicitly handed the call over; the safety lane went one notch above the frozen 8-ceiling to 8.5 on the strength of `regression_suite_replays_green` being merge-gating (ci.yml:814), then applied the errata's own honesty caution: it is scripted replay of 8 traces with the model stubbed, live/capability suites are "planned", and the fuzz targets are unwired with 3/5 fuzzing serde itself (s9, s10). **Resolved: keep 8.5.** It is the first anchor-band subject to break the letter of the "no evals-in-CI" ceiling; the errata forbids crediting it as true model evals, which is what 9+ would require. The scout's census-artifact finding (test_loc 34,710 vs ~529k measured inline, s12 and map.md sec 0) was adopted by every lane and independently re-confirmed by the core lane's brace-matching; nothing here is scored off the census ratio.
2. **Is the 50.8k orchestrator an architecture problem or an interop problem?** The core lane scored it as the crushing architecture dock (c2); the econ lane re-cited it as the mechanism of the interop breadth/depth trap (e14). Both readings are correct and non-conflicting: the file is the architecture defect, and its unreadability is why 30+ interop surfaces cannot be credited as deep. **Resolved: architecture 7 stands; interop 8 stands with the e14 caveat recorded, not double-docked.**
3. **Does the fail-open sandbox (s1, s2) drag safety-enforcement below the 7.5?** The safety lane interpolated 7.5 against nanocoder's identical fail-open sin (which cost nanocoder the 6 rung) offset by a much deeper application layer. The core lane's review-risk flag (session-resume auth concentrated in the 28.7k dispatch.rs) was partially retired by the safety lane's direct read at :1550-1585 (owner-binding present) with the remainder declared as a residual, not a score change. **Resolved: 7.5 stands; the declared residuals (delegate.rs escalation routing, full dispatch.rs surface) go into calibration notes as certification gaps, not deductions.**
4. **docs-dx 9 vs the dual-tree residue.** Pi's 9-rung survived "docs match code"; the core lane's migration residue (c10) could suggest cline-style docking. The econ lane's answer: the split is confined by architecture tests and never leaks into the config/CLI surface users touch, and two blind spot-checks found zero drift (e12). **Resolved: 9 accepted** -- docking requires user-visible confusion, and none was demonstrated.
5. **Strongest-dimension contest between lanes.** Core said operability (8), econ said docs-dx (9), and they are not in conflict once weights are ignored; the integrator's own originality (8) joins the cluster. Final call below.

## Strongest and weakest dimensions

- **Strongest: docs-dx (9).** Evidence: 217-page mdbook kept honest by machinery, not good intentions -- `markdown-schema`/`markdown-help` export reference docs from the live schema (main.rs:1629,6160), README install blocks are generated with do-not-edit fences (README.md:40-44), and two independent blind spot-checks matched code semantics exactly (history-management.md:38-42 vs schema.rs:4597-4602; configuration.md:25-26 vs compatible.rs:652). Error text names the exact config key to fix (history.rs:520-521). It is the only 9 any lane awarded, it exceeds pi's reference point at 7x the scale, and unlike the operability/verification 8.5s it has no errata-style discount attached.
- **Weakest: architecture (7).** Evidence: the orchestrator at `crates/zeroclaw-channels/src/orchestrator/mod.rs` carries ~50.8k production LOC (52,741 total minus 1,960 measured inline test lines) fusing routing, delivery, interrupts, commit-frontier ordering and cost -- 16x codex's worst residual file -- and cross-channel turn semantics are discoverable nowhere else (c2); a 1,585-line `run_tool_call_loop` sits on top of otherwise genuine step decomposition (c4); 11 telescoping constructors forward 15 positional params including consecutive bools (c3); `src/main.rs` still holds 19.2k lines of live composition-root behavior (c10). It also carries the heaviest weight (15), so it is the single largest gap between this subject and the S band. Note: durability also scores 7 but at weight 5 and with far less damning evidence; the architecture 7 is the weakest dimension in both the weighted and the defect-severity sense.

## Rule checks and calibration notes

- **Sync-fork rule: not applicable.** Provenance is original (scout sec 2, re-confirmed: remote matches Cargo.toml repository, no clone markers). There is no upstream to outrank.
- **Archived/dead cap: not applicable.** HEAD 2026-09-26 with hundreds of commits monthly; no demotion taken, so nothing to record under the never-silent clause.
- **Census corrections in force:** test_loc understated ~15x (inline `#[cfg(test)]` corpus ~529k LOC); production is ~550-650k, not the census 1.06M. Verification and architecture scores use measured per-file splits, not the census.
- **Declared residual review gaps (carried, uncertified):** lane-safety -- `tools/delegate.rs` escalation routing incl. the Grok-alias `--sandbox strict` relaxation (schema.rs:3004), and the full 28.7k dispatch.rs resume-auth surface; lane-econ -- per-channel turn economy inside the 50.8k orchestrator, cache byte-stability through the retry layer, zerocode/tauri/web internals, live ACP conformance (read-only mandate bars execution). None changes a score unless a future pass finds a hole.
- **Scored-10-scale convention:** all lane scores are on a 0-10 per-dimension scale; weights (15/15/10x6/5/5) sum to 100, weighted_total = sum(score x weight)/10 = 78.25.
- **Honesty note carried from lane-safety:** SECURITY.md:50-58 presents "Sandboxing Layers" without stating the OS layer is absent by default off-macOS (pi's SECURITY.md:50 is the unmet bar); this is a disclosure-prominence gap inside middle-to-good repo honesty, already inside the 7.5.
