# coro-code — T1 review

Subject: /data/samples/agents/coro-code (Rust, workspace `core` + `cli`, v0.0.8, 19,310 .rs LOC total)
Census row: non_test_loc 16,654 | test_loc 0 (WRONG, see corrections) | contributors 1 | commits 1 (shallow) | head 2025-10-30 | MIT OR Apache-2.0 (manifest) | T1

## Anchor question

Closer to **codel** than to nanocoder: coro has a real loop, CI and 93 tests that codel lacks, but it sits below nanocoder on every enforcement and operability axis (nanocoder at least ships an opt-in jail; coro's bash is explicitly opted *out* of confirmation, and there is no sandbox, resume surface, checkpoint, or loop-detection anywhere). Lineage shape (single loop + tool registry + config system, honest Rust rewriter) echoes the in-corpus trae-agent (44.0 D), which this supersedes by the same README claim (README.md:26 "Formerly known as Trae Agent Rust").

## Provenance & activity

- Census says original; no fork evidence in tree. README.md:26 claims lineage "Formerly known as Trae Agent Rust ... remains compatible with the original tool spec"; the trae tool name `str_replace_based_edit_tool` is registered verbatim (cli/src/tools/registry.rs:86). Distinct codebase/language from bytedance/trae-agent in-corpus -- treat as divergent rewrite, rule (a) n/a.
- Shallow clone (.git/shallow) => commits:1/contributors:1 are artifacts. Remote-verified (GitHub API): pushed_at 2025-10-30T09:34:18Z, created 2025-08-13, archived:false, 369 stars, 24 forks, 13 open issues, license:null. Quiet ~11 months at review -- under the 12-month dead bar, so no rule-(b) cap; but the HEAD commit being an emergency guardrail removal left standing 11 months is itself a governance signal (durability 2).
- HEAD commit (only local commit): 679c57af "temporarily disable the access permission system for the bash tool" (2025-10-30).

## Census corrections

- **test_loc 0 -> ~2,418.** Colocated `#[cfg(test)]` in 25 files, 93 `#[test]`/`#[tokio::test]` fns (e.g. core/src/agent/tokens/conversation_manager.rs:618, cli/src/interactive/file_search/tests.rs). Census glob missed colocated Rust tests -- same failure class as the crush/codel census artifacts.
- License row says "MIT OR Apache-2.0 (manifest)"; confirmed manifest (Cargo.toml:10), but **no LICENSE file exists anywhere in the tree** and GitHub reports license:null. Manifest-only license.

## Core loop (read: core/src/agent/core.rs, 1,297 LOC)

- `execute_task_with_context` (core.rs:726+): while step < max_steps { cancel check -> apply_intelligent_compression -> tokio::select!{ cancelled | execute_step } }. One LLM call per step; tool results appended, next step feeds them back (core.rs:585-590 comment). Termination only via `task_done` tool (core.rs:562-564). No loop detection.
- Cancellation is real and threaded: AbortController/AbortRegistration pair (core.rs:32-34, 291-302), raced at step granularity; incomplete tool calls get synthetic error tool-results on restart (core.rs:826-865) -- conversation-validity hygiene, nice.
- Confirmation seam in loop (core.rs:424-480): asks `requires_confirmation()`, builds ConfirmationRequest, **fails closed** (approved:false on fetch failure, core.rs:449-453). Structurally sound -- see safety for why it is dead.
- Smell: `new_with_llm_config` (core.rs:41-97) and `new_with_output_and_registry` (core.rs:176-232) are ~100 duplicated LOC each including the identical provider-match with GoogleAI/Custom returning `NotInitialized` "TODO" (core.rs:57-58, 192-193) -- two advertised protocols are unimplemented error stubs.

## Compaction (read: core/src/agent/tokens/conversation_manager.rs, 724 LOC)

- Three tiers Light/Medium/Heavy (conversation_manager.rs:16-26) triggered at usage ratios 0.7/0.8/0.9 with targets 0.6/0.5/0.3 (:88-89). Light truncates tool results over a 2,000-token budget (:220-231); Medium LLM-summarizes the older segment into a system message preserving recent pairs (:233-283); Heavy keeps system + recent pairs only (:285-317). `split_preserving_tool_pairs` (:582) keeps tool_use/tool_result pairs across the cut -- tool-call-aware splitting, tested via MockLlmClient (:672-712).
- **Trigger binds the wrong budget**: `ConversationManager::new(llm_config.params.max_tokens.unwrap_or(8192))` (core.rs:72, :204) -- params.max_tokens is the *completion* budget, not the context window. For a gpt-4o-class model, compression fires at ~5.7k estimated context tokens; the agent will live in permanent Medium/Heavy compaction on any real task.
- Token estimation is a char-class heuristic with CJK/ASCII/other accounting (calculator.rs:59-70) -- better than chars/4 for CJK, but never calibrated against provider usage; provider usage is accumulated only for display (core.rs:341-355). No prompt-cache discipline of any kind (zero cache refs in core/src/llm).

## Permissions / safety (read: cli/src/tools/bash.rs, core confirmation seam, handlers)

- HEAD commit removed the bash permission system; bash.rs:466-467 now reads `fn requires_confirmation(&self) -> bool { false // Bash commands can be dangerous }` -- the comment says exactly what the code no longer does.
- No tool in the tree returns true (grep: only bash.rs:466 and status_report.rs:96 override, both false); base default false (core/src/tools/base.rs:25-27). The fail-closed confirmation plumbing in the loop and both output handlers (cli/src/output/cli_handler.rs:269-301 stdin y/N, interactive delegates :271-278) is unreachable in practice.
- No sandbox, no allow/deny list, no path containment, no project trust, no SECURITY.md (grep permission|sandbox|deny across cli+core: nothing binding). Bash executes via persistent shell with full user privileges. This is the codel-2 "present but broken" rung, arguably worse in effect than honest absence, though coro at least does not market safety it lacks.

## Verification

- CI is real and green-shaped: test.yml runs cargo check + clippy + `cargo test --all-targets` on every PR and on release (release.yml:11-12 calls it). One job, ubuntu only; no matrix, no evals, no fuzzing.
- Test quality is bimodal: file_search/input_history/text_utils have genuine behavior specs (tests.rs 7 cases over an engine with cache + git integration), compression has mock-driven property-ish cases (:672-712). The core lanes are existence checks: core.rs:1053-1100 tests are "system prompt config serializes"; registry tests assert tools *can be created* (cli/src/tools/registry.rs:80-124). Zero tests of the loop (execute_step / execute_task_with_context), zero approval-flow tests (nothing approves).
- `coro test` subcommand is a smoke script printing checkmarks (cli/src/commands/test.rs:5-30), not a suite.

## Orchestration / operability / interop

- Orchestration: single loop + max_steps; trajectory journal exists as a library (trajectory/recorder.rs auto_save :100-101) but the CLI never wires it (see findings); no subagents, queues, budgets, resume journal.
- Operability: solid single-session TUI (iocraft app, input history, fuzzy file search with cache), env+file config with env override (cli/src/config/loader.rs), token usage events. **`--trajectory-file` logs "📊 Trajectory saved to:" without ever saving** (run.rs:94-96; recorder constructed fileless at run.rs:57, the `with_file` ctor at recorder.rs:70 is unused). `--must-patch` writes a literal placeholder file "# Changes would be recorded here" (run.rs:85-90). Core export/restore context API exists (core.rs:99-158, state.rs:17) but no CLI surface consumes it -- no resume from shell.
- Interop: MCP via a custom stdio JSON-RPC client wrapped in a single `mcp_tool` (core/src/tools/builtin/mcp.rs, 555 LOC, hand-rolled process framing :30-60); coro-core is published to crates.io as an embeddable library with examples/ (core/examples/custom_system_prompt.rs) -- library-first is a real differentiator at this size. No ACP, no IDE surface, no machine-readable headless event output (headless mode is human text logging).

## Originality

- CKG tool: tree-sitter (8 languages) -> sqlite code knowledge graph as a single tool file with factory registration (cli/src/tools/ckg.rs:1-50, rusqlite bundled cli/Cargo.toml:63). Distinctive in-corpus; depth as a *retrieval* story unproven (no tests for ckg, no loop integration beyond registration).
- CJK-aware token estimator is a genuinely useful wrinkle for the CJK-user segment.
- Everything else (sequentialthinking, task_done, str_replace edit) is trae/claude-spec porting; loop and compaction are standard designs executed plainly.

## Dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | clean core/cli library split + tool factory registry (cli/src/tools/registry.rs:32-86), loop w/ cancel + conversation-validity repair (core.rs:826-865); docked: 200 LOC duplicated constructors (core.rs:41-97 vs :176-232), provider stubs return errors (:57-58) |
| verification | 4 | CI check+clippy+test on PR (test.yml:22-33); 93 colocated tests, strong in utils (file_search/tests.rs), but core loop untested and several core "tests" are existence/serialization checks (core.rs:1053-1100, registry.rs:80-124) |
| safety_enforcement | 2 | bash confirmation disabled at HEAD (bash.rs:466-467, commit 679c57af); no tool enables confirmation; no sandbox/allowlist/containment anywhere; fail-closed seam exists but is dead code (core.rs:449-453) |
| token_economy | 6 | 3-tier compaction with tool-pair-preserving split + LLM summaries, tested (conversation_manager.rs:88-89, 233-317, 582, 672-712); docked: trigger bound to completion budget not context window (core.rs:72), heuristic-only estimator never calibrated (calculator.rs:59-70), zero cache discipline |
| orchestration | 3 | max_steps + abort only; trajectory recorder never saved by CLI (run.rs:57 vs :94-96); no subagents/queues/loop-detection/resume |
| interop | 4 | MCP stdio client tool (mcp.rs:1-60), headless single-task mode, coro-core published as library w/ examples; no ACP/IDE/SDK/JSON event contract |
| operability | 4 | good TUI + config layering + live token events; no CLI resume/snapshot surface despite core API (core.rs:99-158), trajectory flag phantom, must_patch placeholder (run.rs:85-96) |
| originality | 5 | sqlite+tree-sitter CKG tool (ckg.rs:1-30), CJK-aware estimator (calculator.rs:59-70); rest is spec porting |
| durability | 2 | solo author, ~11mo quiet (remote-verified pushed_at 2025-10-30, not archived), no tags, no LICENSE file, guardrail removed "temporarily" as HEAD and left |
| docs_dx | 4 | bilingual README w/ quickstart + config (README.md), `coro tools` listing; no docs/ tree, no SECURITY.md; GoogleAI/Azure claims in config vs TODO stubs (core.rs:57-58) |

Weighted: (6·15 + 4·15 + 2·10 + 6·10 + 3·10 + 4·10 + 4·10 + 5·10 + 2·5 + 4·5)/10 = **42.0 -> D**.

Strongest dimension: token_economy (tied 6 with architecture; the tiered compaction is the most defended part of the codebase).
Weakest dimension: safety_enforcement (2).

## Calibration notes

- Distribution: consistent with in-corpus trae-agent at 44.0 (D); coro is that shape plus working CI plus tiered compaction, minus enforcement, minus ByteDance backing, minus session depth. No rule (a)/(b) demotions. 42.0 is 3 below the 45 C/D cut; the nearest sensitivities would be safety 2->3 (arguing honest-absence over codel's misleading text) or verification 4->5 (file_search suite is real behavior testing) -- either alone keeps it under the cut.
- Shallow-clone hygiene followed: no dead claim made; activity remote-verified.
