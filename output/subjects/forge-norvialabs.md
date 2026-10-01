# forge-norvialabs (Norvia Labs "forge") - T2 deep review

Rust workspace, 19 crates (`crates/`, Cargo.toml:3-22), ~158.3k code LOC by cloc / 188k raw wc. MIT (Cargo.toml:11 LICENSE MIT). Version 0.1.0-beta.10. Remote NorviaLabs/forge, shallow clone, single visible commit "Implement open issue improvements (#810)".

**Identity note:** distinct from the other corpus subject `forge` (manifest `forge-code-evals`, upstream antinomyhq/forge). Different structure entirely (this is a 19-crate Rust workspace with a TUI-first design); name collision only. Provenance flagged for synthesis rule (a).

**Anchor question (one sentence):** Closer to **codex** than any other anchor -- the whole safety architecture (kernel confinement at spawn as the real boundary, separate host-level egress control, refusal to run unconfined, crate-split enforcement vs. decision planes, sandbox scope "matching Codex's workspace-write" stated at crates/forge-tools/src/sandbox.rs:17) is codex's philosophy in a smaller single-team package -- though the total lands in the cline/crush B-band, not at codex's 88.5.

## Core loop (mandatory read)

- `crates/forge-core/src/turn.rs:16-57` -- the multi-step coordinator is 57 lines: loop over `run_model_step` -> `apply_model_response` -> Done/Hitl/Continue/YieldToQueue. Failed-lifecycle re-entry handled explicitly (turn.rs:24-29) with a comment explaining the wedge it prevents. Queue handoff mirrors the TUI path (turn.rs:41-54).
- Session machinery split by concern: `session/turn_ops.rs` (832), `session/tools.rs` (1668), `session/approval.rs` (214), `session/compaction.rs` (386), `session/create.rs` (530, incl. `fork()` at :372). `turn_state.rs`, `lifecycle.rs` (transition reasons), `completion.rs` (evidence-based completion: `ExecutionEvidence`/`EvidenceEntry` at completion.rs:125-237, `dedup_keep_last` helpers.rs:248, `looks_like_dangling_tool_call` helpers.rs:279 to stop text-emitted tool calls being scored as no-op success).
- Lifecycle: `TaskLifecycle` Waiting/Working/Failed with `TransitionReason`; HITL stop at `turn.rs:20-22`.
- Residual god files are in the product plane, not the loop: `forge-session/src/supervisor.rs` **7395 LOC** (multi-repo session actors, goal evaluation, worktree ownership, all supervisor commands), `forge-tui/src/overlays.rs` 6558, `conversation.rs` 6551, `forge-transcript/src/lib.rs` 6016. Per the anchor errata, a >5k product-surface file is a docking factor: architecture capped at 8.

## Compaction / token economy (mandatory read)

- `forge-context/src/compaction/policy.rs` -- pure arithmetic, fully unit-tested: trigger = min(0.85*window, usable) AND looks ahead one turn (`should_compact` :105-106 includes `expected_turn_tokens`) so compaction runs before the overflowing request, not after; post-compaction runway gate at 40% to justify "breaking the cached prefix" (:14-16); tail clamp `window*0.12 in [16k,64k]`, max window/3 (:116-122).
- `forge-context/src/compaction/engine.rs:1-5` -- candidate construction is transactional and pure: "nothing here mutates a session"; validation errors include `MissingProtectedFacts`, `NotSmaller`, `StillOversized`, `InvalidStructure` (:35-47).
- `forge-context/src/compaction/facts.rs:1-16` -- protected facts (user constraints, corrections, explicit decisions, irreversible actions) extracted only from user-authored text, fed to the compaction prompt as must-keep material, and **checked against the checkpoint before install**; compaction is rejected if every user constraint was dropped.
- `forge-core/src/session/compaction.rs:7-18` -- BEGIN/COMMIT/ROLLBACK transaction diagram; provider call detached into `PendingContextCompaction` (front-ends run it off the event loop, :38-72); commit journals + opens a new cache epoch (:282-311).
- Cache discipline: `cache_epoch` counter bumped on compaction/transport change (`turn_ops.rs:713-800`), prompt-wire snapshots with `common_prefix_len` diffing and "CACHE PREFIX INVALIDATED" diagnostics naming the diverging part (:744-770), gated off the production hot path unless DEBUG (:726-733). `forge-model/src/prompt_cache.rs:34-47` normalizes Anthropic-vs-OpenAI cached-token semantics so `prompt_tokens` means one quantity; GPT-5.6 `cache_write_tokens` handled (:20-29).
- Lazy loading: MCP tool schemas deferred behind `SearchToolsTool` once they exceed 10% of usable window (`policy.rs:38-44`, `turn_ops.rs:113-134`, `forge-mcp/src/lib.rs:507-588`); Agent Skills standard with discovery-in-prompt / load-on-demand two-stage (`forge-tools/src/skills.rs:1-35`).

## Permission / sandbox code (mandatory read)

- Two-plane separation enforced by the dependency graph, stated at `forge-tools/src/sandbox.rs:1-15`: governance cannot confine (depends on forge-types only), tools cannot reason about approval. "A misclassification on the decision side costs a spurious prompt or a missing one, not a breach."
- Enforcement: Seatbelt (macOS, profile generated at `sandbox.rs:810`) and bubblewrap (Linux/WSL2, invocation at `sandbox.rs:1104-1304` with order-is-the-security-property comments at :1119-1131). Scope = workspace-write (sandbox.rs:17-37): `.git`/`.forge` read-only inside the workspace so a confined process cannot widen its own permissions; home data masked. Confinement at spawn only, no retrofit (:15-16).
- **Refuse-to-launch**: `availability()` (`sandbox.rs:342-361`) treats macOS `sandbox_apply` failure (nested confinement) via a cached probe (:369-381); bwrap/socat resolved by absolute path only, explicitly so the agent-influenced PATH cannot move the boundary (:389-393). SECURITY.md: "On a host where the OS cannot confine, Forge does not start." Shell is therefore *not* HITL-gated by default -- `governance/src/lib.rs:47-50` states the rationale; `default_hitl_tools() = ["mcp:*"]` (pattern.rs:376-378). Windows is WSL2-only, documented as such (README.md:85-86, 395).
- Egress: per-session proxy with **deny-by-default**, nothing pre-allowed -- former ecosystem hosts (crates.io, npmjs, pypi, github) asserted 403 in tests (`forge-core/src/permission.rs:143-167`); only *personal* `permissions.toml` host rules open it, repo-committed allows never do (:36-48, test :308-330); socket lives outside the workspace (:66-70, test :200-213); proxy start failure leaves network off, "the safe direction" (:52-58). Denied hosts escalate as an approval kind in headless output with `host_grant_stays_filesystem_sandboxed` retry scope (`forge-cli/src/main.rs:205-221`). Subagents inherit the parent grant, never a second proxy (permission.rs:470-511).
- HITL: unknown decision variants read as denial (:69-73 "Fail closed"); approved calls are **re-authorized** against governance before execution (:128, :187-212); sandbox-escalation approvals never execute the redacted display args (:22-33); two consecutive denials stop the turn outright (:39, :111-121); approve-all refuses to enable on an untrusted workspace (supervisor.rs:2332-2343) and trust setup gates first launch (forge-tui/src/launch.rs:13-25).
- Pattern rules (`governance/src/pattern.rs`): `tool(pattern)` can only *narrow* an already-gated call, never create a Deny, never un-gate; shell subjects normalized so `cargo test *` can't be inherited by `cargo test; <anything>` (:60-75).

## Verification

- 3,347 `#[test]`/`#[tokio::test]` functions; 254 rs files, 176 with `#[cfg(test)]`; integration dirs `crates/*/tests` = 23.4k LOC (tools 3,078 incl. `sandbox_enforcement.rs` 1,853 and `permission_contract.rs`).
- `sandbox_enforcement.rs:1-47`: kernel-denial e2e -- spawns real confined processes and asserts the OS refuses; loud skip reasons; `sandbox_is_available_on_linux_in_ci` (:1177) fails the build if CI loses bubblewrap.
- `permission_contract.rs:1-40`: contract-as-a-table harness, written around four real historical defects (inert rule from unresolved symlinks, `--tmpfs /var/run` symlink abort, bind/mask ordering, cross-layer gap); "Assert on behaviour, never on configuration"; "Never skip silently."
- CI: bubblewrap+socat apt-get *because* skips would silently untest (ci.yml:24-31), installer tested in CI (ci.yml:35), clippy `-D warnings`, `cargo deny` advisories/bans/sources with feature-resolved rationale comment (ci.yml:50-77), plus audit.yml, codeql.yml, release.yml. MockModelClient faux provider (`forge-model/src/lib.rs:11`) drives session-level tests.
- **No in-CI model evals, no fuzzing** -> verification stays at the anchors' 8 ceiling (errata).

## Orchestration / operability / interop

- Supervisor: concurrent repo sessions (default 4, supervisor.rs:28), each managed session in an isolated linked git worktree; goal-supervision loop with an LLM evaluator returning one-line MET/NOT_MET/IMPOSSIBLE verdicts (strict parse :38-58, prompt injects the condition as untrusted :61-64) capped at `MAX_GOAL_TURNS=30` (:29). Subagent coordinator with depth limit (default 2), mailboxes, followup/wake, interrupt, `shutdown_descendants` leak hygiene (agent_coordinator.rs:417-450). Task queue with promote/revert states (queue.rs:95-119). Headless backstop `MAX_APPROVAL_ROUNDS=64` (headless.rs:57).
- Durability: SQLite-backed event journal (`forge-durable/src/lib.rs`) with replay restoring queue items, background tasks, pending HITL, composer lines, cache epoch (:85-138); tool-intent-before-execution journalling (:469) and `incomplete_intents` recovery; `replay_perf.rs` test. Resume latest / resume by id (forge-cli main.rs:115-141); `fork()` (create.rs:372). No checkpoint-revert/rewind surface found.
- Interop: MCP client (stdio + remote, McpManager, tool filter), provider plane with OAuth for Anthropic/OpenAI-Codex/xAI/Ollama/opencode (forge-connect/src/*.rs 10.6k), usage/billing via models.dev + Codex usage endpoint (cost.rs:7-9). Headless contract is single-shot `forge bench` with JSON stdout incl. structured `approval_required` kinds -- no streaming event protocol, no RPC server, no MCP server, no ACP, no published SDK, no IDE surface.

## Docs

- README 665 lines accurate to code; FORGE-DESIGN.md 1,193 lines front-mattered `status: reconciled-with-code`, with **80 §-section references from Rust code into the spec**; ARCHITECTURE.md crate/dependency diagram matching Cargo boundaries; SECURITY.md with threat model and an explicit "what the sandbox does not protect" section; AGENTS.md/CONTRIBUTING present. `docs/` itself holds only two working docs; reference docs are thin relative to pi.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | turn.rs:16-57 loop; crate split with tools/governance independence (sandbox.rs:1-15); docked: supervisor.rs 7,395 + two 6.5k TUI files |
| verification | 8 | permission_contract.rs:1-40 + sandbox_enforcement.rs:1-47,1177; ci.yml:24-35,50-77; faux-provider; no CI evals/fuzzing -> ceiling |
| safety-enforcement | 9 | sandbox.rs:342-393 refuse-to-launch; permission.rs:36-58 deny-all egress; approval.rs:69-73,111-121 fail-closed; SECURITY.md honesty; below codex-10: 2 platforms, no native Windows, no execpolicy DSL |
| token-economy | 8 | policy.rs:98-122 next-turn-aware trigger + runway gate; facts.rs + engine.rs protected-fact gate; turn_ops.rs:713-800 cache epochs; no warming/second tier |
| orchestration | 8 | supervisor.rs:28-66, goal loop cap 30; agent_coordinator.rs depth/mailbox/shutdown; durable replay of queue+HITL+background (durable lib.rs:85-138) |
| interop | 6 | forge-mcp client + SearchToolsTool; OAuth provider plane; bench JSON (main.rs:205-247); no server/ACP/SDK/IDE |
| operability | 7 | resume/fork, worktree isolation, intent journalling + incomplete_intents; no rewind/checkpoint-revert |
| originality | 7 | protected-fact compaction validation; confine-or-refuse posture; mktemp wrapper (sandbox.rs:76-97); honest escalation kinds -- but scope explicitly "matching Codex", ideas recombined not pioneered |
| durability | 4 | 1 contributor visible, shallow clone (history evidence-limited); real release cadence + 4 workflows + SECURITY.md SLA; NorviaLabs backing unknown |
| docs-dx | 8 | FORGE-DESIGN reconciled-with-code + 80 code->spec refs; SECURITY.md threat model; installer tested in CI; docs/ dir thin |

**Weighted total: 75.0 -> Band B** (not within 2 pts of the 78 or 65 boundaries).

Sanity checks pass: safety>=5, verification>=5; no calibration-rule demotions (provenance "original", not a sync fork of the other forge).

## Census check

- LOC sane (cloc 158.3k Rust code vs census 142.4k non-test + 19.5k test). `test_loc` **undercounted**: census counted 19,513 (= forge-tui/src/app/tests) but missed `crates/*/tests/*.rs` (+3.9k: tools 3,078, durable 305, model 250, syntax 146, cli 114) and large inline modules (forge-core/src/tests.rs alone 5,746); real test LOC ~35k+ including inline. Tier unchanged (T2 either way).
- Shallow clone confirmed (.git/shallow); head_date 2026-09-27 is fresh, no activity claim made either direction. 1 commit visible => contributor count evidence-limited, not asserted as solo-authored project history (PR #810 in the squashed message implies a real PR flow).
