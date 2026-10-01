# goose — T3 integrator report (run 20260929-0856-t3-giant-review)

Subject: goose @ 04ed836 (2026-09-25), `/data/samples/agents/goose`. Static read-only integration; the subject was never executed. Inputs: `r-core.md` + `f-core.jsonl` (7), `r-safety.md` + `f-safety.jsonl` (9), `r-econ.md` + `f-econ.jsonl` (12), scout `map.md`. Merged findings: `merged-findings.jsonl` (28 records).

**Band: B (weighted total 73/100). Closest anchor: cline.**

## Final scores

| dimension | weight | score | owner | placement in one line |
|---|---|---|---|---|
| architecture | 15 | 7 | reviewer-core | shipped default is the crush-7 shape (6,161-LOC agent.rs, 1,147-line `reply_internal`); the 9-rung goose-agent engine is default-off |
| verification | 15 | 8 | reviewer-safety | frozen 8-rung ceiling: property-style security tests + PR-gated conformance/live smoke; no fuzzing, non-gating model sweep |
| safety-enforcement | 10 | 5 | reviewer-safety | machinery at cline/pi/crush-6, delivery at nanocoder-5: the one enforcement rung (approval) is default-off |
| token-economy | 10 | 8 | reviewer-econ | usage-measured trigger + declared cache semantics; missing codex-9 extras (no-LLM budget tier, image budgets, hooks) |
| orchestration | 10 | 8 | reviewer-econ | durable cron + journal subagents + steer + loop-breaker; turn-only budgets (e12) block 9 |
| interop | 10 | 9 | reviewer-econ | ACP both directions with conformance CI, rival CLIs as providers, CI-published SDKs, headless json |
| operability | 10 | 8 | reviewer-core | WAL journal resume, truncate rewind, versioned diagnostics bundle, llm_request ring logs; no checkpoints/session tree |
| originality | 10 | 8 | integrator | verified in code, not marketing (see below) |
| durability | 5 | 4 | integrator | AAIF/Linux Foundation backing, Apache-2.0, active head; cadence/contributors unverifiable from shallow clone |
| docs-dx | 5 | 8 | reviewer-econ | broad verified doc tree; docked for the e10 default-behavior drift |

Lane scores are already expressed on their weight scales, so the weighted total is the direct sum: 7+8+5+8+8+9+8+8+4+8 = **73**. Bands: S ≥88 (with safety/verification floors), A 78-87, B 65-77 → **B**. 73 is not a cap artifact (see rules below); the A boundary (78) is 5 points away and the cheapest honest path there is not available to goose: it would need safety-enforcement to reach ~8 (approval default-on plus something under it), which the code does not ship.

## Merged findings

28 findings merged from three lanes, deduped by concept. No two lanes coined different ids for the same concept, so no id consolidation was needed; every record keeps its original id as `canonical_id` with `aliases: []` and a `lane` field. Three adjacent-but-distinct pairs were kept separate and cross-referenced via `related`: `goose-c6`/`goose-e9` (foreign importers vs the headless contract, both touch session import/export), `goose-c7`/`goose-e4` (process-level panic containment vs scheduler crash recovery), `goose-c4`/`goose-s3` (journal-derived confirmation idempotency vs the fail-closed permission algebra). Deduping kept stronger evidence in exactly one place where overlap was tempting: the default-off discount for the state machine lives in `goose-c2` only, not re-counted against token-economy (econ explicitly declined that discount since both loops carry the compaction paths, `f-econ e2`).

## Score disagreements and how they are resolved

The three lanes own disjoint dimensions, so no two reviewers produced conflicting numbers for the same dimension. There were, however, four cross-lane tensions an integrator must settle:

1. **Closest anchor: codex (econ) vs crush/cline (core) vs nanocoder (safety).** Each lane stated its anchor within its own slice and they disagree about the whole animal. Resolution: **cline** is the closest overall anchor. Goose matches cline on operability (8/8), orchestration (8/8), docs-dx (8/8), sits one notch below on architecture (7 vs cline's docked 8) and safety delivery (5 vs 6), and exceeds cline only on interop (9 vs cline's ~7; cline's ACP commitment is server-only per the econ lane's own comparison). Codex remains the mechanism-level anchor for the four econ dimensions and verification, but codex's safety-enforcement sits on sandbox rungs goose removed (seatbelt, `blog 2026-02-23:22`), which dominates any whole-subject distance.
2. **Scout claimed "no rewind concept"; core proved truncate-based rewind exists** (`term.rs:299`, `manage_sessions.rs:224`, `fork_session.rs:39`). Code wins: rewind is credited inside operability 8; what remains absent is git-checkpoint-style revert, which is why operability stops at 8.
3. **Prior inventory (`/data/samples/reports/goose.md`, a B-band quality-7 feature survey) claims "no prompt caching"; econ found `cache_semantics.rs` with lookback-aware breakpoint placement and exact-index tests.** Code wins again: prompt-cache-marking is real (`goose-e3`, re-verified by the integrator at `crates/goose-provider-types/src/cache_semantics.rs:10-46`) and is credited in token-economy 8 and originality; the prior report is stale on this point and is not used as evidence.
4. **The same default-off fact appears in three lanes** (core c2 for the engine, safety s1/s8 for approval/scanner, econ e10 for tool-pair summarization). These are three different shipped defaults, not one finding triple-counted, so all three stand; but the *score impact* is deliberately counted once per dimension: architecture 7 absorbs c2, safety 5 absorbs s1/s8, docs-dx 8 absorbs e10, and no additional cross-dimension penalty was applied for the general "flag-gated repo" theme.

## Integrator-owned dimensions

### Originality: 8/10 (weight 10) — verified in code, not marketing

The marketing-adjacent claims were checked against source, not README:

- **Generic engine crate** (`goose-c1`): `crates/goose-agent/Cargo.toml` deps are `goose-provider-types`, tokio, rmcp, etc. -- no dependency on `goose`; `machine.rs:158-175` is a real load-step-apply loop over a `SessionLoader`/`EffectHandler` runtime. Not vaporware; but it is default-off (`state_machine/mod.rs:72-75` `enabled()` returns false unless `GOOSE_STATE_MACHINE` is set, re-verified), which caps originality rather than cancelling it: the corpus contains the substrate, shipping it is a config away.
- **LLM permission judge** (`goose-s4`): `permission_judge.rs:66-68` carries the untrusted-data/fail-closed instructions verbatim in the prompt template, with the injection test at :228-247. This is the corpus's only model-as-approver -- genuinely novel, and novel in the direction of more attack surface, which is why the lane correctly noted codex's deterministic execpolicy out-asserts it.
- **Prompt-cache semantics as declared data** (`goose-e3`): `CacheSemantics::for_model` enumerates per-(provider,model) cache behavior with a safe `ImplicitStrict` default, gateway-claude cases, and exact marked-index tests. Nobody else in the anchor set models this as data.
- **ACP in both directions** (`goose-e7`): `impl Provider for AcpProvider` at `acp/provider.rs:677` -- re-verified; goose mounts rival ACP agents as model backends, not just serves ACP.
- **Rival harnesses as providers** (`goose-e8`): `providers/claude_code.rs:255` doc-comment describes spawning the Claude Code CLI as a persistent child process driven over stream-json. Verified.

Why not 9: much of goose is first-class assembly of known patterns (cron scheduler, compaction tiers, subagent sessions, diagnostics), and the single most original artifact -- the engine -- is what the project itself does not run. 8 = multiple verified corpus-first mechanisms with one credibility ding for the flagship being off by default.

### Durability: 4/5 (weight 5) — census metadata

- **License:** Apache-2.0 (`Cargo.toml:11` + LICENSE file). No relicensing risk signal.
- **Institutional backing:** the strongest in the cohort tier: Block donated goose to the Agentic AI Foundation at the Linux Foundation alongside MCP and AGENTS.md (`documentation/blog/2026-04-07-goose-moves-to-aaif/index.md:11`); `Cargo.toml:13-15` authors `AAIF <ai-oss-tools@block.xyz>`, repository `aaif-goose/goose`. Not archived, not dead, not a sync fork.
- **Commit cadence / contributors:** NOT directly measurable -- the census clone is shallow (1 commit), stars are null in the manifest. Indirect signals used instead: head commit 2026-09-25 is four days before the scout date (active), PR number #12522 at head implies a long, high-volume history, version 1.52.0, 29 blog posts in 2026 in-tree. These support "active" but cannot establish contributor breadth (single-maintainer risk is unfalsifiable from census data).
- Docked one point for exactly that: unmeasurable cadence/contributors, plus a governance transition only ~6 months old at review time and residual block.xyz contact identity.

## Strongest and weakest dimensions

- **Strongest: interop (9/10).** Evidence: `AcpProvider implements Provider` (`acp/provider.rs:274,677`) making this the only anchor-subject ACP implementation in both directions; server surface with fork/load/delete/list + auth/TLS transport (`acp/server.rs:44-60`, `acp/transport/`); per-PR MCP conformance (`mcp-conformance.yml:4-11`) and live-provider smoke (`pr-smoke-test.yml:1-2,157`); CI-published pypi/maven/npm SDKs; rival CLIs consumable as subscription backends (`claude_code.rs:255`, `codex.rs:139`).
- **Weakest: safety-enforcement (5/10).** Evidence: `GooseMode::Auto` is the `#[default]` (`goose_mode.rs:22-25`) and every host resolves a missing `GOOSE_MODE` to it (`agent.rs:396`, `session/mod.rs:1364`), so a fresh install executes tool calls with no prompt; the injection scanner is `unwrap_or(false)` (`security/mod.rs:72`) and fails open when enabled (`scanner.rs:304`); the egress inspector hard-Allow after detection (`egress_inspector.rs:348-385`); the sole OS sandbox was removed and the removal announced (`blog 2026-02-23:22`). What holds it at 5 rather than 4: fail-closed permission internals (`permission_inspector.rs:111`), tested deny-dominance algebra, atomic permission storage, and the on-by-default OSV stdio gate (`stdio.rs:21`).

## Calibration notes

- No band demotion applied or needed: provenance is **original/upstream** (Block→AAIF transfer documented in-repo; remote and PR-number corroborate; census `rename_clone_suspect: false`), so the sync-fork-must-not-outrank-upstream rule does not engage. Subject is not archived/dead, so the B-cap rule does not engage. B here is arithmetic (73), not a cap, and recording it here per the never-silent rule.
- safety-enforcement 5 clears the S-band floor check trivially only because the total is nowhere near 88; if a future run lifts the total, the 5 on safety must be re-examined against the floor.
- Test-density dispute resolved: scout's ~4% ratio used the inflated 882k census denominator (includes `documentation/` 311MB + `ui/`); safety's honest engine-side ratio (~30k tests / ~270k Rust) is used. Verification stays 8 -- the ceiling comes from missing fuzzing/sanitizers/gating evals (`model-toolcall-conformance.yml:8-12` self-declares non-gating), not from density.
- Interop 9 keeps the econ lane's caveat: roaming/telegram edge quality was deferred and the safety lane neither verified nor refuted it; the egress log-only gap (`goose-s5`) was judged an enforcement failure (counted in safety 5) and deliberately not double-counted against interop.
- Originality and durability are integrator scores made from direct code reads and census metadata respectively; both are recorded as `lane: integrator` in the prose tables. Durability would rise to 5 with full-history metadata (cadence/contributors) from the source remote, or fall to 3 if the AAIF project showed dormancy -- neither is checkable read-only here, and we did not fabricate it.
- Prior report drift flagged for the corpus: the older `/data/samples/reports/goose.md` survey ("no prompt caching", "no rewind") is contradicted by code at head and was excluded from evidence.
