# Boundary re-review: pi (independent, full rubric)

Phase 1 put pi at 78.5 (A floor). This is a fresh full-rubric read at ~248k LOC claimed / measured 203,455 non-test + 165,819 test LOC across 624 `*.test.ts` files. Read-only; no subject code executed. `.git` not shallow; HEAD 2026-09-25 (d6af72e18), 3,143 commits in last 6 months -- active, evidence not limited.

## Closest anchor

Closer to **cline (76.5)** than to codex (88.5): both have tested loop semantics and a clean core with no kernel-level safety floor, but pi edges above on token economy, operability and docs, and below on interop surface area (MCP/ACP absent).

## Core loop + state

Provider-agnostic 898-LOC loop, `packages/agent/src/agent-loop.ts:37` (`agentLoop`), event-sourced, continuation-guarded (`agent-loop.ts:77-81` rejects continuing from an assistant message). Tool-call pipeline validates args then consults `config.beforeToolCall` and honors `block`/`terminate` at `agent-loop.ts:721-746`; blocking semantics are tested (`packages/agent/test/agent-loop.test.ts:1735,1793`). Three-plane split (agent / ai / coding-agent) with the loop owning no provider or product concern. Package boundaries are CI-enforced, not folklore: `package.json:21-29` (`check:entry-graphs`, `check:pinned-deps`, `check:runtime-deps`) run inside the `check` step of CI.

## Compaction + cache discipline

Trigger measures the *projected* context, not raw history: `agent-session.ts:598-599` and `:2731` call `estimateProjectedContextTokens(...)` into `shouldCompact` (`core/compaction/compaction.ts:289`); re-compaction chains from the previous projected compaction boundary (`compaction.ts:900-940`). Branch summarization on tree navigation is real (`compaction/branch-summarization.ts`, 382 LOC). Cache warming is an engineered feature with explicit economics: `$0.05` minimum expected savings, 0.15 idle-continuation probability measured from their own usage, TTL-aware delay, and an Anthropic budget-thinking replayability guard (`core/cache-warmer.ts:17-60`), covered by `test/cache-warmer.test.ts`. This is the strongest cost-discipline implementation in the anchor set.

## Safety enforcement

No sandbox by design, stated plainly: `SECURITY.md:50` ("intentionally does not have a sandbox"), `docs/security.md` table of isolation options, and an unusually honest caveat that project trust cannot undo the initial `sessionDir` lookup (`docs/security.md:31`). What actually binds: the project-trust gate before loading `.pi` resources (`core/project-trust.ts:24-29`, trust tests `test/trust-manager.test.ts`) and the enforced blocking hook (`agent-loop.ts:739`). Approval policy itself is extension-owned (`examples/extensions/permission-gate.ts`), i.e. opt-in. That is exactly the "tested gate, nothing underneath" rung -- 6, not 7: no built-in approval default and no OS boundary.

## Verification

624 test files / 166k LOC, spec-named after regressions (`bash-close-hang-windows.test.ts`, `agent-session-tree-navigation.test.ts`). Tests assert loop mechanics rather than snapshots (blocked-tool-call termination, cache-warm decisions against fixture TTLs). CI runs build+check+test on every push/PR to main (`.github/workflows/ci.yml:35-42`), plus a contributor gate (`pr-gate.yml`). Ceiling matches cline/codex at 8: no model evals in CI (`packages/evals` exists with vitest-evals/autoevals and doc-audit evals, `packages/evals/package.json:9-14`, but no workflow runs it) and no fuzzing.

## Orchestration / interop / operability

- Orchestration 7: steering/followUp input queues in-session (`agent-session.ts:263-272`, RPC `streamingBehavior` `docs/rpc-commands.md:20`), durable store with rewindable/fork document semantics (`packages/durable/src/types.ts:36-50`), RPC server plane. No built-in subagents, no cross-session queue, no loop-detection breaker.
- Interop 6: excellent headless contract (SDK, JSON, RPC JSONL with request-id correlation `docs/rpc.md`), CBOR remote-session protocol + server/client packages (`packages/protocol/package.json:8`). No MCP, no ACP anywhere (grep of all package.jsons and sources empty), so no import from rival harnesses. Deliberate, but the ecosystem gap is real.
- Operability 9: session tree with `/tree` `/fork` `/clone` (`docs/sessions.md:25-29`), `--continue/--resume`, crash journal `~/.pi` crashes.json with capped records (`core/crash-log.ts:16-22`), bug-report flow, telemetry package.

## Durability / docs

MIT, solo-weighted (Zechner 3,783 of ~6k commits; Ronacher 734 next), heavy cadence, published under `@earendil-works` on npm; thin institutional backing but not one-person-churn -- 7. Docs: 39 in-repo files (`packages/coding-agent/docs/` incl. how-pi-works, session-format, rpc, security, containerization) that match the code read here -- 9.

## Architecture caveat (the A/B swing point)

Residuals exist: `modes/interactive/interactive-mode.ts` is a 6,852-LOC UI host class (class at line 415) and `core/agent-session.ts` is 4,023 LOC -- bigger than the god-file residuals codex was allowed at 9. I still score 9 because the files are cohesive presentation/session-orchestration layers (components split into `modes/interactive/components/`), the core loop is untouched by them, and boundary enforcement is mechanized in CI. But a stricter reading docks this to 8 and pi lands at 77.0 (B). Phase 1's 78.5 survives my independent recount, at the floor, on this dimension's goodwill.

## Scores

architecture 9 | verification 8 | safety-enforcement 6 | token-economy 8 | orchestration 7 | interop 6 | operability 9 | originality 9 | durability 7 | docs-dx 9 → **78.5, band A** (band confirmed, but floor-sensitive; see caveat).

Strongest: tie between operability/token-economy/originality. Weakest: safety-enforcement and interop, both 6.
