# Boundary re-review: amazon-q-developer-cli (2026-09-30)

Provisional 65.0 (B floor); dispatched on the fuzz-exclusion hairline. Independent recount:
**59.5 / C** -- B floor NOT held, but not for the dispatched reason. The gate is fine; the
lane placements were inflated.

Anchor sentence: closest anchor is **crush** -- monolithic-loop CLI, tested in-loop approval
with zero OS enforcement, real-but-standard CI, no blocking model verification; q-cli reads as
crush-minus (much smaller test corps, no loop detection, no LSP, stalled head) and crush-plus
(better crate separation, git checkpoints, knowledge-wiki tool, institutional backing), i.e.
low-B to high-C; I land just below nanocoder's 60.5 on the strength of orch/durability.

## Adjudication 1: the rust.yml fuzz exclusion -- TRIVIA, gate not degraded

- The command is at `.github/workflows/rust.yml:76` (not :77):
  `cargo test --locked --workspace --lib --bins --test '*' --exclude fig_desktop-fuzz`.
- The excluded crate does not exist: workspace members (`Cargo.toml:3`) list exactly 10 crates;
  zero hits for `fig_desktop` anywhere in the repo, including `Cargo.lock`. It is rename-rot
  from the Fig desktop era.
- Cargo semantics for an unmatched `--exclude` under `--workspace`: non-fatal warning
  ("excluded package(s) not found in workspace"), not an error. `--exclude` accepts glob
  patterns and on the `--workspace`/default selection path an unmatched spec warns; the hard
  "did not match any packages" error is the explicit `-p` path. Corroborating sanity check:
  if it errored, every push since the crate's removal would be red -- implausible for an AWS
  repo with a release cadence, and visible-red is by definition not the silent
  exist-but-inert failure the "broken-as-configured" demotion targets.
- Net: the Test job compiles and runs tests for **all 10 real crates** on ubuntu+macos
  (`rust.yml:48-76`); the clippy job runs the full workspace with `-D warnings`
  (`rust.yml:16,47`); fmt + cargo-deny also push-gated (`rust.yml:118-137`). Nothing named
  fails to be tested. One dead flag string is config hygiene, not a verification rung.
- Side trivia (no dock): `cargo-llvm-cov` is installed (`rust.yml:64-66`) but the run line is
  plain `cargo test` -- coverage instrumented, never collected or gated (same
  "instrumented-but-unenforced" family as forge/jazz).
- **Ledger precedent (broken-as-configured CI)**: a stale/extraneous CI argument docks
  verification only if it (a) silently skips tests that should run, or (b) leaves the job red
  and the org ignores red. A no-op exclusion that cargo warns-and-continues past is neither:
  trivia, no rung move. Contrast the codebuff case (public CI genuinely runs build+smoke only)
  -- that is the (a) shape.

## Adjudication 2: shipped permission framework -- the OLD chat-cli one; agent-crate flaws are pre-ship landmines

- The shipped binary is chat_cli: workspace `default-members = ["crates/chat-cli"]`
  (`Cargo.toml:4`).
- Shipped approval engine: `crates/chat-cli/src/cli/chat/mod.rs:2390-2406` gates every tool use
  through `requires_acceptance` (Allow/Ask/Deny) with `trust_all_tools` as an explicit, warned
  opt-in (`mod.rs:2405`, `trust_all_text()` warning `mod.rs:472,1425`). The execute tool is an
  allowlist-with-escalation design: multi-line and DANGEROUS_PATTERNS (`<(`, `$(`, backtick,
  `&&`, `;`, ...) force acceptance (`tools/execute/mod.rs:51-68`), per-segment safe-command
  allowlists with `find -exec`/`grep -P` special-cases (`:92-118`). Binds in-loop AND tested:
  `tools/execute/mod.rs:303,377,415` acceptance-table tests, `chat/mod.rs:4323`
  trust_all flow test.
- The flagged `crates/agent/src/agent/permissions.rs` (unconditional `Allow` for
  ExecuteCmd/Introspect/SpawnSubagent at `:67`; ls/image_read checked against `fs_write`
  path lists at `:48-61`) is the NEXT-GEN framework and is **unreachable from any shipped
  surface**: the agent crate's own CLI is commented out with its main printing
  "Hello, world!" (`crates/agent/src/main.rs:1-21`), and chat-cli imports only
  `agent::agent_loop` types (`crates/chat-cli/src/agent/rts/mod.rs:12-14`) inside a module
  headed `#![allow(dead_code)]` (`rts/mod.rs:1`). No ChatSession/evaluate_tool_permission call
  site exists outside the agent crate itself and its tests.
- Shipped-behavior score: safety-enforcement = **6**, the cline/pi/crush "tested approval,
  nothing underneath" rung. Zero OS-sandbox evidence (grep for seatbelt/bwrap/landlock/seccomp
  in chat-cli: no hits). Both agent-crate defects stand as findings (b2, b3) with impact med,
  not high: fail-open ExecuteCmd becomes a real hole the day the framework is wired; the
  ls/image_read mismatch currently fails in the Ask direction (annoying) but shows the
  read/write policy model is confused (the comment says "reuse the same settings for fs read"
  while the code passes `fs_write` lists, `permissions.rs:47-48`).

## Adjudication 3: generated-code accounting -- confirmed, applied zero-credit/zero-blame

- Generated: 203,219 LOC in the five `amzn-*` client crates, each file bearing
  "Code generated by software.amazon.smithy.rust.codegen.smithy-rs. DO NOT EDIT"
  (e.g. `crates/amzn-codewhisperer-client/src/lib.rs:48`).
- Hand-written: 77,127 LOC total -- chat-cli 53,309, agent 13,051, semantic-search-client
  9,142, chat-cli-ui 1,307, telemetry-definitions 318. Inline `#[cfg(test)]` tests ≈ 11.8k
  (chat-cli ≈10.2k across 73 files, agent ≈1.3k, ssc ≈1.3k); no separate `tests/` tree.
- Largest hand-written file: `chat-cli/src/cli/chat/mod.rs` at 4,699 LOC -- under the ERRATA
  5k product-file docking threshold, but it fuses REPL loop, approval gating, rendering and
  compaction; architecture 6, not 7 (the 13k dead parallel framework is a dual-tree smell
  analogous to cline's documented dock, though less load-bearing).

## Lane recount vs provisional 65.0

| dim | mine | weighted | rationale (delta from a naive B read) |
|---|---|---|---|
| architecture | 6 | 9.0 | clean workspace split vs dead agent-framework parallel + 4.7k fused loop file |
| verification | 6 | 9.0 | real 2-OS clippy/test/fmt/deny CI + ~11.8k behavior-asserting test LOC; no fuzzing, evals dispatch-only (terminal-bench.yaml:6-13), coverage wired-not-collected; far from crush-7 corps scale; NOT docked for the exclusion (adjudication 1) |
| safety-enforcement | 6 | 6.0 | shipped approval tested, zero sandbox (adjudication 2) |
| token-economy | 6 | 6.0 | overflow-triggered auto-compact (chat/mod.rs:990-1012) + manual /compact strategies + token_counter; no cache discipline, no cost visibility |
| orchestration | 4 | 4.0 | no shipped subagents/queues/loop-detection; beta delegate-to-profile tool + todo lists only |
| interop | 6 | 6.0 | MCP client + headless `-c`/JSON output; no LSP, no ACP, no SDK, no IDE surface in-tree |
| operability | 7 | 7.0 | git checkpoint subsystem (init/restore/expand/diff/clean, cli/chat/cli/checkpoint.rs, beta-flagged), --resume (chat/mod.rs:231-233), diagnostics, experiment flags |
| originality | 6 | 6.0 | knowledge-wiki semantic search, checkpoints-in-agent, tangent mode, introspect tool -- verified, partial corpus priors |
| durability | 6 | 3.0 | AWS backing + dual-license (Cargo.toml:12) but remote-verified `git ls-remote origin HEAD` == snapshot head 15cc8f3 (2026-04-23) -- five months static, 6 workflows; not archived, rule (b) NOT triggered |
| docs-dx | 7 | 3.5 | mdbook tree (hooks, built-in-tools, agent-format, introspect), SECURITY.md, installer docs |

Weighted total: 9.0+9.0+6.0+6.0+4.0+6.0+7.0+6.0+3.0+3.5 = **59.5 / C**.

The gap to 65.0 is not the fuzz gate (worth 0.0 by adjudication 1); it is orchestration 4
(vs ~5), interop 6 (vs ~8 -- 8 is cline's published-SDK/four-surface rung, unearned here),
durability 6 (vs ~9 -- 9 is codex/cline institutional CI; a five-month-static head is not 9),
and originality 6. Per the claw-code boundary precedent, this recount is authoritative for the
band. Sensitivity: even the maximal lenient re-read (verif 7, interop 7, dur 7, docs 7) gives
63.0 -- still C; B would require two full rungs up on orchestration or interop, neither
supported by the shipped code.

## Calibration notes for synthesis
- Ledger precedent recorded: broken-as-configured CI ruling (see adjudication 1).
- Durability: stalled-head flag; re-check `archived` on aws/amazon-q-developer-cli at synthesis
  (rule (b) cap B would then be non-binding -- subject is already C).
- eval-harness-outside-ci convergence instance (terminal-bench dispatch-only).
- Generated-LOC accounting rule applied: 203.2k smithy LOC zero-credit/zero-blame; census
  hand-written mass is 77.1k.
