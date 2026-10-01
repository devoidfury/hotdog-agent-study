# maki — T2 deep review

Subject: maki (/data/samples/agents/maki) — Rust coding agent, ratatui TUI, everything-not-the-loop implemented as embedded-Lua plugins.
License: MIT (file + Cargo.toml:6). Provenance: original per census, no fork flag; single author (tontinton). Not subject to rule (a).

## Anchor question

**Which anchor is this closer to, and why?** Pi — a single-maintainer vision harness where the Rust core keeps only the loop and everything else is an extension layer, with exemplary tested approvals but no OS sandbox underneath; maki matches pi's compaction craft and exceeds its interop (MCP + ACP + Claude-compatible headless), and trails it on cache warming, session-tree depth, and institutional durability. Placed above crush (69) on verification depth and boundary cleanliness, below pi (78.5).

## Census sanity-check (CORRECTION)

- LOC totals are right (census 204,260 ≈ cloc 204,260) but the **test split is badly wrong**: census `test_loc: 18,385` vs measured ~90k — inline `#[cfg(test)]` modules ~70.1k across 231 .rs files (brace-balanced extraction), Rust `tests/` dirs 12.7k, Lua plugin specs ~11.3k (`plugins/*/tests/spec.lua`). Real split ≈ 114k non-test / 90k test. Same failure mode as the cline/nanocoder census artifacts. Tier stays T2 (total LOC, and the surface warrants it).
- Shallow clone (`.git/shallow`, commits=1). HEAD 2026-09-27 is 2 days old and release artifacts exist for v0.5.5, so no dead/inactive claim is made in either direction; activity is asserted only from HEAD + release evidence, which is safe (recent HEAD under shallow clone proves liveness, not the reverse).
- Contributor count 1: taken with grain of salt from shallow metadata, but nothing in-repo contradicts a solo bus factor.

## Core loop (mandatory read: maki-agent/src/agent/run.rs)

Loop is a turn state machine in `Agent::run_loop` (run.rs:380) with every recovery path explicit and bounded:
- `sync_model` before compaction check so a mid-run model switch never compacts against the wrong window (run.rs:385-387,406-424).
- Overflow recovery: provider 413 triggers compaction exactly once per success streak (`MAX_OVERFLOW_RECOVERIES`, run.rs:654-666) — a second overflow returns the honest error.
- Auth errors park the run on a user-response channel with cancel race, capped attempts (run.rs:668-694).
- Stalled turn (no text, no tools): empty-marker into history + bounded nudge only when recent tool results exist (`recover_stalled_turn`, run.rs:743-758).
- max_tokens continuation capped by `max_continuation_turns` (run.rs:525-531).
- Cancellation sanitizes half-streamed history (`sanitize_cancelled_history`, run.rs:336-341) and lands streamed partial text with a cancelled-note.
- `carry_from` watermark (run.rs:280,522) marks input no turn has answered yet, so compaction can never summarize away an unanswered prompt; burst inputs each judged by hooks.
- Hooks with Verdict semantics (`agent.user_message` can drop/rewrite user input, `agent.stop` can resume a finished run with a synthetic message, capped by `MAX_STOP_CONTINUATIONS`) — run.rs:556-643.
- Every request recomputes MCP tool set (`request_tools`, run.rs:984-990) so late servers and loaded deferred tools take effect next turn.

God-file check: largest loop-side files run.rs 2636, tool_dispatch.rs 2635, permissions.rs 2260 — no 3k+ loop file; the 5.6k outlier (`maki-lua/src/runtime.rs`) is the Lua host, and the biggest UI file is 3.1k (lua_float.rs) — errata docking factor absent, but `ToolContext` is a ~30-field bag threaded everywhere (run.rs:772-804).

## Compaction (mandatory read: maki-agent/src/agent/compaction.rs)

- Trigger: gauge (provider-reported tokens with a chars/4 floor seeded pre-first-turn) vs `usable = window - reserved` (compaction.rs:394-411); reservation = buffer, floored by model `min_output`, ceilinged at 50% of window (`MAX_RESERVED_PERCENT`, compaction.rs:40) — the comments document a real llama.cpp `n_ctx 4096` compaction-death-loop this fixed.
- Protected tail is a **byte budget** (64 KiB, compaction.rs:27) walking newest-first, not a message count — comment at :23-26 documents the count-based version that let one giant MCP dump re-blow the window every retry.
- Overflow-retry ladder for the summarization request itself: strip images/thinking → collapse tool results to placeholders → drop oldest round with orphaned-tool-result cleanup (compaction.rs:460-537, 565-604).
- Layer-selectable collapse: `agent.compact.prepare` gets the host's budget picks (index/tool/input/bytes/truncated text) and returns edited indices; the host applies the edit so no layer can orphan a call or eat a user message (compaction.rs:503-563). Text shipped to Lua is capped 4 KiB per result (:44).
- Summarizer model can differ from session model via `resolve_compaction_model` + `model_policy`, billed at the summarizer's price (run.rs:852-905).
- A layer may skip an auto-compaction — but not the one the provider already forced (`CompactReason::Overflow` unskippable, compaction.rs:54-56).
- Post-compact continues with a configurable nudge that restores todo lists and prompts memory saves (compaction.rs:21); unanswered prompt substitutes for the nudge (run.rs:830-850).
- Gauge reset against the post-summary transcript with the next request's real system+tools (run.rs:891-895).

## Permissions / safety (mandatory read: maki-agent/src/permissions.rs, tool_dispatch.rs, plugins/bash)

- **Grammar-parsed bash scopes**: the bash tool declares `permission_scopes` produced by a tree-sitter-bash parse per command segment (plugins/bash/init.lua:295-315); unparseable command ⇒ single full-command scope with `force_prompt = true` (fail-closed, :301-308). "Allow always" generalizes per-segment: `git diff && rm -rf /` stores `git *` + `rm *`, never `git *`-covers-everything, with `SHELL_KEYWORDS` guarding block-openers from wildcard generalization (permissions.rs:39-43, 933-953).
- `enforce` (permissions.rs:695-770+): deny default when no user-response channel exists (`user_abort`, :745-747) — fail-closed, contrast nanocoder's fail-open shell. Re-checks the decision under the channel lock (stale-rule race), then only an answer tagged with this ask's id is consumed (`ANSWER_ID_SEPARATOR`, :34-37) — stale-answer guard. Every accept/deny emits an otel `tool_decision` with source attribution (rule/yolo/user_once/user_session/user_always/user_abort, :22-28).
- Wildcard `*` allow *and* deny are both surfaced with warnings because they bind builtin tools too (:333-351).
- Lua hooks can *escalate* a rule-allowed call into a prompt (Verdict::Ask → `force_prompt` path, tool_dispatch.rs:107-126) but can never de-escalate below rules; `enforce_permission` refuses dotted names to prevent builtin-default impersonation (tool_dispatch.rs:810-815).
- Folder-trust gates every file under `.maki/` — project config, hooks, plugins are untrusted until an explicit, terminal-confirmed grant (`maki trust add`), with per-run `--trust` and stored deny remembered (src/project_trust.rs:13-46, maki-config/src/project.rs:22-34).
- **No OS sandbox**: no seatbelt/bwrap/landlock anywhere in the tree (grep: zero hits). Bash runs via `maki.fn.jobstart`. This is the "tested approval, nothing underneath" rung; the posture is not oversold in README (sandbox claim is scoped to the interpreter: "Sandbox limited by time & memory"), but there is no SECURITY.md.
- Hook-chain deadline fail-open: agent-slot hooks race a 30s cap and on expiry the loop proceeds with `Verdict::Unchanged` (hook.rs:19-21, 95-100) — a wedged policy plugin silently stops filtering. Filed as safety-hole.
- SSRF: every Lua-side HTTP goes through a guard that vets DNS, pins the vetted IP for the actual connect, re-validates each redirect hop manually, with an explicit host-allowlist escape hatch (maki-lua/src/api/net.rs:183-187, 277, 431-433, 692; 24 tests in file).

## Token economy beyond compaction

- `index` tool: tree-sitter file skeletons (name + exact line spans) offered before reads; README quantifies +59 tok/turn vs −224 tok/turn on reads.
- `code_execution`: monty-embedded Python where every tool is an async function, so filter/transform/pipe happens outside the context window; `gather()` re-implemented to preserve sibling results on partial failure; interpreter `open()` is disabled while any plugin hooks `read` to prevent bypassing read hooks (plugins/code_execution/init.lua:20-40). Interpreter tool calls route through the same gated dispatcher (code_execution/init.lua:1-4: "sandbox and dispatch live in Rust, which exposes primitives only").
- Deferred MCP catalog: tool defs beyond a budget are hidden behind one `tool_search` tool; a *permitted* call counts as loading (definition joins next request), a denied one loads nothing (mcp/mod.rs:53, 353-391; tool_dispatch.rs:886-890).
- Skills: prompt carries only `<available_skills>` name+description list (skill_helpers.lua:20-39); bodies load on tool call; scans `.claude/skills`, `.opencode`, `.agents` too (init.lua:15-21).
- Prompt caching: two ephemeral breakpoints over the wire-tail plus system for Anthropic (anthropic/shared.rs:42, 208-227); no warming, no cache-stat telemetry surfaced like pi.
- rtk integration: if the external `rtk` binary is installed, bash commands are pre-rewritten for output compaction (plugins/bash/init.lua:105-134); config-disable documented.

## Verification

- ~90k test LOC (see census correction) that assert properties: run.rs replays scripted `StreamResponse`s through a MockProvider and asserts captured request transcripts, compaction tests cover the budget/collapse/truncation matrix including "truncate_oldest_round_preserves_text_beside_orphan" (compaction.rs:1551-1690), 46 permission tests incl. symlinked-scope resolution (permissions.rs:1291-1351), 34 dispatch tests incl. doom-loop matrix (tool_dispatch.rs:1142), 343 UI tests (app/tests.rs 7.3k LOC).
- Distinctive: Lua plugin specs (`plugins/*/tests/spec.lua`) executed *inside the embedded plugin host* from a Rust test_case harness (maki-lua/tests/spec.rs:6-29) — the extension layer is CI-tested against the real runtime.
- CI: fmt+stylua, clippy lint, nextest, cargo-machete, docs-generation drift check (`just gen-docs-check`, rust.yml:96), Windows lint+build — but the test job is ubuntu-only (rust.yml:81-96); no macOS/race matrix, no fuzzing, no in-CI model evals (benchmark reports ship with releases externally, not in CI). Verification 8-ceiling per errata, minus one notch for single-OS test execution.

## Orchestration / operability / interop

- Subagents via `task` plugin: model-tier choice per child (weak/medium/strong capped at parent), individually cancellable (subagent_cancels), each with its own navigable chat surface (`/tasks`, Ctrl-X); `batch` runs N tool calls concurrently with per-child failure markers; mailbox + interrupt queue + `/compact` mid-run (run.rs:914-963). No durable queue, no crash-resume of an in-flight run, no budgets beyond max_turns/max_turn_output. Doom-loop breaker (threshold 3, identical name+input hash) skips the call with an injected error rather than aborting the run (tool_dispatch.rs:25, 69-92, 916-946) — tested.
- Sessions: append-only JSONL with epoch cursors, per-session file claims/locks so two processes cannot append to one log, failed append rolls back to last full record (maki-storage/sessions.rs:1-8, 136-236). Resume, chat rewind (Escape-Escape; README honest that code rewind doesn't exist). ChildGuard reaps spawned processes with bounded kill (child_guard.rs). Panic handling via color-eyre + crash logs (maki-storage/log.rs). No git checkpoint/stash equivalent (cline 8-rung item).
- Interop: full ACP server crate with elicitation + permission translation (maki-acp/src/{server,elicitation,permissions}.rs, docs at site/docs/content/acp); MCP client with OAuth (maki-agent/src/mcp/oauth/); headless `--print --output-format stream-json` **deliberately Claude-Code-output-compatible** plus stream-json input SDK mode (src/cli.rs:66-69, src/sdk_mode.rs); OTel export in "same format as Claude Code" (README, maki-otel cost_usd metrics emit.rs:53-133). No MCP-server mode, no published SDK, no IDE plugin beyond ACP.

## Originality (verified in code, not README)

Everything-is-a-Lua-plugin (all 21 builtin tools incl. bash are plugins over a Neovim-shaped API with libuv/treesitter/async, maki-lua 40.3k LOC), tree-sitter-derived permission scopes with safe generalization, monty code-mode with hook-bypass-proof `open()`, layer-selectable compaction collapse, doom-loop dedupe-skip, budget-tail compaction. Several of these are corpus-first or corpus-best.

## Durability

One maintainer, no SECURITY.md, no governance doc; but MIT, active HEAD, release matrix (linux/macos/windows, install.sh/ps1, nix flake), benchmark-driven release culture. Bus factor 1 caps this at nanocoder's rung, slightly above for release infrastructure.

## Scores (against frozen anchors)

| dimension | score | best evidence |
|---|---|---|
| architecture | 8.0 | run.rs:380-536 turn machine; 17-crate split; no >3k loop file; ToolContext 30-field bag (run.rs:772-804) and 2.6k dispatch/runtime files keep it under 9 |
| verification | 7.5 | ~90k test LOC, MockProvider loop tests (run.rs:1042-1130), in-host Lua specs (maki-lua/tests/spec.rs:6-29); nextest ubuntu-only (rust.yml:81-96), no evals/fuzz (8-ceiling minus) |
| safety-enforcement | 6.5 | tested binding approvals (permissions.rs:695-770, fail-closed :745) + grammar scopes (plugins/bash/init.lua:295-315) + trust gate; nothing underneath at OS level (zero seatbelt/bwrap hits) |
| token-economy | 8.0 | budget tail (compaction.rs:27), reservation ceilings (:40), overflow ladder, index/code_execution/rtk tool-side economy; cache marking only (shared.rs:42), no warming → under pi's 8 on cache craft, at it overall |
| orchestration | 7.0 | task/batch/mailbox/doom-breaker + JSONL claims; no durable queue, budgets, or crash-resume of live runs |
| interop | 8.0 | ACP server crate + MCP w/ OAuth + Claude-compatible stream-json (src/cli.rs:66-69) + rival-harness skill dirs (plugins/skill/init.lua:15-21); no MCP-server/SDK/IDE surface |
| operability | 7.5 | epoch-cursor append-only journals (sessions.rs:68-236), rewind (chat only, honestly labeled), reauth/stall recovery, ChildGuard; no git checkpoints |
| originality | 8.0 | plugin-host-everything, grammar scopes, monty code-mode, collapse-selection hook — all verified in code |
| durability | 4.5 | solo bus factor, no SECURITY.md; MIT, active, real release infra |
| docs-dx | 8.0 | 21-section docs tree incl. lua-api/token-economy/folder-trust; CI-drift-gated docgen (rust.yml:96); install scripts |

**Weighted total: 74.5 → band B** (upper B). Strongest: token-economy/interop/originality (8) — name one: **originality** (the only 8s carrying mechanisms no other anchor has). Weakest: **durability** (4.5).

## Calibration notes

- No rule (a) (not a fork). No archived cap. No demotion applied.
- Census correction recorded: test_loc ~90k not 18,385.
- Not a safety S-band candidate: no kernel enforcement anywhere.
