# ferrum — T1 review (2026-09-29)

Rust, MIT, `https://github.com/ominiverdi/ferrum` (Codeberg-hosted releases), v0.7.9.
"A small Rust-native Linux coding agent." Shallow clone confirmed (`.git/shallow`), HEAD
2026-09-21 (recent; no activity claims needed). Single contributor per census.

**Anchor question:** closer to **crush** than any other anchor — same shape (deterministic
shell policy instead of approval prompts, repeated-tool loop breaker, resumable journal,
honest security posture) — but ferrum has zero CI, no orchestration surface, and a much
larger fused product file, which is why it lands below crush's 69 rather than near it.

## Census sanity

- Actual Rust LOC: src 39,856 + tests/ 1,752 = 41,608. Census `test_loc: 1,659` misses the
  **~13,008 inline `#[cfg(test)]` LOC** across 62 modules in `src/` (same glob-miss pattern as
  crush/cline/nanocoder). Corrected: test_loc ≈ 14.8k, non_test ≈ 26.8k (census 42,575
  overstates non-test). Tier unaffected — T1 either way.
- 477 `#[test]` functions total.
- Shallow clone: no dead/low-activity assertion made; release cadence artifacts
  (release-version.txt, v0.7.9 deb/rpm assets) suggest active solo maintenance.

## Core loop (read)

`run_turn_inner` at `src/agent/mod.rs:3147`: appends user msg → builds tool set (builtin +
history + MCP, filtered by `resolve_available_tools`) → `LoopGuard` → first loop
(`:3191`) does budget check (`ensure_provider_request_budget`), streaming or non-streaming
provider call with cancellation race (`:3244` `cancel::race_with_cancel_grace`), tool
execution, then second loop (`:3469`) for final synthesis with a second overflow-recovery
path (`:3571`). State is an `AgentSession` with explicit `TurnOptions`, event-sink trait
(`AgentEventSink`), and `TurnCancellation` — clean seams, but everything (loop, TUI,
rustyline completion, slash commands, pickers, compaction, rendering) lives in one
10,178-LOC `mod.rs`. Overflow recovery: provider-overflow triggers one forced compaction +
retry (`:3297-3311`).

**LoopGuard** (`:2498-2596`): per-round fingerprints, consecutive-repeat count,
consecutive-error count; graduated `Continue|Nudge|ForceFinal` actions; explicit config
limit plus hard 256-round cap (`:64,66`, repeat-force at 7). Tested
(`nudges_then_forces_repeated_tool_calls:2598`).

## Compaction (read)

`compact()` (`:4588-4687`): prior summaries folded (latest only) into new summarize span;
token-budget split keeping recent tail (`COMPACTION_KEEP_RECENT_TOKENS` capped at
max_context/2); `avoid_orphan_tool_results` split adjustment; protected last-user-message;
LLM summary with its own budget guard (`:4704`) and non-LLM fallback on failure;
**monotonic gate**: skip unless forced when `after >= before` (`:4672-4677`); **authority
ordering**: generated summary is placed first and immutable runtime/repo policy is re-appended
*after* it so summarized user/tool text cannot gain system authority (`:4666-4669`), tested
(`compaction_keeps_one_untrusted_summary_before_immutable_policy:7016`).
Trigger: `projected_request_tokens` (`:1782`) = max(local chars/4 estimate incl. serialized
tool defs, provider-reported `context_tokens` from latest assistant usage + estimated tail)
(`context_tokens_from_usage:8545`), checked before every model request incl. mid-loop
(`:4335-4384`), hard-fail if still above budget after compaction. One-shot summary-request
budget guard. `history_search`/`history_read` tools (`:2348-2364`) query the session JSONL
including pre-compaction archived messages, returning JSONL line numbers.
No prompt-cache discipline (cache tokens only reported from provider usage, `openai.rs:2495`);
no branch summarization; estimation is chars/4 + image heuristics (`:8595-8628`).

## Permissions / safety (read)

Two layers, no approval prompt in CLI by design ("deterministic rejection layer",
`docs/security.md:20`):
1. **shell_guard** (`src/tools/shell_guard.rs`, 1,439 LOC): full tree-sitter-bash parse
   (`:82`), 256KB byte / 20k node / 256 depth syntax limits (`:9-11`), embedded-command
   recursion with depth cap, and `shell_policy.rs` (1,053 LOC) executable tiering across
   low/medium/high. Env-assignment authority deny-list (`assignment_changes_authority`,
   `shell_policy.rs:5-39`: BASH_ENV, LD_PRELOAD, GIT_SSH_COMMAND, NODE_OPTIONS, BASH_FUNC_*…).
   Enforced at dispatch: `src/tools/mod.rs:223,237,347,363` and interactive `!`/`!!` via
   `src/agent/mod.rs:9614`. Default tier is medium (allow builds/network, deny
   substitutions, inline interpreters, out-of-root mutation).
   Table-driven tier-contract test mirrors the docs matrix
   (`tier_capability_contract_is_table_driven:1267`).
2. **ACP client approvals**: under ACP, policy-passing (and all MCP) tool calls escalate to
   the editor client via `session/request_permission` (`src/acp.rs:172-243`), with
   cancellation; order is policy-first then client-ask (`src/agent/mod.rs:4094-4119`).
Plus: cgroup-v2 delegated containment per bash/MCP child with pre_exec attach and
`cgroup.kill` whole-tree kill (`src/process_containment.rs:22-74`, used `bash.rs:101-120`,
`mcp.rs:7`), atomic identity-checked writes with protected credential targets
(`write_policy.rs`, `atomic_file.rs`), 0600 history files (`agent/mod.rs:8794-8804`),
terminal-text sanitization. Docs honest to an unusual degree: "Ferrum is not a sandbox"
(`docs/security.md:31`), containment design note explicitly non-committal
(`docs/containment.md:3`).

## Verification

~14.8k test LOC / 477 tests, and they assert properties: tier table contract, compaction
ordering and monotonic gates, session journal semantics (43 tests in `jsonl.rs`), MCP frame
limit boundaries (`mcp/transport.rs:527-603`), 26 whole-binary ACP stdio conformance E2Es
spawning `CARGO_BIN_EXE_ferrum` (`tests/acp_stdio.rs:86`) against a **compiled-in fake
provider** scripted via `FERRUM_FAKE_SCRIPT` env (`src/providers/fake.rs:30`; faux-provider
seam in the shipped binary). `bench/` has 10+ scored task harnesses with validate/score
scripts — manual only. **Zero CI**: no `.github/`, no workflow files of any kind; nothing
runs any of this on any commit. No fuzzing, no evals-in-CI.

## Interop / orchestration / operability

ACP agent built on `agent-client-protocol-schema` 1.4 (`Cargo.toml:15`), docs/acp.md +
docs/zed.md; MCP stdio client with bounded frames (2,590 LOC incl. transport); headless
`-p` print mode (text only; no `--output-format json`). No subagents, no queues, no
background tasks (deferred with honest design note `docs/background-tasks.md:3`);
`wait` tool for foreground polling. Sessions: JSONL journal with mode/title/tools/compaction
records, `sync_checkpoint`, bounded session listing per cwd
(`session/jsonl.rs:564,804`), `/usage day|week|month` rollups incl. cache tokens, `/perf`
latency recorder, man page + deb/rpm packaging.

## Scoring rationale vs rungs

- architecture 6: clean provider/tools/session/acp/mcp/config separation and explicit
  loop-state model, but 10,178-LOC `agent/mod.rs` fuses loop+TUI+policy-wiring; errata says
  >5k product files dock both sides of a rung; crush (7) fuses 2.4k+1.9k, so ferrum sits below it.
- verification 6: corpus and E2E harness are genuinely property-level, but the 7 rung
  (crush/nanocoder) requires CI execution; ferrum has none.
- safety-enforcement 7: above the 6 "tested approval" rung — default-on grammar policy,
  table-tested tiers, hijack-var block, cgroup.kill containment, honest limits; below 8 —
  no kernel isolation, syntax layer is accident-guard not boundary (docs agree).
- token-economy 7: measured-usage projection trigger + monotonic compaction gate +
  overflow ladder beats the 6 rung (crush/nanocoder single auto-summarize); lacks cline-7's
  cost-projection breadth? no — cline 7 has budget projection too; ferrum matches, no cache
  discipline, no branch summaries keeps it under pi's 8.
- orchestration 4: loop guard + cancellation + resume journal only; nothing above the
  codel-4 queue rung except journals; nanocoder-6 (daemon+subagents) out of reach.
- interop 7: ACP conformance-tested + MCP client ≈ nanocoder/crush rung; no SDK, no JSON
  headless stream, no MCP server.
- operability 7: strong journal + usage/perf diagnostics + packaging; missing crush-8's
  recover-middleware-at-boundaries posture.
- originality 7: compaction authority-ordering invariant, table-driven capability contract,
  in-binary scripted provider, cgroup.kill containment — all real in code; each idea has
  corpus company elsewhere (grammar policy, faux providers), so not 8+.
- durability 4: solo maintainer, no CI/CD, Codeberg; release artifacts and honest docs above
  nanocoder's 4 signals but same one-person ceiling.
- docs-dx 8: 26 in-repo docs incl. man page, spec, and security contract docs whose tables
  are test-enforced; pi-9 has breadth+generated reference; ferrum is 8-quality at T1 scale.

**Strongest dimension: docs-dx (8)** (safety-enforcement and token-economy tie at 7).
**Weakest dimension: orchestration (4).**

Weighted total: 63.0 → band C. Boundary note: within 2 pts of B (65); any reviewer who
scores verification or safety a point higher would cross it. Calibration: no rule applies
(original work, `docs/spec.md` states pi-inspired but not a port; not archived).
