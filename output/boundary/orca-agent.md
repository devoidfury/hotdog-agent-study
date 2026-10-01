# orca-agent -- independent boundary re-review (T2-depth, read-only)

Main review: 78.75 / A (0.75 above floor) -> protocol-mandated independent re-score. This pass
formed its own scores from file:line reads only; the main report was read for calibration context
and its 9-rung claims were treated as hypotheses to attack, not accept.

Anchor sentence: closest anchor is **codex** -- the four dimensions that place orca above the
cline floor (safety-enforcement, token-economy, orchestration, operability) are all codex rungs
absent from cline's corpus, and the lineage is declared in the repo itself: `.boss/orca-codex-harness/architecture.md:1`
("Orca Codex-Style Harness"), compaction constants with codex/grok/claude alignment comments
(`crates/orca-provider/src/context.rs:19-38`), plugin discovery reading `.codex/plugins`
(`crates/orca-runtime/src/mentions.rs:439`), and a codex-wire decode test
(`crates/orca-runtime/src/protocol/wire.rs:1739`) -- all re-verified first-hand.

## Verification of the census/provenance (shallow clone: `.git/shallow` present, HEAD ced3454)

- LOC: total Rust 544,356 (wc); test-dir ~91k + inline `mod tests` ~148k ~= 239k test -> the
  census 544k/103k split was wrong and the main review's ~395k/~240k correction is right.
- Workspace 12 crates; `crates/orca-runtime` = 331,048 of 544,356 Rust LOC = **60.8%** (main
  said 54%; that figure is product-only, raw share is worse -- does not change the rung).
- Remote re-checked independently (protocol codel lesson): fork:false, created 2026-06-05,
  pushed_at 2026-09-30T05:05:58Z (run day), archived:false, MIT, 509 stars / 4 forks /
  0 subscribers / discussions off, owner type User (echoVic). Provenance: **original**, self-rename
  blade->orca residue confirmed (`Cargo.toml:59` `name = "blade-deepseek"`, `SECURITY.md:28`
  advisory URL names the dead repo). Census contributors:1 survived remote check -- bus factor 1
  is fact, not shallow artifact.

## First-hand reads (what I actually checked)

**Core loop.** `agent_loop.rs:35` run_agent_loop (file 724 LOC) with typed AgentLoopContext/
RuntimeTurnContext; child delegation recurses the same loop (`agent_loop.rs:244` `run_agent_loop(`);
turn body `runtime_turn_loop.rs:292` loop with child-result delivery, deduped system-guidance append
(`:303-320` marks child guidance "not a user instruction" -- a small injection-hygiene touch worth
noting). No per-entry-path loop found; the fusion cost sits one level up.

**God-file mass (architecture docking).** `runtime_host.rs` 39,726 total, `mod tests` starts :23301
-> ~23.3k product LOC in one file; ERRATA >5k rule makes this a docking factor on both sides of the
8/9 gap. Loop separation is genuinely better than crush's (crush 7 fuses inside 2.4k files); mass
keeps it below cline 8. 7 holds.

**Compaction/token economy.** `context.rs:19-46`: 0.90 hard / 0.80 soft window fractions with
codex-alignment comments; `COMPACTION_TARGET_TOKENS = 48_000` a fixed retention budget explicitly
anti-window-scaling with reasoned comment -- a better-argued target than any anchor states.
Wire-equivalent trigger (`context.rs:589-600`: request + tool schemas, pi's projected-context idea).
Cache-aware micro-compaction preserving an exact provider cache prefix (`context.rs:1778+`).
Image budget 8192 / stale-output 2KB guardrails (`context.rs:31-32`). Prompt-cache layer is
honest audit metadata, self-labeled ("audit metadata only", "Deterministic lower bound ... not
provider KV hits", `prompt_cache.rs:18-40`) -- the main review did NOT over-credit it, and 8.5
(not 9) is the correct placement: above pi's 8 (three-tier stack + wire trigger + summary cache),
below codex's 9 (no cache-key-gated reuse machinery).

**Safety enforcement.** Default AutoEdit -> WorkspaceWrite network_access:false
(`server/command_exec_sandbox.rs:194-207`); ExecutionBroker refuses launches on
Unavailable/Advisory enforcement (`crates/orca-core/src/execution_broker.rs:243-253`, unit-tested
`:362-415`); Linux strict:true builders with `fail_closed_command` exit-126 never-runs sentinel
(`orca-tools/src/sandbox/linux.rs:187,229-244,397-400`); bwrap + landlock/seccomp, no plain-shell
path. s4 silent-skip CONFIRMED: `sandbox_seatbelt_available()` returns false on non-macOS
(`tests/runtime_lifecycle_contract.rs:2389-2400`), ten+ kernel-negative tests gated on it, and
grep shows macOS appears in workflows only as a release-build matrix
(`release.yml:148,152`), never a test job -- CI never proves kernel blocking on any platform.
Workflow VM is node:vm with lexical FORBIDDEN_IDENTIFIERS/COMPUTED_PROPERTY_NAMES sets
(`workflow/host.mjs:11,21,27,467`) -- real boundary but hand-rolled. Permission rules are naive
first-match globs with no command normalization (`orca-core/src/approval_rules.rs:74-110`).
Safety 8 (well above the 6-rung, docked from 9 for s4 + s5 + s9) is correctly placed.

**Test corps (do they prove properties?).** 4,558 `#[test]` in crates, 487 in root contracts,
21 root test files spawn the real binary. Sampled `approval_contract.rs:6-38`: spawns
`CARGO_BIN_EXE_orca exec --provider mock` over JSONL and asserts exit code 3 + approval.requested +
approval.resolved(deny) + terminal status -- semantic, not existence. Journal tests are fault-injected:
`set_faults(JournalFaults{..})` around flush (`execution_journal.rs` tests :169), torn-final-line
reopen (:353), unflushed tool.started never replays (:135). CI: ubuntu nextest `--workspace
--all-targets` + clippy + dual-arch Windows test lanes (`runtime-contract.yml:51,94-95`,
`windows-ci.yml:55,130`). No fuzzing, no evals, no property framework in workflows (grep clean).
Verification 8 under the frozen ceiling is right; the s4 silent-skip places it at the band bottom
but not below, because the broker/capability/CLI/contract suites DO execute every CI and what CI
never attempts is the ceiling issue the errata already books for codex too.

**Orchestration 9 -- the key attack.** Every named exhibit verified real AND semantic:
- `LeaseReservationPool` non-double-spend comment + reservation math (`budget_controller.rs:22-29`),
  durable restore (`:91`); four optional budget dims (`orca-core/src/budget.rs`); CLI
  --max-turns/--max-tool-calls/--max-cost-usd/--max-wall-time-secs. No anchor has leases.
- Workflow repair test by name, looped over ALL three terminal outcomes with a reopened
  TaskRegistry asserting the interruption window surfaces as Failed before repair
  (`workflow/runner.rs:3584-3640`) -- this is a property test, not existence.
- Per-agent cache resume records call_path+input_hash+output+attempt+usage
  (`workflow/state.rs:21-60`).
- Detached/sync subagent recovery + foreign-session rejection (`tests/subagent_recovery_contract.rs:543-552`).
- Pause/resume/clone/restart-failed/restart-phase as CLI commands (`src/cli.rs:283-307`).
I looked for reasons to discount (thin assertions, self-referential fixtures, untested wiring) and
did not find them. **9 holds.** 10 does not: no codex-queue-grade operator surface, single-maintainer
maturity risk is real but is a durability fact, not an orchestration rung fact.

**Operability 9.** `--resume-at` tested both accept and reject
(`tests/history_contract.rs:682,754`; flag `src/cli.rs:207`); doctor read-only + stable schema +
credential redaction (`tests/doctor_cli_contract.rs:21,67`); opt-in preview-first retention. Matches
the codex/pi 9-rung on evidence; not 10, agreed.

**Interop 7.5.** MCP crate is client-only (`orca-mcp/src/` = client/lib/transport, no server);
ACP pinned `=0.10.4` (Cargo.toml:18) with daemon docs; sole provider impl is
deepseek_http.rs; no SDK package, no IDE tree. Between nanocoder 7 and cline 8. Holds.

**Docs.** Both drift gates verified: clap-tree == public-cli-manifest
(`src/cli.rs:908-930`), forbidden-claims sweep over public docs
(`tests/public_docs_contract.rs:4-20`). Identity tax verified (`Cargo.toml:59`, `SECURITY.md:28`).
8 is a fair rung -- the gates are the credit, the volume is not.

## Agree/disagree table

| dimension | main | boundary | verdict |
|---|---|---|---|
| architecture | 7 | 7 | agree (runtime_host ~23.3k product verified; loop separation real; note raw orca-runtime share 60.8% vs main's 54%) |
| verification | 8 | 8 | agree (frozen ceiling; s4 silent-skip confirmed = band bottom, not below: contracts execute every CI) |
| safety-enforcement | 8 | 8 | agree (broker refuse, strict backends, exit-126 fail-closed verified; s4/s5/s9 dock from 9 justified) |
| token-economy | 8.5 | 8.5 | agree (tier stack + wire trigger real; prompt-cache audit-only honestly not credited) |
| orchestration | 9 | 9 | **agree after attack** -- repair test, lease pool, recovery contracts, cache records all first-hand-verified semantic |
| interop | 7.5 | 7.5 | agree (ACP strong; MCP-client-only, no SDK/IDE, DeepSeek-only verified) |
| operability | 9 | 9 | agree (fault-injected journals, resume-at both ways, doctor redaction) |
| originality | 8 | 8 | agree (four corpus-firsts verified in code; codex-derived chassis caps at 8) |
| durability | 4.5 | 4.5 | agree (bus factor 1 remote-reconfirmed by me; 4 months; active release cadence keeps it above nanocoder 4) |
| docs-dx | 8 | 8 | agree (both drift gates verified; identity tax verified) |

Weighted total: **78.75 -> A.**

## A/B verdict

**Rule A (band A stands).** Hinge dimensions: **orchestration** (9 vs 8.5) and **verification**
(8 vs 7.5) -- the only two swings that could take the total below 78 (either alone: 78.25 / 78.0,
both A; combined: 77.5 = B). My independent first-hand verification of every orchestration exhibit
found them real, semantic, and CI-executed, and the verification ceiling placement (silent-skip =
in-band deduction) defensible because the executing suites cover broker/capability/CLI surfaces and
the missing kernel proofs are exactly what the frozen 8-ceiling already prices into every anchor.
Neither discount is warranted; A at 78.75 is my authoritative boundary score. No calibration rule
(a) or (b) triggers: provenance original (fork:false re-confirmed by me 2026-09-30), archived:false
with run-day push.

## Findings

Zero new findings. Every phenomenon observed in this pass maps to an existing merged concept
(god-file-host-wiring, enforcement-untested-in-ci, operation-journal-invariants,
execution-budget-engine, prompt-cache-audit-only, unrenamed-manifest-identity, regex-denylist,
permission-policy, evals-in-ci). One numeric footnote for synthesis, not a finding: orca-runtime
raw Rust share is 60.8% (331,048/544,356); the main report's 54% is product-LOC-only.
