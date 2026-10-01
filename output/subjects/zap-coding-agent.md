# zap-coding-agent -- T1 review

**Anchor question.** Closest to **nanocoder (60.5, C)**: a solo-maintainer, feature-dense
single-product agent with genuinely tested permission mechanics but wrong safety defaults
and thin institutional signals -- with zap sitting slightly below nanocoder on verification
(nanocoder's specs run in CI with a coverage gate; zap's do not run in CI at all) and slightly
above it on token discipline.

## Facts / census sanity

- Census: Rust, non-test 54,039 / test 1,847, 1 contributor, 1 commit, shallow, HEAD 2026-08-19, T1.
- **Corrections.** `.git/shallow` present; 1 commit is a clone artifact, do not read history depth from it.
  `test_loc: 1,847` undercounts: there are ~3,969 LOC of in-file `#[cfg(test)]` modules in src
  (378 `#[test]`/`#[tokio::test]` fns); real test LOC ~= 5.8k. `non_test_loc: 54,039` overcounts the
  agent core: cloc across `src tests` = 34,739 Rust code (of which ~3,969 inline tests => ~30.8k core),
  and 55.9k repo-wide only when counting website/markdown/HTML (website/, content/, demos/, drafts/).
  Tier T1 still stands on either number.
- Version 0.15.142 (Cargo.toml:3) implies a heavy patch-release cadence despite the single visible commit.

## Provenance & license

- Census flag `original`; nothing contradicts it. Single contributor; no fork markers.
- **License: manifest-only.** `Cargo.toml:10` declares `license = "MIT"` but `find . -iname '*license*'`
  returns nothing in the tree -- no LICENSE file. Intent is permissive but legally a declaration without a
  license text; all portables below are concept-level and `effort_for_us` values assume clean-room.
  Recorded as finding zap-coding-agent-10 (license-risk).
- Identity hygiene: no similarly-named subject in the corpus; worked only from this directory.

## What the code actually does (core surfaces read)

**Loop** (`src/session/turn.rs`, 570 LOC): classic tool-loop, `MAX_TURNS=50` (session/mod.rs:40).
Notable mechanisms, all real in code:
- Per-turn model routing by a task classifier with restore-on-exit (`routing::route_for_turn` turn.rs:68,
  session/routing.rs:12-27, task_classifier.rs:25).
- Casual-turn fast path: greeting-sized system prompt, last message only, no tools
  (`build_casual_system_prompt` context_manager.rs:10; used turn.rs:163, 300-310).
- Skill injection is projected *before* the compaction check so matched skills can't push a "75%"
  context past 100% (turn.rs:74-93), then ranked/truncated to `skill_token_budget` (turn.rs:148-158).
- Edit-ledger block injected into the system prompt, survives window eviction (turn.rs:195-206).
- Orphaned `tool_use` repair from an interrupted prior turn (turn.rs:213-230).
- Overflow-triggered compact-and-retry, stream-drop retry with exponential backoff (turn.rs:325-360).
- SLM-mode exact-repeat loop detection: `name:input` fingerprint-set equality, one-shot anti-loop
  nudge appended to the tool result (turn.rs:500-537).
- Per-turn cost + context-bar reporting (turn.rs:375-430).

**Compaction** (`src/session/history.rs`, `session/mod.rs`, `commands/session_mgmt.rs:110`): sliding
window of last 8 user turns + oversized-tool-result pruning (session/mod.rs:79-127), LLM summarization of
turns that slide off, prepended as a synthetic user/assistant ack pair (turn.rs:302-318,
mod.rs:200 `dropped_summary`), auto-compact at projected >= 90% with a 3-failure circuit breaker
(turn.rs:121), hard stop at 100% when a budget is set (turn.rs:107-119). One summarize strategy -- no tiers.

**Permissions** (`src/permission_manager.rs` 350 LOC): Auto/Ask/Deny modes; Ask prompts on 5 write tools
(permission_manager.rs:8) with session-grant "always" via *grant classes* -- edit_file/write_file/batch_edit/undo_edit
share one grant, shell is isolated (permission_manager.rs:13-21), and the invariants are unit-tested
(:189-300), including the deliberate two-layer MCP gate (quick_check allows, session layer upgrades --
documented at :229-232). Destructive substring patterns force confirmation **even in Auto mode**
(shell.rs:97-127, wired at session/tools.rs:83). Hard-block denylist guard_shell (shell.rs:14-141).
Sandbox: off (default, config/mod.rs:373) / workdir (just `current_dir`, weak, honestly labeled) /
container (`docker run --rm --network none`, project ro, tmpfs /tmp -- shell.rs:160-178). Project trust
gate for repo-local hooks + `.mcp.json` (trust.rs:1-58, tested :60+). SECURITY.md (docs/SECURITY.md:14-16)
explicitly says the denylist "is not a security boundary -- trivially bypassed". Secret-egress warning
on user messages before cloud sends (turn.rs:31-57).

**Orchestration**: `spawn_agent` subagents with depth guard (tools/agent.rs:8), background agents +
in-process scheduler (session/background_agent.rs 162 LOC, session/scheduler.rs 283 LOC), declarative
YAML workflows (workflow.rs:24-110). No durable queue, no resume journal, crash story = orphan repair +
per-turn sqlite message save (turn.rs:556-560).

**Interop**: MCP client stdio+SSE (mcp.rs:1-33) with *lazy connect*: unconnected servers surface only as
a synthetic `mcp_connect` stub tool (tools/mod.rs:221-244, tests :305-385). SDK mode: NDJSON stdin/stdout
(`--sdk --auto`, cli.rs:388-390, tests/sdk_e2e.rs:1-30). Remote control: axum webchat + public tunnel
(ngrok -> localhost.run) gated by a 18-byte CSPRNG token, ws refused without it (remote.rs:1-25). No MCP
server, no ACP, no IDE surface, no published SDK package.

**Verification**: 378 inline `#[test]`s that assert semantics (permission grant classes, trust gate,
`cache_control` marking anthropic.rs:527, MCP stub composition, loop/agent-loop tests 461 LOC
session/agent_loop_tests.rs), 2 Rust e2e files (provider_e2e.rs hits *live* provider APIs; sdk_e2e.rs
spawns the binary, all `#[ignore]`d), 14 shell e2e scripts (tests/e2e/), a real eval harness with ~15+
task JSONs and pass/token/cost reporting (evals/README.md, src/bin/evals.rs). **But CI runs none of it**:
build.yml's steps are build/package/release only (build.yml:70-96, zero hits for `cargo test` in
.github/), and security-audit.yml only runs cargo-audit + cargo-deny (:23-24). This is the single
biggest gap in the subject.

**Operability**: sqlite session store with `/sessions` browse/resume and `/branch` forking
(cli.rs:24, session_mgmt), file-level snapshots + `undo_edit` tool (snapshot.rs:18-72, tools/undo.rs),
audit log (audit.rs, turn.rs:282), raw SSE debug log path named in user-facing errors
(turn.rs:400-406: "~/.zap/llm.log"). No git-based checkpoint.

## Scores (against anchor ladder)

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | clean single-crate split, max file 1,221 LOC (tui/input.rs); loop separated from TUI via channel (turn.rs:22, tui/channel); docked: Session is a hub struct touched by ~10 submodules, session/mod.rs 737 + turn.rs 570 |
| verification | 4 | 378 semantic inline tests (permission_manager.rs:189-300) + eval harness (src/bin/evals.rs) but zero test/eval execution in CI (build.yml:70-96; grep cargo test .github/ = 0 hits); e2e needs live keys, sdk_e2e all #[ignore] (sdk_e2e.rs:6) |
| safety-enforcement | 6 | pi rung: tested Ask-mode approval + tested project-trust gate (trust.rs:23-58, tests :73+) + exemplary honesty (docs/SECURITY.md:14-16) + destructive-confirm-even-in-Auto (session/tools.rs:83); default sandbox off (config/mod.rs:373), denylist bypassable by design-admission |
| token-economy | 7 | pre-compaction projection of to-be-injected skill tokens (turn.rs:74-93); window+drop-summary (mod.rs:79-127,200); casual/SLM prompt tiers (context_manager.rs:10); Anthropic ephemeral cache marks (anthropic.rs:77-100,254-261); per-turn $ + ctx bar (turn.rs:375-430). Dock: single summarize strategy, len/4 estimates, no cache-breakpoint stability discipline |
| orchestration | 5 | spawn_agent depth guard (tools/agent.rs), scheduler.rs+background_agent.rs exist, YAML workflows (workflow.rs:24); no journal/queue/crash-recovery semantics |
| interop | 5 | MCP client + lazy mcp_connect stub (tools/mod.rs:221-244), SDK NDJSON (cli.rs:388), remote tunnel (remote.rs:1-25); no server-side MCP, no ACP, no IDE, nothing published |
| operability | 6 | sqlite resume + /branch, file snapshots/undo_edit (snapshot.rs:18-72), llm.log diagnostics named in errors (turn.rs:400-406), orphan tool_use repair (turn.rs:213-230) |
| originality | 7 | mechanisms verified in code, not just README: classifier per-turn model routing (routing.rs:12-27), casual-turn fast path with token math (turn.rs:78,163), edit-ledger injection (turn.rs:195-206), lazy MCP connect, SLM nudge ladder + fingerprint loop nudge (turn.rs:500-537), CSPRNG-token tunnel |
| durability | 3 | single contributor, no institution; version 0.15.142 and recent HEAD 2026-08-19 suggest active solo cadence, but shallow clone hides true history; docs/SECURITY.md exists (rare for solo) |
| docs-dx | 7 | honest SECURITY.md + ARCHITECTURE.md/FEATURES.md/ZAP.md, specs+plans in docs/, install.sh/ps1, agent.toml.example, evals README; docked: no CONTRIBUTING/CHANGELOG, security-review-*.md drafts published raw |

**Weighted total: 56.0 -> band C.** (9 + 6 + 6 + 7 + 5 + 5 + 6 + 7 + 1.5 + 3.5)

Strongest dimension: **token-economy** (7). Weakest: **verification** (4) -- a real, semantics-testing
corpus that CI never executes; the tests' existence is currently a promise, not a property.

Boundary check: 56.0 is >=7 from every band boundary (45/65/78/88); no within-2-pts note required.
No calibration demotions applied; not a fork, rule (a) N/A.
