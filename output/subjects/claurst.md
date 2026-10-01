# claurst — T2 deep review

Subject: /data/samples/agents/claurst · Rust · GPL-3.0 (LICENSE.md) · 12 crates under `src-rust/crates/`
Census row: non_test 121,972 / test 20,932 · head 2026-09-02 · shallow:true · provenance `divergent-fork:claude-code`

## Anchor question (one sentence)

Closest anchor: **crush** -- both are solo-vision Rust terminal agents with a solid separated loop, real property-pinning tests on a 3-OS matrix, tested-approval-with-nothing-underneath safety, and one standout original mechanism; claurst carries more feature surface than crush but is dragged below it by a misleading sandbox toggle and an impersonation layer crush never needed.

## Census sanity

- LOC: `find src-rust -name '*.rs' | wc -l` = 294 files, 148,186 total lines; crate sums ≈121k matches non_test. Test counting: 1,800 `#[test]`/`#[tokio::test]` fns, mostly inline `mod tests` (integration `tests/` dirs are only 1,366 LOC). Census test_loc 20,932 is plausible for inline+integration test code. No correction.
- Shallow clone confirmed (`.git/shallow`; single visible commit `b0637c9`, PR #403 from an external contributor). Census `commits: 1, contributors: 1` is a shallow artifact. Remote-verified: created 2026-03-31, pushed_at 2026-09-02, not archived, 10,310 stars, 7,725 forks, 45 open issues (GitHub API, 2026-09-29). Active, young (~6 months), single account identity (Kuberwastaken) -- true bus factor is ~1 but PR flow exists.
- Provenance correction: not a "fork" in any code-lineage sense -- a behavioral reimplementation (different language, independent implementation) derived from a spec produced off the leaked Claude Code npm sourcemap. Upstream claude-code is NOT in the corpus, so calibration rule (a) cannot be checked locally: evidence-limited.

## Provenance & license (in-repo evidence only)

- README.md:20,210-218: self-declared "clean-room Rust reimplementation"; cites Phoenix v. IBM / Baker v. Selden; links the author's blog "Claude Code's Entire Source Code Got Leaked via a Sourcemap in npm, Let's Talk About it" as "the original breakdown of the findings."
- spec/ (27,144 LOC of md) is a catalog of the proprietary TS tree: spec/00_overview.md:3-8 names `X:\Bigger-Projects\Claude-Code` with per-file LOC ("main.tsx 4,683 lines"); spec/12_constants_types.md:592,735 reproduces the verbatim identity string "You are Claude Code, Anthropic's official CLI for Claude."
- The clean-room framing fails its own prerequisites: single-author pipeline (same person's AI analyzed the leak and same repo implemented it; no clean-team firewall, no "never referencing" verifiable), and the implementation comments openly port the source line-by-line in intent: query/src/compact.rs:314 "Compaction prompt (matches TypeScript prompt.ts)", :1368 "Mirrors the TypeScript stripImages helper"; core/src/memdir.rs:3-6 maps four TS modules 1:1; 134 hits for TS-mirroring comments across the crates. Same-shape precedent in corpus: claw-code-agent `ported-proprietary-source`.
- GPL-3.0 file present. Under protocol license discipline: concepts-only, effort_for_us assumes clean-room, no code-adjacent portables recommended.

## Core loop (line-level read)

`query/src/lib.rs:381 run_query_loop` -- the loop is the strongest architecture signal:
- Loop owns the cancel token and rebinds the tool context to it (:398-402, issue #218 fix, commented with the bug class).
- Per-turn: tool-result budget truncation (:600-615), history sanitization at a single choke point that heals broken tool_use/tool_result pairing before BOTH provider paths (:620-628, issue #229), max-steps graceful degradation -- final tool-less summary turn with an anti-recursion guard (:425-430, :631-640), effort escalation for the `ultracode` keyword per-turn (:640-647), todo-completion nudge injected into the system prompt after turn 2 (:689-697), 45s stream-stall detection with bounded retries (:1026-1045, :1110-1113), max_tokens recovery counter (:405-407).
- Cancellation: `runner/tools.rs run_tool_batch` abandons in-flight futures and synthesizes cancelled tool_results so history stays provider-valid (runner/tools.rs:82-107).
- No repeated-tool-call loop breaker (grep: none); crush is the anchor at loop-detection and claurst lacks it.
- Boundaries: 12 crates with real separation (api = providers/wire, query = loop, tools = tool impls, tui/commands/cli = surfaces, acp/mcp/plugins/bridge/buddy). Docked for `core/src/lib.rs` 5,318 LOC module-fest (config+permissions+hooks+cost in one file) and TUI mass: 46,993 LOC across tui, `prompt_input.rs` 5,084 (over the errata's 5k docking bar), `render.rs` 3,913.

## Compaction / token economy (line-level read)

`query/src/compact.rs`:
- Two tiers: micro-compact at 75% window with LLM summary of the head (keep last 10 msgs, :245-316) and auto-compact at 90% (AUTOCOMPACT_TRIGGER_FRACTION :44) with 13k buffer; circuit breaker after 3 consecutive summarizer failures (:57-86); 80/95% warning states (:60-61).
- Trigger sizing prefers the provider-reported usage of the last real turn over char/4 estimates, with the correct comment about cache-read undercounting, explicitly crediting pi's estimator (:883-899) -- measurement quality near pi's rung.
- Window resolution via a models.dev registry with a plausible-value guard against 4096 placeholders (:827-873) -- good multi-provider hygiene.
- Keep-split snaps to tool_use/tool_result pairing boundaries (:1138-1164); summary carries a "Files touched" manifest that is parsed and re-merged across successive compactions (:566-751) so file context survives cascaded compactions.
- Fatal gap: `CacheControl::ephemeral` is modeled (api/src/lib.rs:250-262) but grep across all crates: zero construction sites -- every request sets cache_control None (:604). Anthropic prompt caching is never enabled; every turn re-bills full context. The token-economy ceiling here is cost blindness at the billing layer, not the window layer.

## Permissions / safety enforcement (line-level read)

- Engine: core/src/lib.rs:2522+ permission rules (deny-over-allow, persistent-over-session, glob patterns :2588-2608), mode ladder with ordered evaluate() Bypass → deny → allow → AcceptEdits(Edit only) → Plan(reads-only) → default-by-danger-level (:2795-2890). Reads inside workspace roots auto-allow; MCP resource reads always ask (:2860-2870).
- Central backstop: query/src/runner/tools.rs:65-72 gates any tool with `self_gates()==false` at a gated level before execute; tools self-declare gating to avoid double-prompt (tools/src/lib.rs:520-532). The contract is pinned by tests (query/src/lib.rs:2504-2600: denied tool must not run; self-gating must not double-prompt; read-only never gated). This is a genuine tested chokepoint.
- Bash risk classifier: core/src/bash_classifier.rs Safe→Critical ladder with wrapper stripping (sudo/env/nohup recursion :43-56) and pipe-to-shell detection (:68-88). Text heuristics, not grammar-parsed (compare corpus `grammar-parsed-bash-policy`), but structured, ordered, and used for auto-approval decisions.
- MCP trust (exemplary): core/src/mcp_trust.rs:1-27 -- project-origin MCP servers never launch without user approval; allowlist lives at `~/.claurst/mcp_trust.json` keyed by root+fingerprint so "a repo can never grant itself trust"; settings merge hard-blocks project files from flipping `trustProjectMcpServers`, bypass-prompt-skip, or allowed bash prefixes (core/src/lib.rs:1992-2003, with SECURITY comments).
- Safety hole 1 -- phantom sandbox: `/sandbox-toggle` (commands/src/sandbox.rs:14-30) promises "shell commands run in an isolated environment"; `sandbox_mode` is written to UI settings (:137) and read only by its own status display (:44). Zero enforcement sites anywhere in tools/api/pty_bash -- bash spawns via plain `Command::new("bash")` / PTY (tools/src/pty_bash.rs:163-181,393). Misleading by design, worse than absent.
- Safety hole 2 -- project-settings self-grant: the same merge that protects the MCP trust flag lets a cloned repo's `.claurst/settings.json` inject `hooks` (core/src/lib.rs:1962) -- arbitrary shell commands run on PreToolUse/Stop/UserPromptSubmit via `sh -c` (:3986) -- and extend `permission_rules` (:2005). Open a hostile repo, get code execution or pre-allowed tool rules. The codebase demonstrates it understands this exact attack class three lines away and didn't apply it here.
- No OS-level sandbox of any kind (no seatbelt/bwrap/landlock anywhere; grep). Approvals are the entire enforcement story, and the product's own UI claims more than exists.

## Verification

- 1,800 test fns; tests pin properties, not existence: backstop contract a/b/c (query/src/lib.rs:2504-2600), compaction thresholds with boundary tests at 80/95/90% (compact.rs:1753-1830), circuit-breaker open/reset (:1812-1830), settings-merge origin tagging for MCP (:4649+), bash-prefix bypass expectations (core/src/lib.rs:4628).
- CI (.github/workflows/ci.yml): cargo test --workspace --locked on ubuntu/windows/macos with `--test-threads=1` (documented global-state races -- honest but a design smell, :60-66); clippy `-D warnings` enforced; rustfmt advisory-by-design with a stated reason (:68-76). Separate vscode-extension CI, npm publish, release workflows.
- Ceiling: no model evals, no fuzzing anywhere (grep none) -- same ceiling as all anchors at 8, but the corps here is thinner and mostly unit-level; integration dirs are 1,366 LOC. 7, not 8: cline/pi/codex corps are 1-2 orders larger and drive full flows against scripted servers; claurst's loop itself is barely integration-tested end-to-end (no faux-provider harness found).

## Orchestration / operability / interop

- Subagents (tools/agent_tool.rs), swarms as a tool (tools/team_tool.rs TeamCreate → N parallel run_agent dispatch), coordinator mode with tool filtering (query/coordinator.rs:158-199), managed-orchestrator mode (query/managed_orchestrator.rs), multi-turn goals with budget guards (`/goal`, query/goal_loop.rs:79-99: complete/paused/budget_limited stop reasons), cron scheduler (query/cron_scheduler.rs), command queue (query/command_queue.rs), "ultracode" fan-out procedure (core/src/effort.rs:281+ ULTRACODE_PROCEDURE: plan→delegate→integrate→verify across Agent/TeamCreate/TaskCreate).
- Sessions: JSONL transcript with UUID chains, tombstones, leaf entries, `truncate_after` + non-destructive fork variant (core/src/session_storage.rs:475-532), `--resume [id|last]` (cli/src/main.rs:141-143), TUI rewind flow (tui/src/app/prompt.rs:11), shadow-git per-turn file-change snapshots (query/src/lib.rs:441-455, core/snapshot/), optional sqlite storage, /share gist export.
- Interop: full ACP server crate (initialize/session/new/prompt/cancel, request_permission routed to editor; README:155-180) + registry manifest template + own VS Code extension that spawns `claurst acp` (editors/vscode/README.md:5-7) with extension CI; MCP client (mcp/src/lib.rs client module, oauth-capable); headless `claurst -p`; npm/bun binary-distribution wrapper (npm/install.js). MCP server-mode: no. No published SDK.
- Crash recovery: resume journals exist; crash-time atomicity of transcripts/snapshots unproven (no tests for torn-write recovery found).

## Originality (verified in code)

- `ultracode` keyword effort tier with a delegated-workflow addendum, composable with /goal (core/src/effort.rs:273-405, keywords.rs:72). No other corpus subject has this shape.
- Buddy companion (crates/buddy, 1,118 LOC Tamagotchi system) + voice input (core/src/voice.rs) -- surface originality, thin mechanism.
- Files-touched manifest surviving cascaded compaction (compact.rs:566-751) -- genuinely useful original mechanism.
- OAuth stealth layer (below) is also "original" in mechanism, but it is a liability, not credit; originality is scored on constructive ideas.

## Boundary-risk / negative originality

- core/src/oauth_config.rs:51-135 + api/src/lib.rs:590-656: a documented stealth-impersonation stack -- `claude-cli/<ver>` user-agent, extracted proprietary billing salt `59cf53e54c78` + reconstructed `cc_version` client-hash, the verbatim Claude Code system-prompt prefix injected as system[1], own attribution stripped from requests (:618-634), and a wreq/BoringSSL client chosen because its "TLS fingerprint matches Bun (the official client)" (:658). Comment at :170 states the purpose outright: so Anthropic's auth server accepts Pro/Max tokens "through Claurst," with a live-verified note that it draws interactive subscription quota (:181-190). This is deliberate provider-gate evasion: a ToS/anti-circumvention hazard that also makes the "clean-room" claim untenable (you do not need a leaked bundle's salt and fingerprint to reimplement behavior).

## Scores (weighted total 65.0, band B)

| dim | score | best evidence |
|---|---|---|
| architecture 7 | loop ownership + 12-crate split vs core/lib.rs 5,318 + tui 5,084 file | query/src/lib.rs:381,620-628; core/src/lib.rs; tui/src/prompt_input.rs |
| verification 7 | property tests + 3-OS CI + clippy gate; no evals/fuzz, thin E2E | query/src/lib.rs:2504-2600; ci.yml:31,60-68 |
| safety-enforcement 4 | tested backstop + exemplary MCP trust, but phantom sandbox, project-settings self-grant, no OS sandbox, impersonation layer | runner/tools.rs:65-72; mcp_trust.rs:1-27 vs sandbox.rs:14-30, core/src/lib.rs:1962,2005 |
| token-economy 6 | usage-truthed trigger + 2 tiers + breaker, but zero prompt-cache marking | compact.rs:44,891-899 vs api/src/lib.rs:250-262 (never called) |
| orchestration 7 | agents/teams/goals-with-budgets/cron/queue; crash semantics unproven | team_tool.rs:1-17; goal_loop.rs:79-99 |
| interop 7 | ACP server + own VS Code ext via ACP + MCP client + npm dist; no MCP server, no SDK | editors/vscode/README.md:5-7; README.md:155-180 |
| operability 7 | resume/fork/rewind/tombstones + shadow-git checkpoints; no crash-recovery evidence | session_storage.rs:475-532; query/src/lib.rs:441-455 |
| originality 7 | ultracode procedure, compaction file-manifest, buddy/voice | effort.rs:281-405; compact.rs:566-751 |
| durability 5 | 6 months old, active (pushed 2026-09-02 remote-verified), 10.3k stars, but bus factor 1, no SECURITY.md, legal cloud | GitHub API 2026-09-29; repo (no SECURITY.md) |
| docs-dx 7 | 14 docs/ 7,929 lines, installers, ACP docs; docked for sandbox claim contradicting code | docs/; commands/src/sandbox.rs:18-30 |

Strongest dimension: architecture (tied with verification/orchestration/interop/operability/originality at 7; architecture carries the weight). Weakest: safety-enforcement (4).

## Calibration notes

- Rule (a): upstream (claude-code) not in corpus → not checkable locally; recorded as evidence-limited. Provenance corrected from `divergent-fork:claude-code` to reimplementation-from-leaked-source-spec; synthesis should treat lineage as `ported-proprietary-source`-family, not fork-family.
- Boundary: total 65.0 sits exactly on the B/C boundary; safety-enforcement is the swing dimension (a phantom-sandbox reading of "present but misleading" would pull sub-65 territory if weighted harder).
- License discipline applied: findings are concepts-only; no code-adjacent portables recommended; effort_for_us assumes clean-room rework.
