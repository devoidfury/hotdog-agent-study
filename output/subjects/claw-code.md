# claw-code — T2 deep review

Remote: ultraworkers/claw-code (Rust workspace under `rust/`, Python "porting workspace" under `src/`).
Census: 127,586 non-test / 11,125 test LOC, 1 contributor, shallow clone (HEAD 08106b0, remote pushed_at 2026-08-16), MIT file, provenance "original", T2.

## Anchor question

Closest anchor: **nanocoder (60.5, C)** -- real-but-fail-open safety machinery, one-operator churn, behavior-asserting tests with a big presentation god-file; claw-code sits a notch lower because the god file is 14x larger than crush's docking factor, the sandbox fallback is misdescribed to the user, and compaction is LLM-free and lossy.

## Provenance (honest answer: census is wrong)

This is **not original**. It is a self-declared multi-stage rewrite of proprietary Claude Code: a Python mirror pass and a Rust rewrite, both explicitly developed against an archived Claude Code TypeScript snapshot.

- `src/__init__.py:1` -- package docstring: "Python porting workspace for the Claude Code rewrite effort".
- `src/context.py:24`, `src/parity_audit.py:7` -- `archive/claude_code_ts_snapshot/src` as the reference root; the snapshot itself is gitignored (`.gitignore:2`) and absent from the tree.
- `src/parity_audit.py:12-33` -- `ARCHIVE_ROOT_FILES` maps claude-code TS root files (`QueryEngine.ts`, `costHook.ts`, `replLauncher.tsx`, ...) 1:1 to the Python tree; `:35-69` maps every subsystem dir (`memdir`, `upstreamproxy`, `native-ts`, `outputStyles`, ...).
- `src/reference_data/archive_surface_snapshot.json`, `commands_snapshot.json`, `tools_snapshot.json` -- committed machine-readable snapshots of the proprietary tree's file surface; entries read "mirrored from archived TypeScript path tools/AgentTool/AgentTool.tsx" etc. These were extracted from the snapshot and ARE committed.
- Rust crate names and vocabulary: `rusty-claude-cli`, `rust/MOCK_PARITY_HARNESS.md` (Anthropic `/v1/messages` mock), compaction continuation strings shaped like the upstream product (`compact.rs:4-7`).
- GitHub description: "An agent-managed museum exhibit ... developed and maintained with no human intervention." README says plainly it is "not the serious production project" and redirects users to two sibling harnesses.

Distinction from claw-code-agent (a different subject -- do not conflate): that is a Python port with verbatim prompts; this is a Rust re-implementation whose *architecture, file taxonomy, and command/tool surface* were mirrored from the same proprietary lineage. I saw no verbatim proprietary prompt text in the Rust tree; the derivation evidence is structural (committed surface snapshots + 1:1 module maps). Census `provenance_flag: original` must read `port-derived:claude-code`. Lineage stats: claurst, claw-code-agent, crab-code, claw-code = four derivative shapes, one lineage.

## Census sanity

- LOC inflated: measured `find`+`wc` gives 115,957 Rust LOC total (104,547 non-test / 11,410 in `tests/` files) + 4,550 Python (694 test) = ~120.5k all-in vs census 138.7k. Non-test overstated ~19k (md/other blobs?). True code total ~108.4k non-test -- still T2.
- `commits: 1 / contributors: 1` is a shallow-clone artifact: `PARITY.md:9` logs 292 commits / 3 authors at the 2026-04-03 checkpoint alone. Remote-verified: created 2026-03-31, pushed 2026-08-16, NOT archived, 195k stars / 108k forks. Do not call dead; do note the 6-week quiet and the self-declared exhibit status.
- License file MIT; the org's MIT grant cannot clear derivation from a proprietary source any more than claurst's or kode-cli's can.

## Mandatory line-level reads

### Core loop -- `rust/crates/runtime/src/conversation.rs`

`ConversationRuntime::run_turn` at `conversation.rs:325-...`: post-compaction session-health canary (`:332-341`, probe failure aborts the turn with a named recovery command), push user text, then a bounded iteration loop (`:354-363` max_iterations error), stream request from full session clone (`:365-377`), usage recording (`:384`), assistant push, auto-compaction check on EVERY iteration including the terminal one (`:408-412`, comment cites issue #3106), tool-use extraction, then per tool call: pre-hook with input rewrite / cancel / deny channels (`:419-446`), permission via `authorize_with_context` with the hook's override carried as `PermissionContext` (`:447-458`), execute + post hooks + failure hooks (`:460+`, `:235-323`). Inline tests at `:849+` cover the user→tool→result roundtrip with usage tracking, max-iterations abort (`:1808`), and API error propagation (`:1849`). The loop is honestly structured: policy and hooks are consulted before every execution, denial returns a corrective tool-result rather than halting.

The problem is everything ABOVE the loop: `rust/crates/rusty-claude-cli/src/main.rs` is 19,831 LOC / 305 top-level functions -- REPL driver (`:7048`), turn dispatch (`:1086`, `:7751-7761`), sandbox advisories (`:4599`), ACP status (`:10632`), config/diff reports (`:1103+`), JSON error shaping with hint tables (`:654-677`). `tools/lib.rs` 10,892 and `commands/lib.rs` 7,183 join it: the three largest files are ~33% of all Rust LOC.

### Compaction -- `rust/crates/runtime/src/compact.rs` (846 LOC) + driver

- Trigger is usage-truthed: `conversation.rs:571-576` compares cumulative *provider-reported* input tokens against a threshold (env-configurable, `:188`), not a char guess -- the pi-style "measure what the model received" instinct, then forces compaction (`max_estimated_tokens: 0`, `:581`).
- Summarizer is LLM-FREE: `summarize_messages` `compact.rs:192-287` builds a `<summary>` fold -- role counts, deduped tool names, last-3 user requests, inferred pending work, key files, and a per-message timeline with every block truncated to 160 chars (`:336-358`). Deterministic, zero-cost (forge-shaped), but on prose-heavy sessions this is orientation, not memory.
- Genuinely good correctness work: tool-use/tool-result pair safety at the keep-boundary with a walk-back loop and a comment explaining the OpenAI-compat 400 it prevents (`:124-163`); prior-summary merge that deliberately FLATTENS highlights because re-nesting compounds summary inflation per cycle (`:290-330`); continuation message shaping (`:70-90`); session records each compaction (`:180`, `session.record_compaction`).
- Tests (`:574+`) assert the boundary fix, the flatten invariant, and idempotent re-compaction. `mock_parity_harness.rs` scenario `auto_compact_triggered` proves it end-to-end through the real CLI.
- No second tier, no provider-overflow recovery rung, no image handling. `estimate_message_tokens` is `len/4` (`:458-473`) but only gates the secondary config path.
- **Prompt cache: never enabled.** Zero `cache_control` hits across the entire Rust tree, yet `api/src/prompt_cache.rs` tracks `cache_read_input_tokens` and runs an unexpected-cache-break detector with attribution (`:314-345`: fingerprint-version / model-hash / system-hash diffing, 2,000-token drop floor at `:13`). On Anthropic (the default provider, `api/src/client.rs:18`) the cache it's monitoring can never turn on -- observability for a mechanism it never requests (claurst-8's exact pairing).

### Permission / sandbox -- `permissions.rs` (734) + `permission_enforcer.rs` (738) + `sandbox.rs` (499) + `bash.rs`

- `PermissionPolicy`: 5 modes, per-tool required-mode map, allow/deny/ask rules, unconditional `denied_tools` checked first (`permissions.rs:95-120`); hook overrides enter as `PermissionContext` (`:39-66`).
- `PermissionEnforcer.check` fail-CLOSES when a prompt would be required but no prompter exists (auto-deny, `permission_enforcer.rs:39-56`) -- correct default posture for headless lanes.
- Sandbox is REAL but Linux-only: `unshare --user` probe with cached candidate mappings (`sandbox.rs:284-370`), launcher builder adds `--net` on opt-in, redirects HOME/TMPDIR into `.sandbox-home/.sandbox-tmp` (`sandbox.rs:210-262`), container detection (`:110-153`). Network isolation defaults OFF (`:127`).
- **The fail-open + the lie**: when no launcher applies (macOS/Windows/no-unshare), `prepare_command` falls back to bare `sh -lc` and sets HOME/TMPDIR env vars "if filesystem_active" (`bash.rs:314-320`, `:340-347`) -- env redirection is not filesystem isolation -- while `main.rs:4599` tells the user "Filesystem isolation is still active." A `dangerously_disable_sandbox` boolean is plumbed through the bash tool input (`bash.rs:28,93`) with no extra gate visible at the runtime boundary. `bash_validation.rs` adds destructive-command warnings and semantic checks (1,004 LOC, lane-1 in PARITY.md) -- deny-aid regexes, not enforcement. Trust resolver (folder trust allowlist) exists in `trust_resolver.rs`. This is nanocoder's rung (real machinery + fail-open) with a phantom-control claim bolted on -- below it, per the rubric's "present but misleading" doctrine.

## Verification (real look)

1,385 `#[test]` + 32 `#[tokio::test]` inline across 75 files; 11,410 LOC in 14 `tests/` files. `mock_parity_harness.rs:17-171` is a scenario table (12 cases incl. `write_file_denied`, `bash_permission_prompt_approved/denied`, `auto_compact_triggered`, `token_cost_reporting`) running the built CLI in a clean env against `mock-anthropic-service` -- faux-provider e2e with approvals scripted, the codex/amazon-q pattern. `output_format_contract.rs` (5,986 LOC) pins the headless JSON contract; `resume_slash_commands.rs`, `path_scope_enforcement.rs` cover resume and workspace scoping. CI: `rust-ci.yml` gates fmt + `cargo test --workspace` + clippy + docs source-of-truth scripts + roadmap-id checks + Windows PowerShell smoke (`:64-140`); `rust.yml` is a second duplicate gate. No evals, no fuzzing, tests run on ubuntu only. Rung 6: solid, one standard design, tested -- and PARITY.md is machine-generated from the harness (`run_mock_parity_diff.py`), which is a genuinely nice CI-of-the-docs trick.

## Scores (against the frozen ladder)

| dim | score | best evidence |
|---|---|---|
| architecture | 5 | loop cleanly in `conversation.rs:325`; hooks/policy separate; but `main.rs` 19,831 + `tools/lib.rs` 10,892 + `commands/lib.rs` 7,183 = 33% of Rust LOC in three files |
| verification | 6 | 1,417 test fns; `mock_parity_harness.rs:17-171`; `rust-ci.yml:86-110` fmt/test/clippy + docs gates; no evals/fuzz, ubuntu-only |
| safety-enforcement | 4 | fail-closed enforcer `permission_enforcer.rs:39-56`; real unshare jail `sandbox.rs:210-262`; but fail-open `sh` fallback `bash.rs:314-320` + misclaimed "Filesystem isolation is still active" `main.rs:4599`; `dangerously_disable_sandbox` input channel `bash.rs:28` |
| token-economy | 4 | usage-truthed trigger `conversation.rs:571-576`; pair-safe boundary `compact.rs:124-163`; flatten merge `:290-330`; but LLM-free lossy fold `:192-287`, single tier, ZERO `cache_control` marks while a cache-break detector monitors it (`prompt_cache.rs:314-345`) |
| orchestration | 6 | `task_registry.rs:53-244` lane heartbeats + stalled-lane board; `team_cron_registry.rs`, `worker_boot.rs` (2,441); session fork `session_control.rs:288`; no budgets, crash semantics unproven, no generic loop breaker (max-iterations only, `conversation.rs:356-363`) |
| interop | 6 | MCP client/stdio/tool-bridge/hardened lifecycle (`mcp_*.rs`, 5 crates' worth), LSP client, headless `--print/--output-format json` contract-tested (5,986 LOC); ACP honestly reports "not implemented" (`main.rs:10632-10643`); no IDE surface, no published SDK |
| operability | 6 | global+workspace session store with list/latest/fork/delete (`session_control.rs:95-288`), `doctor` (`claw-analog/src/doctor.rs`), config reports + `config_validate.rs` (1,018), usage/cost estimator (`usage.rs:49-120`), error hints that name the fix (`main.rs:654-677`); no checkpoint/rewind |
| originality | 6 | parity-harness-generates-PARITY.md pipeline; post-compaction session-health probe (`conversation.rs:332-341`); summary re-nesting-inflation guard; attributed cache-break detection; honest-status ACP surface. Everything else mirrors the claude-code shape |
| durability | 3 | active (remote-verified: pushed 2026-08-16, not archived, 195k stars) but 6-month organism, self-declared "museum exhibit ... not the serious production project" (README IMPORTANT block), agent-maintained by external harnesses, census 1 contributor |
| docs-dx | 7 | 14.7k md LOC incl. per-goal verification maps (`docs/g002..g013-*.md`), MODEL_COMPATIBILITY, container docs, SECURITY.md; docs drift gated in CI (`rust-ci.yml:64-83` check_doc_source_of_truth / check_release_readiness) |

weighted_total = 5*1.5 + 6*1.5 + 4 + 4 + 6 + 6 + 6 + 6 + 3*0.5 + 7*0.5 = **53.5** → **C**

Strongest dimension: **docs-dx (7)** -- the g0xx verification maps plus a CI job that fails on stale branding/links is a discipline most B-band subjects lack.
Weakest dimension: **safety-enforcement (4)** -- real jail machinery that silently becomes an env-var suggestion off Linux, plus the product telling the user otherwise.

## Boundary-risk note

53.5 is 8.5 above the D line and 11.5 below C's ceiling -- no boundary demotion/promotion issues. No dimension near the S gates (moot).

## Notes for synthesis

- Provenance correction matters more than the score: this is the corpus's fourth claude-code-lineage entry, and the only one whose derivation is evidenced by COMMITTED surface snapshots of the proprietary tree (`src/reference_data/`).
- "Museum exhibit maintained by agents" is honest (README, PHILOSOPHY.md) and the repo deserves credit for that register -- it is exactly the pi/SECURITY.md honesty style -- but it caps durability regardless of the star count.
- Calibration rules touched: none applied (not archived; not a sync-fork of another subject). No record needed in calibration-notes beyond the census corrections above.
