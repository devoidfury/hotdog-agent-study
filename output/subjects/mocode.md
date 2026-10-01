# mocode — T1 review

- Subject: `/data/samples/agents/mocode` (manifest `mocode-ai` v1.6.6, MIT, remote `wanxunyang/mocode`). Identity confirmed from package.json:2 — NOT mini-kode / minicode / nanocoder; no facts imported from those.
- Census row: TS, 71,001 non-test / 11,273 test LOC, contributors 1, head 2026-09-26, shallow clone (`.git/shallow` present, 1 commit), suggested T1, provenance "original".

## Anchor question (answered before scoring)

Closer to **crush**: same overall shape — approvals that bind and are tested with honest non-kernel ceilings, real orchestration machinery (resume journals, cron), MCP client, verification capped by no-model-evals-in-CI — with mocode's token economy well above crush's single-summarize rung and crush's loop-detection breaker missing on our side; expect it to land a point or two above crush's 69.

## What it is

Solo-authored TypeScript terminal coding agent, ~65.6k non-test TS across `src/` (28 subsystems) plus a Rust ratatui TUI (experimental, ADR-gated) that spawns the TS agent-host over an NDJSON protocol, and Electron work-app/pet-app consuming a separated `@mocode/runtime` boundary. ~28 builtin tools (src/tools/builtins), MCP client, skills (Agent Skills frontmatter standard incl. reading `~/.claude/skills`), memory store + graph + async reflection, background jobs with crash-resume checkpoints, cron scheduler, bots, headless `-p`/`--json` mode, per-turn rollback that explicitly avoids git.

## Core loop (read: src/agent/core.ts, pipeline.ts, run-coordinator.ts, stages/)

- Entry `runAgentCore` (src/agent/core.ts:17-27) assembles a per-run pipeline; a `legacy`/`staged` migration is in flight (`pipeline.ts:36-54`): every stage (model, tools, capabilities, termination, history, context) can be selected per run. The real loop is `runAgentCoreLegacy` (run-coordinator.ts:70-1118).
- Loop is pure logic, zero direct TUI dependency — all display via injected `AgentHooks` (core.ts header comment, spawn.ts:1-15); main agent, silent subagents, headless, and the host binary all share one loop implementation (the nanocoder anti-pattern of per-entry-path loop copies is absent).
- Per-step immutable tool-policy snapshot: "本步只捕获一次不可变 policy snapshot" (run-coordinator.ts:166-192); schema, runtime backstop, and descendant permissions derive from one effective allow-list; mid-step `add_tool_groups` expansion cannot enlarge the current step's ceiling.
- Tool dispatch: read-only runs batched parallel in call order; orchestration (subagent) calls chunked by `subAgentConcurrency` with permission preflight strictly in order; same-file mutations serialize on canonical-path resource locks; plan-mode runtime backstop denies hallucated write/exec calls even if the schema filter was bypassed (run-coordinator.ts:946-960). Permission check runs before execution for every non-safe call and every denial pushes a paired `tool_result` (protocol integrity invariant, :786-918).
- Abort semantics: history restored to turn start, mode restored, no orphan `tool_call_id` (header comment :14-19). Completion is accepted without any framework verification gate (run-coordinator.ts:1073-1075) — honest, by-design ("agent 自主决定是否验证").
- Docking: the legacy/staged dual path means dispatch logic exists twice (inline legacy ~600 LOC + `stages/tool-dispatcher.ts` 517 LOC) until migration lands; largest file is 1,239 LOC (src/ui/batch.ts), no god files, loop file 1,118.

## Compaction / context (read: src/session/scheduler.ts, compact.ts, context/budget.ts)

- Budget scheduler is the sole automatic history-rewrite entry point: 60% low-pressure = deterministic zero-LLM cleanup only (superseded prune, stale artifacts, retrievable-result clearing, age-aware encoding), 80% high-pressure = all cleanup then always LLM summarize (scheduler.ts:114-163; budget.ts:53 `pressureTriggerRatio: 0.8`; `lowPressureRatio` default 0.6, config/index.ts:727). Crossing 80% commits to compacting without re-checking per-stage — deliberate anti-oscillation design (:147-149).
- Three compaction layers: push-time per-message cap, summarize (main), micro-compact fallback preserving `tool_call_id` without LLM (compact.ts:45-56).
- User-intent preservation via two hard channels independent of summarizer goodwill: verbatim user channel (per-user-group budget min(20k tok, 10% window), Codex-style, compact.ts:935) and a **deterministic intent/constraint ledger** `extractUserDirectives` (compact.ts:462-495) citing COMPINT (summarizers retain ~17% of user constraints; independent extractor 90%+, compact.ts:54-55,464). Ledger accumulates across compaction generations.
- Occupancy is measured, not guessed: provider-reported usage feeds an EWMA estimator calibration with outlier rejection (token-calibration tests :32-61), ephemeral tail injections are made visible to the pressure line (scheduler.ts:52-56; context-budget.test.ts:123-162), and a real overflow regression is pinned ("289k/256k bug", context-budget.test.ts:100).
- Prompt-cache discipline: compactor forks the parent request (same system + history prefix + appended instruction) so the summarizer call itself hits the provider cache (compact.ts:1044-1046, compact-fork.test.ts:74-132 incl. auto-rollback when the fork wouldn't fit); subagent delegation rebuilds a byte-identical parent prefix, and the test asserts byte equality (spawn-permissions.test.ts:157).

## Permissions / sandbox (read: src/permissions/index.ts + README, src/sandbox/*)

- Default-on (config/index.ts:748 `permissionEnabled: process.env.MOCODE_PERMISSION !== 'false'`). Grants are fingerprint-bound, never tool-name-bound: exact-command hash for `run_command`, resource-path hashes for file tools, stable-args hash otherwise, coarse resources forced to retain args (permissions/index.ts:84-97); approving `npm test` provably does not admit `npm publish` (permissions/README.md).
- Scopes once/session/project + an explicit versioned `alwaysAllowTools` field; the retired v2 `allowForever` list is deliberately NOT migrated so upgrades cannot silently restore whole-tool authority (:100-122). Persistence failure never widens authority (:124-139).
- Fail-closed off-TTY: pipes/CI deny unless `MOCODE_PERMISSION_NON_INTERACTIVE_ALLOW=true` (:214). Dangerous-risk tools default the highlighted panel option to DENY (:251). `computer` tool types/keys containing URLs/password/payment patterns force once-only review, bypassing every grant cache, incl. CJK keywords (permissions/index.ts:57-68).
- Path jail: `jailResolve` realpath-based containment rejecting `../`, absolute escapes, and symlink-out-of-root incl. the new-file "nearest existing ancestor" path (sandbox/jail.ts:36-62), glob pattern vetting, centrally applied via `enforceSandbox` policy set (sandbox/policy.ts:39-70); escape tests use real symlinks (tests/sandbox.test.ts:20-40).
- Command layer is explicitly labeled "非安全边界" — env-secret denylist + catastrophe denylist only, bypasses acknowledged in-source (sandbox/command.ts:1-43). No bwrap/seatbelt; `run_command` confinement is cwd-pinning plus approval. Honesty about the ceiling is exemplary (pi-grade), the ceiling is real.
- Skill trust gate: project-sourced skills require one-time confirmation with sha256 over SKILL.md+scripts+references, invalidation on content change, strict fail-close non-TTY (skills/trust.ts:1-11).

## Verification

- 74 test files / 492 tests / 12.5k LOC node:test + CI matrix (typescript/electron/rust) running build, typecheck, lint, tests, offline eval smoke, cargo fmt/clippy `-D warnings`/test, and a Rust↔TS host-protocol smoke (ci.yml:16-110).
- Faux provider: `__setChatCreateImpl` stubs OpenAI `create` with hand-shaped SSE chunks and drives the full stream→tool_call→exec→refeed→final-text loop asserting history structure, hook order, byte-reproducibility (agent-core.test.ts:1-57); same trick tests compaction fork.
- Property-style tests exist (300-case invariants, context-budget.test.ts:164,200); permission/skill-subtraction tests assert fail-closed narrowing (spawn-permissions.test.ts:69,204).
- Ceiling: no fuzzing, no model evals in CI. The 61-task coding benchmark (basic/hard/advanced, evals/README.md) needs a real API key and runs manually; CI gates only the offline smoke (ci.yml:59-61). Per errata this caps verification at 8; density (12.5k test vs 71k code) and thin loop-periphery coverage put mocode one rung under pi's 8 → 7.

## Orchestration / operability / interop

- Subagents: same-source-as-parent design (identical prompt/tools/prefix for cache hits), depth gate tested at 3 (subagent-depth.test.ts:31), parallel/changed-files tests, abort tree-kill propagation (spawn.ts:1-15,69).
- Background jobs: atomic full-history checkpoint after each tool batch, `mocode resume-job <id>` crash resume (jobs/checkpoint.ts:1-19, index.ts:105), worktree-isolated runs (`--worktree`, headless.ts:12-15), cron daemon (src/schedule/), bots with tool whitelists, message-bus tool.
- No mechanical loop detector — repetition mitigation is advisory prompt text ("after repeated identical failures, change approach", work-discipline.ts:31) plus the maxSteps cap; crush remains the only subject with a real breaker.
- Rollback: per-turn file/workspace snapshots entirely git-agnostic, never touches index/staging (rollback/store.ts:22-26, 720 LOC).
- Interop: MCP client (src/mcp, single-server failure non-fatal, index.ts:14-24); NDJSON host protocol as a published-boundary package with CI-enforced Electron boundary check (`check:electron-boundary`, ci.yml:70-72); Agent Skills open standard and `~/.claude` skill directory compatibility (skills/discover.ts:4-6); no ACP, no published SDK, no IDE plugin.

## Originality

Directive-ledger compaction (compact.ts:462-495), compact-fork summarizer + byte-identical delegation prefix as cache strategy (compact.ts:1044, run-coordinator.ts:196-215), tool-route disclosure groups with immutable per-step snapshots plus a small-model classifier auto-expanding groups (tools/router.ts:133 "Jev router"), refusal to migrate `allowForever` (permissions/index.ts:112-114), and ADR-governed experimental Rust stack with named owner + promote-or-archive deadline 2026-10-31 (rust/README.md:3-7, docs/adr/0001) — governance unusual for a solo repo.

## Durability

One contributor, shallow clone (activity claims limited to: HEAD is 3 days old, version 1.6.6, nothing indicates decay; no remote re-check needed per 10-month rule). No SECURITY.md, no institutional backing. Discipline signals (ADRs, runbooks incl. a TUI-freeze runbook, CI, eval harness) are strong for a solo project but bus factor is 1 → 4, the nanocoder rung.

## Census sanity

- non_test 71,001 ≈ my recount: src 51,469 + packages 8,971 + rust 8,020 + evals 3,168 + bin = 71.6k. Sane.
- test_loc 11,273 vs `tests/` recount 12,522 (~10% under, likely excludes helpers/tsconfig); rust/tests/render.rs unaccounted. Minor, no tier impact.
- shallow=true confirmed; head_date recent; contributors 1 consistent. suggested_tier T1 correct (65.6k non-test TS in the core, whole tree inside T1 band).

## Provenance

Census "original", `rename_clone_suspect: false`; manifest name `mocode-ai`, README/bin consistent; nothing suggests a sync-fork — rule (a) not engaged. Solo repo, cannot fully exclude prior private lineage; treating as original on the record.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7 | hooks-decoupled pure loop (run-coordinator.ts:1-19, core.ts:17); per-run stage assembly (pipeline.ts:36); but in-flight legacy/staged duplication doubles the dispatch path (run-coordinator.ts:330-1090 vs stages/tool-dispatcher.ts) |
| verification | 7 | faux-SSE full-loop test (agent-core.test.ts:37-57); 300-case property tests (context-budget.test.ts:164,200); 3-stack CI + clippy -D warnings (ci.yml:88-96); no fuzzing, model bench manual-only (evals/README.md) |
| safety-enforcement | 7 | fingerprint grants + fail-closed non-TTY + deny-default panel (permissions/index.ts:84,214,251); realpath jail w/ symlink tests (jail.ts:36-62); skill trust hash gate (trust.ts:1-11); command layer self-labeled non-boundary (command.ts:1) — strongest non-kernel enforcement observed, kernel rung absent |
| token-economy | 8 | 60/80 dual-pressure tiering (scheduler.ts:114-149); compact-fork cache reuse (compact.ts:1044 + compact-fork.test.ts:74); measured-usage EWMA + ephemeral visibility (context-budget.test.ts:123-162); lazy tool-schema groups (profiles.ts:96,184) |
| orchestration | 7 | tested depth-gated subagents (subagent-depth.test.ts:31); atomic job checkpoints + resume-job (jobs/checkpoint.ts:1-19); cron + worktrees; no loop breaker (work-discipline.ts:31 advisory only) |
| interop | 7 | MCP client non-blocking (mcp/index.ts:14-24); versioned NDJSON host protocol + CI boundary gate (packages/protocol, ci.yml:70-72); `~/.claude` skills compat (discover.ts:4-6); no ACP/SDK/IDE |
| operability | 7 | sessions + retention + rollback snapshots (persist.ts:20-29, rollback/store.ts:22-26); trace emitter throughout; runbooks incl. tui-freeze; no crash-recovery middleware posture at every boundary (crush's 8 trait) |
| originality | 8 | directive ledger (compact.ts:462-495); delegation byte-identity (spawn-permissions.test.ts:157); JEV tool-router (router.ts:133); allowForever non-migration (permissions/index.ts:112); ADR-promotion-deadline governance (rust/README.md:3-7) |
| durability | 4 | contributors=1, shallow, no SECURITY.md, solo bus factor; ADR/runbook discipline noted |
| docs-dx | 7 | bilingual README + 344-line usage + 4 ADRs + 5 runbooks + honest subsystem READMEs (permissions/README.md); no docs site, eval README hardcodes `F:\mocode` paths |

**Weighted total: 70.5 → band B.** Strongest: token-economy (8, tied with originality). Weakest: durability (4).

## Calibration notes

- Safety 7 is earned only by the combination: default-on, fingerprint-scoped, fail-closed-off-TTY, hash-gated skills, path jail — each tested — PLUS honest ceiling documentation. A subject missing any leg must not read this row as license for 7.
- No demotions; provenance original so rule (a) N/A. Boundary check: 70.5 is >4 from 65 and >7 from 78; no boundary note.
