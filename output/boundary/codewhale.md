# codewhale — independent boundary re-score (T2-depth pass)

Read-only static review of `/data/samples/agents/codewhale` @ 94130d9, never executed. Main report read for calibration context only; every number below re-derived from disk or remote. Shallow clone (`.git/shallow` present) so no dead-activity inference from HEAD; durability fields closed via remote GitHub API check on `github.com/Hmbown/CodeWhale`.

**Closest anchor: codex (S 88.5)** — codex-shaped typed execpolicy crate wired into a live loop with end-to-end denial acceptance tests, three compaction paths including an LLM-free prune tier, prefix-cache pinning, and a fleet/workflow journal layer; the distance from codex is not mechanism quality but the fused 87%-of-tree crate and the platform-asymmetric sandbox default.

## Independent verification of load-bearing claims

- **Core loop (architecture):** `run_turn` spans `crates/tui/src/core/engine/turn_loop.rs:700-3106` (~2,407-line single function; next sibling `plan_tool_calls` at :3115). Engine pair `turn_loop.rs` 8,586 + `engine.rs` 8,427 = 17,013 LOC. `crates/tui` = 1,083,484 of 1,248,332 non-test .rs LOC = 86.8% of the tree. UI coupling inside the loop is real but config-gated: `terminal_chrome_enabled` used at `turn_loop.rs:725-729, 1290, 1344, 5556`. Non-test files >5k beyond the loop: `tui/src/lib.rs` 21,206, `runtime_threads.rs` 15,522, `client.rs` 14,666, `config.rs` 13,544, `tools/workflow/mod.rs` 13,544, `client/chat.rs` 8,313 — errata rule docks both sides. Counterweights verified: separate `execpolicy` (8,903), `protocol` (8,097), `runtime` (7,369), `core` crates; compile-enforced parity lives at `crates/tui/src/core/protocol_parity.rs` (main report's "core/protocol_parity.rs" path is shorthand; the file exists, no-wildcard exhaustive projections).
- **Compaction/token-economy:** three paths via `CompactionPath` incl. `PruneOnly` (`crates/tui/src/compaction/last_round.rs:23,48`); LLM-free prune machinery `crates/tui/src/compaction.rs:789-865` with projected post-prune re-check (`compaction.rs:668-669` `estimate - reclaimed < threshold*4/5`); refusal-on-violation enforcement is real code, not doc: `last_round.rs:349-402` (`survives()` + `validate_last_round_coverage` bails and does not replace history; comments record the specific false-pass bugs the checks killed — orphaned tool_use/tool_result pairs, summary-counts-as-assistant-survival). Survival contract: `SURVIVAL_CONTRACT.md` + `survival_contract.rs` (289 LOC) + `validate_survival_contract.mjs`. Prefix cache: `crates/core/src/prefix_cache.rs:1-15` SHA-256 pinned prefix + per-request verification + change attribution; self-declares derivation ("inspired by Reasonix's Pillar 1").
- **Safety:** execpolicy crate has real structure (`src/matcher.rs` compiled globs + arity-aware normalize, `approval_mode.rs`, `shell_expand.rs`, `toml_rules.rs`; deny>ask>allow layering documented at `lib.rs:100-108`). End-to-end denial property test confirmed real: `crates/tui/tests/integration/shell_denial_acceptance.rs:21-32` — denied bash cannot be reached through `task_shell_start` / `tasks` gate actions against a loopback mock provider, with a positive control (`unrestricted_task_search_and_start_still_executes`). Sandbox platform asymmetry confirmed: seatbelt automatic on macOS (`sandbox/mod.rs:352-355`), Linux bwrap deliberately opt-in (`:347-349, 359` `prefer_bwrap`), honesty test `linux_default_never_claims_an_unwired_sandbox` (`mod.rs:1013`); sandbox tree ~6.0k LOC incl. `seccomp.rs` (407, dormant) and `windows.rs` (66, contract-only).
- **Verification:** 16,653 `#[test]`/`#[tokio::test]` attributes; sibling `tests.rs` files total 222,850 LOC + `tests/` dirs 38,134 = ~261k test-like LOC before inline `mod tests` (728 files carry `#[cfg(test)]`) — census 61,627 is ~4.3x under. CI: 31 workflows; nextest with hermetic home and policy-filtered authoritative leg (`ci.yml:402-413`); cargo-deny, codeql, security-audit present. The in-CI `eval` legs (`ci.yml:908, 953`) run the offline harness that self-declares "without calling the network or any LLM endpoints" (`crates/tui/src/eval.rs:1-4`) — under the evals-in-CI rule (job must block AND observe model behavior) these do not break the verification-8 ceiling; no fuzzing anywhere (grep cargo-fuzz/afl/libfuzzer: 0 hits).
- **Orchestration:** subagent machinery 20,578 LOC (`tools/subagent/mod.rs`); fleet surfaces (`commands/groups/core/fleet.rs`, views); workflow journal with restart reconciliation (`tools/workflow/journal.rs:56-95` reconcile_snapshot/cancel/missing against the durable run record); `FleetDenialGuard` (`core/engine/dispatch.rs:76-98`) with SHA-hashed read observations so unchanged re-reads cannot reset denial counters — strongest loop-breaker observed in corpus.
- **Interop:** ACP stdio server 5,220 LOC (`crates/tui/src/acp_server.rs`), `/import-claude` 543 LOC (`import_claude.rs`), MCP client+server, published `@codewhale/runtime-sdk` (`npm/runtime-sdk/package.json:2`, workspace at `package.json:11`), VS Code extension (`extensions/vscode`), four chat bridges (`integrations/`).
- **Operability:** global panic hook confirmed at `crates/tui/src/lib.rs:1815` (re-scout's retracted risk item stands retracted); session tree in `crates/runtime/src/session_tree.rs`; request preview surface at `core/engine/preview.rs`.
- **Provenance:** rename-clone (honest self-rebrand of DeepSeek-TUI): LICENSE:3 retains "Copyright (c) 2024-2025 DeepSeek-TUI Contributors"; `crates/app-server/src/lib.rs:37-42` states it in-code ("CodeWhale began life as DeepSeek-TUI"); `npm/deepseek-tui` migration package present. Census `provenance_flag: original` is wrong. Rule (a): DeepSeek-TUI is not in the corpus, ceiling cannot bind; per open-interpreter precedent no policy demotion for an attribution-preserving rebrand.
- **Durability (remote-verified, shallow clone):** `pushed_at 2026-09-30T20:16Z` (review day; hyperactive), `archived: false`, 41,042 stars / 3,565 forks / 188 open issues, `created_at 2026-01-19`, owner type **User** (personal account, no institutional floor), MIT license clean.

## Scores (independent)

| dimension | score | weighted | vs main | note |
|---|---|---|---|---|
| architecture | 6 | 9.0 | = | crush's fusion shape at ~7x scale (single 2,407-LOC fn, 87% crate); extraction discipline credits keep it above nanocoder's 5 |
| verification | 8 | 12.0 | = | 261k+ test-like LOC, property-shaped e2e denial tests; eval legs offline + no fuzzing = frozen 8 ceiling (errata) |
| safety-enforcement | 7 | 7.0 | = | real kernel enforcement + live denial tests on macOS; Linux opt-in / seccomp dormant / Windows contract-only hold it under 8; above the 6 rung because there IS something underneath approval, and it is tested |
| token-economy | 8.5 | 8.5 | = | three paths + survival-contract refusal + pinned-prefix drift attribution; lacks codex's 9-rung hooks + image budgets (flat ~1k est.) |
| orchestration | 9 | 9.0 | = | fleet + reconciled workflow journals + budgets + best loop-breaker in corpus; codex 9 rung |
| interop | 9 | 9.0 | = | codex's 9 plus ACP (which codex lacks) and rival-import; Fleet-only SDK scope keeps it off 10 |
| operability | 8 | 8.0 | = | panic hook + crash dumps + session tree + verified auto-resume; short of codex/pi 9 rollout-journal breadth |
| originality | 8 | 8.0 | = | survival contract and compile-enforced parity verified real; DNA substantially attributed-derived (Reasonix, Codex COMPACT_PROMPT, pi) — corpus 8 rung |
| durability | 7 | 3.5 | = | same-day push, 41k stars, honest rebrand; owner ≈90% commit dominance + personal account + mid-life pivot cap it at 7 |
| docs-dx | 8 | 4.0 | = | 100-file docs tree incl. ARCHITECTURE/AUTHORIZATION_ORDER/CACHE; migration-confusion paper trail (alias blocks, "Copied verbatim" comment) is the dock |
| **total** | | **78.0** | **0.0** | |

## Boundary ruling

78.0 sits exactly on the A floor; every weight-10 dimension is a single-notch drop to B (77.0), weight-15 drops go to 76.5, and durability 7→6 gives 77.5. The genuine judgment hinges, in order:

1. **safety-enforcement 7 vs 6** (-1.0 → B). The 6 rung is "tested approval, nothing underneath"; codewhale has an automatic kernel sandbox on macOS with live denial tests AND a typed execpolicy engine enforced in-loop — something underneath, tested, just not all-platform. I hold 7 and consider 6 an over-dock.
2. **architecture 6 vs 5** (-1.5 → B). 5 (nanocoder) is a loop trapped in UI hooks forcing parallel re-implementations; codewhale's fusion is inside a Rust engine with machine-checked crate boundaries and one host surface. 6 is right.
3. **token-economy 8.5 vs 8** (-0.5 → 77.5 B). Off-ladder interpolation, but codewhale's projected post-prune re-check + survival-contract refusal exceed pi's 8 on enforcement while missing codex's hooks/image budgets; 8.5 defensible, 8 would be a defensible-strict alternative.
4. **durability 7 vs 6** (-0.5 → 77.5 B). 7 = pi's rung (strong community, thin institution); codewhale's throughput and adoption match or exceed it, its bus factor is worse than pi's only marginally. 7 holds.

None of these are placements I judge wrong, so no downward move. **Verdict: A.** Boundary score agrees with main to 0.0 on every dimension.

## Disagree/agree table (mine vs main)

architecture 6=6 · verification 8=8 · safety 7=7 · token-economy 8.5=8.5 · orchestration 9=9 · interop 9=9 · operability 8=8 · originality 8=8 · durability 7=7 · docs-dx 8=8. **Zero numeric disagreements.** Two evidence-path nits: protocol_parity lives at `crates/tui/src/core/protocol_parity.rs` (main cites "core/protocol_parity.rs"); `update.rs` migration grep did not hit case-insensitively in `crates/tui/src/update.rs` at this HEAD — provenance conclusion unaffected (LICENSE:3 + app-server/src/lib.rs:37-42 + `npm/deepseek-tui` suffice).

## Census corrections (standing test-glob errata class)

- `test_loc: 61,627` → ~261k+ measured via sibling `tests.rs` (222,850) + `tests/` (38,134) + inline `mod tests`; same miss pattern as crush/cline/nanocoder errata.
- `contributors: 1` shallow artifact; remote shows a real tail, though owner dominance (~90%) is genuine.
- `provenance_flag: original` → rename-clone (honest self-rebrand of DeepSeek-TUI; attribution preserved).
- `suggested_tier: T3` accepted (1.25M LOC, top-of-corpus adoption); this boundary pass was T2-depth per dispatch.

## Findings

No new findings. Every observation this pass made maps onto an existing canonical concept already recorded in the T3 merge (`/etc/hotdog/workflows/runs/20260929-0855-t3-giant-review/merged-findings.jsonl`, 34 records): god-file-loop (c1), ui-coupled-loop (c2), permission-policy (s1), opt-in-hard-enforcement (s3), sandbox-request-failopen (s4), e2e-runs-against-real-sandbox (s5), no-fuzzing-advisory-evals (s11), compaction-tiering (e1), prompt-cache-marking (e2), survival-contract-schema (e4), loop-detection (e7), retired-tool-replay-surface (e8), interop-matrix (e9), rival-config-import (e11), migration-confusion-surface (e13). Re-filing them under `codewhale-b*` ids would corrupt convergence counts; zero new findings is the correct output.
