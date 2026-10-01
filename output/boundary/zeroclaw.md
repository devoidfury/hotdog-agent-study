# Boundary re-review: zeroclaw (independent T2-depth pass, full rubric)

Subject: `/data/samples/agents/zeroclaw`, HEAD `0c28d7c` (2026-09-26, not shallow).
Independence: main report `subjects/zeroclaw.md` read for calibration context only AFTER my line reads below were done; anchors read were the frozen excerpts + errata only. No subject code executed.

Scale check (my own brace-matched measurement, read-only): production 591,948 LOC + inline `#[cfg(test)]` corpus 543,372 + standalone tests/benches 41,534, across 1,163 `.rs` files; 19,693 `#[test]`/`#[tokio::test]` functions. Census (`test_loc: 34,710`, `non_test_loc: 1,063,392`) is wrong in both directions -- the census test-glob is blind to Rust inline tests; my scores use measured splits. Census errata class: applies, confirmed.

Provenance: **original** -- origin remote `https://github.com/zeroclaw-labs/zeroclaw.git` matches `Cargo.toml:9` repository field; full 5,412-commit history from 2026-02, no rename/import markers; NOTICE declares the one spec-derived module (`src/verifiable_intent/`). Rule (a) not applicable -- no upstream to outrank.

## Closest anchor (one sentence)

Closer to codex than to any other anchor -- it is the only non-codex subject I have found with a tested enforcement layer beneath the approval gate, a journal-grade session store, and cache-key-level provider engineering -- but it earns none of codex's individual rungs, so at 77.5 it sits just below pi's 78.5 on the ladder.

## Architecture — 7 (agree)

- Crate split is genuine codex idiom (21 crates + 3 apps, `Cargo.toml:2`): api/config/runtime/channels/providers/infra separation, turn-step decomposition in `crates/zeroclaw-runtime/src/agent/turn/mod.rs` with an explicit `TurnState` (`:996`) and `ToolLoop` context struct.
- But the mass story is real: `crates/zeroclaw-channels/src/orchestrator/mod.rs` measures 52,741 total / 50,770 production LOC (my brace-match) -- ~16x codex's worst residual file -- fusing routing, delivery, interrupts and cost with no other place to learn cross-channel turn semantics; `src/main.rs` still holds 12,336 production LOC of composition-root behavior; `run_tool_call_loop` spans `turn/mod.rs:928-2513` (~1,585 lines) on top of the decomposition.
- Crush rung (7): better separation than crush's fused `agent.go`, far worse mass than cline's 8-rung "thin hosts." 7 is right and carries the heaviest weight.

## Verification — 8 (DISAGREE with main's 8.5; deciding hinge)

- The corps is real and I re-measured it: 543k inline test LOC, 19,693 test functions, behavior-asserting -- live Landlock denial tests (`crates/zeroclaw-runtime/src/security/landlock.rs:984-1019`: write-to-readonly-root, read-from-writeonly-root, unlisted-sibling all asserted denied) run merge-gated in a dedicated job (`ci.yml:992` test-landlock, aggregated into the single required `gate` at `ci.yml:1305`); approval-gate semantics tested with scripted providers (`turn/approval_gate.rs:419,483`).
- The CI eval gate exists: `crates/zeroclaw-eval/tests/regression_suite.rs:21` `regression_suite_replays_green` runs the 8 fixtures in `evals/regression/` on every workspace nextest run (`ci.yml:814`) inside the required gate. It is genuinely merge-gating; I do not dispute that.
- **But it is scripted replay against a stubbed model** (`crates/zeroclaw-eval/src/replay.rs:1-3` -- a `ModelProvider` that replays canned `LlmTrace` steps). That is the same scripted-mock-provider design that pins codex at the frozen 8 (204 suite modules driving full approval flows against scripted mock SSE servers), and the errata's 8-ceiling ("no in-CI model evals, no fuzzing") describes this tree exactly: no live/capability eval in any of the 32 workflows, and the 5 fuzz targets in `fuzz/fuzz_targets/` are never run in CI (grep `fuzz` across `.github/workflows/` = 0 hits), 3 of the 5 fuzz serde itself.
- Ruling: the errata blocks 9+ without real evals/fuzzing; the 8.5 rewarded the *letter* of "an eval ran in CI" for a mechanism codex already has in equivalent form. Awarding it would put zeroclaw strictly above frozen codex on verification, which my own reads do not support (the one genuinely extra artifact -- the fixture-fail-sensitivity meta-test at `regression_suite.rs:33` -- is a single test, not a tier). 8, the ceiling rung, per the standing errata and consistent with the cline/open-interpreter boundary lanes, which held their 8 for identical missing-evals/fuzzing reasons.

## Safety-enforcement — 7.5 (agree, with a certification gap noted)

- Real parser-backed exec policy: shell-word parsing with dialect handling, env-assignment stripping, normalization (`crates/zeroclaw-config/src/policy.rs:1023,1189,1413`) -- a curated parser, not regex deny-listing, not a user-facing DSL.
- Enforcement beneath the gate is tested live: landlock denial tests as above; seatbelt/firejail/bwrap/docker backends (`security/` module, 25 files); approval is fail-closed in-loop with backchannel cancellation tested (`approval_gate.rs:483`).
- Docked exactly as the main review did, and I re-verified every dock: `NoopSandbox` fallback on every unavailable/explicit path (`security/detect.rs:350,368,376,390,398`); `sandbox-landlock`/`sandbox-bubblewrap` compiled OUT of the default build (`Cargo.toml:242-248` default vs `:349-350`), so default Linux users get a sandbox only if firejail/docker happen to be installed; `SECURITY.md:50-58` presents "Sandboxing Layers" (workspace isolation, command allowlisting, forbidden paths) with no statement that the OS layer is absent-by-default off macOS -- below pi's `SECURITY.md:50` disclosure exemplar.
- 7.5 interpolation between the 6-rung ("tested approval, nothing underneath") and codex's 10 stands: nanocoder's identical fail-open sin cost it the 6 rung, but zeroclaw's tested machinery depth (CI-gated kernel denial where landlock exists) is beyond anything at 6-7 on the frozen ladder. Residual, uncertified (carried, not docked): the full 28.6k-line `rpc/dispatch.rs` resume-auth surface and `tools/delegate.rs` escalation routing.

## Token-economy — 7.5 (agree)

- Best cache engineering I have read in the study: rolling Anthropic breakpoint with an `[image]` text placeholder so the cache marker never rolls backward past an image-only turn (`crates/zeroclaw-providers/src/anthropic.rs:43-48`); projected provider-facing token counts (`turn/mod.rs:317`) with budget enforcement against reported usage (`:520`).
- Zero LLM summarization anywhere, by design: middle-drop trim with synthetic breadcrumb (`agent/history.rs:451,519`) and an explicit "No summarization, no splicing" overflow retry (`agent/loop_.rs:2808`); context loss is deterministic, never a summary's silent paraphrase -- but there is no summarization tier equivalent to pi's branch summaries or codex's three-tier ladder.
- Budget honesty is good, defaults are soft: opt-in proactive-trim fraction (`schema.rs:3778`) behind `UNCONFIGURED_CONTEXT_WINDOW_FALLBACK = 32_000` explicitly labeled legacy (`schema.rs:5255,3806`).
- Cache discipline ~9-rung, compaction ~5-6-rung, projection/budget ~7: 7.5 interpolation holds.

## Orchestration — 8 (agree)

- Generation-fenced session queue: per-session transcript generation, explicitly exempt from idle eviction with the stale-holder rationale in-comment (`crates/zeroclaw-infra/src/session_queue.rs:17-22`).
- SOP run engine with CAS claim semantics and rollback of sibling reservations (`crates/zeroclaw-runtime/src/sop/dispatch.rs:1021-1072`); idempotent cron outbox (`runtime/src/cron/outbox.rs`); subagent delegation gated by per-alias target-mode policy (`tools/delegate.rs:586` `delegate_target_mode`).
- Cline rung (8): durable scheduling + containment + journals; below codex's 9: no first-class queue, delegated cost ceiling admittedly unenforced.

## Interop — 8 (agree)

- ACP server is real and deep: `crates/zeroclaw-channels/src/orchestrator/acp_server.rs` (10,680 LOC) with durable session store (`infra/src/acp_session_store.rs`, idempotent principal migration test `:1762`); full MCP client plane (10 `mcp_*` files in `crates/zeroclaw-tools/src/`); rival-harness adapters wrapping Claude Code / Codex / Gemini / OpenCode / Grok CLIs with a shared env-clearing allowlist (`crates/zeroclaw-tools/src/coding_cli.rs:15-17`) -- no anchor wraps competing harnesses as tools.
- Capped below 9: no MCP server, published crates are nominal during the "microkernel transition" (`Cargo.toml`: workspace crates `publish = false`), and the 51-channel breadth outruns depth -- the unreadable orchestrator is exactly why breadth cannot be credited as deep.

## Operability — 8 (agree)

- JSONL session store + trim-breadcrumb provenance sidecar kept as a canonical fact beside the transcript (`infra/src/session_store.rs:64-75`); rehydrate-after-reap with the admission permit held for the incarnation (`dispatch.rs:4228`, replacement guard `:4472`); panic containment in the hook runner (`runtime/src/hooks/runner.rs:155`); stall watchdog (`infra/src/stall_watchdog.rs`).
- Below pi/codex's 9: no fork / rewind / checkpoint -- grep for fork/rewind/checkpoint in `session_store.rs` = zero hits; sessions resume but never branch.

## Originality — 8 (agree)

Every claimed mechanism verified in code myself: generation-fenced rehydrate-after-reap (`session_queue.rs:17-22` + `dispatch.rs:4228`), trim-breadcrumb sidecar (`session_store.rs:64-75`), rolling cache breakpoint with image placeholder (`anthropic.rs:43-48`), rival-harness CLI adapters with shared env allowlist (`coding_cli.rs:9-17`), SOP CAS claims (`sop/dispatch.rs:1021`). All real, none marketing. Capped at 8: the chassis is independently-reproduced codex (turn-step decomposition, rollout journal, curated exec policy), and the noveltices are refinements, not a new paradigm.

## Durability — 7 (agree)

Alive: HEAD 2026-09-26; monthly commits 1376/1358/362/316/754/460/402/384 (Feb->Sep, my `git log`) -- decaying off a viral launch but no dead-cap trigger; 497 contributors, top author 632/5412 (~12%), no single-maintainer bus factor; MIT/Apache-2.0 dual, LICENSE files present; 32 CI workflows incl. a single required `gate` (branch-protection comment `ci.yml:1299-1301`). Community org, no institutional backstop -- pi's 7 rung, same thin-stewardship profile with broader contributor base.

## Docs-dx — 9 (agree)

224 in-repo markdown pages under `docs/`; drift is gated by machinery, not intention: `installer-drift` job merge-gated (`ci.yml:1225`, aggregated `:1305`), `docs-style` job, README install blocks generated with do-not-edit fences (`README.md:40-44`), `schema-export` in default features (`Cargo.toml:247`) so reference docs derive from the live config schema. Spot-check of the README fence matched the generator marker. No user-visible dual-tree confusion at the config/CLI surface. 9 holds.

## Score table vs the main T3 review

| dimension | weight | main | boundary | delta |
|---|---|---|---|---|
| architecture | 15 | 7 | 7 | -- |
| verification | 15 | 8.5 | **8** | **-0.5** |
| safety-enforcement | 10 | 7.5 | 7.5 | -- |
| token-economy | 10 | 7.5 | 7.5 | -- |
| orchestration | 10 | 8 | 8 | -- |
| interop | 10 | 8 | 8 | -- |
| operability | 10 | 8 | 8 | -- |
| originality | 10 | 8 | 8 | -- |
| durability | 5 | 7 | 7 | -- |
| docs-dx | 5 | 9 | 9 | -- |
| **total** | | **78.25 (A)** | **77.5 (B)** | **-0.75** |

## A/B verdict: B

**B, 77.5.** The deciding dimension is verification: 8.5 -> 8 (-0.75) is the single hinge that fires, and it fires on the errata's own terms -- the frozen 8-ceiling (tested semantics, no in-CI model evals, no fuzzing) describes this tree exactly, the merge-gating CI "eval" is scripted replay against a stubbed model (`replay.rs:1-3`), and crediting it above codex's identical-scripted-mock 8 would invert the ladder. Other downward hinges considered and NOT taken: safety 7.5 (tested kernel denial CI-gated where compiled in -- strictly above every rung below 7.5), token-economy 7.5 (9-rung cache offsets 5-6-rung compaction), docs-dx 9 (drift machinery verified). Even one step at 7.5-rung dimensions (-0.5) would also cross the 78 floor, so A here requires accepting the verification 8.5 letter-break; my independent read does not. Per precedent, the boundary score is authoritative: **77.5 / B is final** (demotion recorded in calibration-notes.md, not silent).

## Census errata for synthesis

`test_loc: 34,710` understates the inline `#[cfg(test)]` corpus ~15x (measured 543,372 + 41,534 standalone); `non_test_loc: 1,063,392` overstates production (~592k measured). New archetype: Rust inline-test blindness (opposite direction from the deepagents/acr fixture-inflation family; filed as zeroclaw-b3 under the shared census-discipline id with nuance noted).
