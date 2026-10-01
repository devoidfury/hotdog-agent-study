# Boundary re-review: jazz (jazz-ai) - independent ten-dimension pass

Provisional 76.5 (B), 1.5 under A floor (78). Named hinges: interop 7 (ACP/MCP-server
absence), token-economy ladder, verification 8 vs evals. Read-only static review;
`.git/shallow` present (1 commit) - history is evidence-limited, but HEAD commit date
2026-09-27 is positive recency evidence. Non-test TS ~136k LOC (wc sum) -> T2 volume.

**Closest anchor: pi (78.5, A).** Single-author strong-vision system with cache discipline
treated as a first-class feature, honest security posture, deep in-repo docs; jazz is
broader on surfaces (bots/daemon/A2A) but thinner on institutional backing and has an
estimator-based rather than measure-based compaction trigger. Not cline: no IDE surfaces,
no multi-surface SDK.

## (1) Token-economy: is the ladder tested against provider cache semantics? Measure vs estimate?

The ladder is real, wired, and tested at the unit level:
- Four rungs: clear 0.5 / warn 0.7 / compact 0.8 / trim 0.95
  (`packages/core/src/agent/context/context-window-manager.ts:21,27,30,41`). Ordering is
  an enforced design property, not a convention: trim deliberately sits *above* compact
  so it can never silently degrade the design to a sliding window
  (`context-window-manager.ts:33-41`), and config validation rejects a compact ratio at
  or above trim and a warn ratio at or above compact
  (`context-thresholds.test.ts:36-49`). Clear rung gated in the loop
  (`packages/core/src/agent/execution/agent-loop.ts:1058-1060`), warn at
  `agent-loop.ts:1119-1133`, trim at `agent-loop.ts:1346`.
- Budget math includes request overhead (tool schemas, provider scaffolding):
  `context-window-manager.ts:326-331` (comment: counting messages alone understates by
  10-30k with MCP attached), and a test asserts compaction fires on overhead the messages
  alone would not reach (`context-window-manager.test.ts:306,319`).

**Measure vs estimate: jazz's trigger MEASURES NOTHING pre-call; it estimates, but the
estimate is feedback-anchored to provider ground truth.** `TokenCounter.countText` is a
pre-call estimator - exact gpt-tokenizer BPE for OpenAI families, chars/token ratio
otherwise (`token-counter.ts:20-24,215-238`) - and after each response `calibrate()`
consumes the provider's authoritative `usage.promptTokens` to learn the per-model ratio
AND to measure per-request overhead as the usage-minus-estimated-messages gap
(`token-counter.ts:342-401`, wired at
`packages/core/src/agent/metrics/agent-run-metrics.ts:311`). Calibration is tested with
synthetic usage reports (`token-counter.test.ts:205-251`). So:
- vs **codex 9**: codex gates prompt-cache reuse by cache keys
  (`core/src/client.rs:354-390`, cache-key *semantics*); jazz has no keyed reuse, only
  breakpoint placement + an `openai: promptCacheKey` on the system message
  (`packages/adapters/src/llm/ai-sdk-service.ts:390-394`).
- vs **pi 8**: pi's trigger measures the projected outgoing request
  (`compaction.ts:289,919`); jazz's trigger estimates that request and then corrects the
  estimator from real usage. One round-trip of lag behind pi, but with a genuinely
  measured overhead component pi's trigger does not separately model.

Cache-breakpoint discipline is tested **against modeled provider cache semantics, not
live providers**: conversation-prefix breakpoint on the last content part for Anthropic,
provider-gated to anthropic/ai_gateway/openrouter, no breakpoint for providers without
explicit cache control, and the standout test - a trailing ephemeral nudge must NOT carry
the breakpoint (`ai-sdk-service.test.ts:1950-2016`; implementation
`ai-sdk-service.ts:520-529`). Ephemeral nudges exist for context/budget/time/cost
pressure (`agent-loop.ts:85-176,1247`) and are deliberately never persisted, which keeps
the cached prefix stable across turns - a cache-monotonic property by construction.

Rung: **8.5**. Above pi's 8 on ladder depth (4 rungs vs 2), overhead-in-budget, config
ordering gates, and cache-aware nudging; below codex's 9 because the trigger estimates
(ratio-branch inexact for all non-OpenAI families), there is no cache warming or keyed
reuse, and cache behavior is validated against option-shape assertions, not live cache
hit/miss accounting.

## (2) Interop: 7 vs 6 on ACP/MCP-server absence

Confirmed absences (grep across `packages/`, `docs/`): zero ACP references; no MCP
*server* exposure - `MCPServerManager` is a client-side connector
(`packages/adapters/src/mcp/mcp-server-manager.ts:1-3`), and the package.json
`mcp-server` keyword is marketing, not evidence.

But 6 (the pi rung: no MCP at all, by philosophy) is wrong for jazz, because what IS
present and tested exceeds the 7 exemplars' floor:
- MCP client with real coverage: connection manager + oauth + elicitation + trust tests
  (`mcp-server-manager.test.ts` 10 tests, `mcp-trust.test.ts`).
- A2A server-side door: Google Agent2Agent protocol implemented on the peer transport
  with agent card and `SendMessageResponse` envelopes, tested by parsing responses with
  the official `@a2a-js/sdk` classes (`packages/adapters/src/peers/a2a.ts:1-27`,
  `peers/a2a.test.ts:1,70,147-152`) - i.e. tested for wire compatibility with stock
  clients, not self-consistency. Reuses jazz's own tier/persona/token auth rather than
  bolting on a second trust system (`a2a.ts:9-13`). No anchor subject has this.
- Headless contract: `jazz run --json` envelope with dedicated tests
  (`packages/cli/src/commands/run/envelope.ts:179`, `envelope.test.ts`).
- Published package with CI provenance: `@jazz/plugin-sdk` via trusted publishing
  (`.github/workflows/release-plugin-sdk.yml:26-28`) - though types-only plugin ABI,
  not a rival-harness-importable agent SDK.
- Surfaces: CLI + 4 messaging bridges + daemon - breadth without IDE presence.

Ruling: **7 stands**. crush got 7 with MCP-client-plus-gate-test and no SDK/IDE; jazz is
at or above that on every axis. 8 (cline: broad published SDK + four IDE/CLI surfaces) is
not reached: no IDE surface, SDK is types-only, no ACP. Demoting to 6 for absence-based
reasoning would ignore tested protocol surface; 7 is correct.

## (3) Verification: does ANY CI workflow run the eval harness?

No. The eval harness is substantial - tasks, judge, metrics, held-model A/B,
memory-judgment calibration, LSP ambient-comparison task (`evals/README.md`,
`evals/runner.ts`, `evals/judge.ts`, `evals/memory-judgment-calibration.ts`) and is
typechecked (`package.json` `typecheck:evals`) - but grep across all seven workflows in
`.github/workflows/` for eval/evals returns **zero hits**. The only scheduled workflow is
`community-plugin-catalog.yml:5-6` (catalog build cron), which runs no evals. CI runs:
markdown lint + docs links, lint, typecheck, catalog rebuild, binary build, an output-
asserting binary smoke test, and `bun run test` (`ci.yml:176-179`).

Per the ERRATA, the verification-8 ceiling (real test corps, no in-CI evals, no fuzzing)
is exactly jazz's shape, and a workflow that exists but never runs evals does not qualify
- there is not even a nightly. No fuzzing anywhere (grep: zero hits). Test corps is real:
347 test files, loop-level property tests including budget caps and meltdown detection
(`agent-loop.test.ts:1431-2283`), binary smoke asserting on output rather than exit code
(`ci.yml` smoke job). **8 stands.** The eval harness living entirely outside CI is
recorded as finding jazz-b5.

## Other dimensions (independent pass)

- **architecture 7.5**: Effect-TS tag-DI with an interfaces layer
  (`packages/core/src/interfaces/`), loop cleanly separated from every product surface
  (`agent-loop.ts` 1,744 LOC, `agent-runner.ts` 1,146 LOC), adapter split. Docked from 8:
  surface-layer weight - `chat/commands/handler.ts` 3,326, `cli-app.ts` 2,437, bridges
  2,434/2,085. Errata's >5k-file rule not triggered; loop/surface separation does hold,
  which puts it at the cline rung less host thinness.
- **safety-enforcement 6.5**: above the tested-approval rung (6): approvals bind on all
  surfaces; LLM command-risk classifier gates `execute_command` and fails closed on
  timeout/ambiguity (`command-risk.ts:1-29`); denylist honestly labeled "defense-in-depth
  denylist, not a sandbox" (`shell.ts:41-47`); kernel-enforced per-conversation uid +
  JAZZ_HOME isolation for multi-user bot bridges (`chat-sandbox.ts:9-28`) - but engages
  only under root, and the interactive CLI has nothing underneath the approval gate.
  Honest posture (SECURITY.md, self-aware comments) keeps it from nanocoder's 5.
- **orchestration 8**: durable run records outliving the process
  (`run/run-record.ts:1-12`), park/resume restricted to lone tool calls so replay is
  exactly-once (`run/resume.ts:1-8`), job-queue + wake-trigger tools, escalating budget
  pressure nudges (iteration/time/token/cost at 50/80/90, `agent-loop.ts:85-176`), daemon
  (1,818 LOC). Cline rung; short of codex's queues+journals+agent-graph.
- **operability 7**: session logs, park/resume, detach, daemon, config wizard, self-update
  (`update-binary.ts`); no rewind/fork/checkpoint evidence found - below cline/crush's 8.
- **originality 8**: cache-aware ephemeral nudging (breakpoint-skipped pressure messages),
  per-conversation uid OS sandboxes, usage-feedback token calibration with measured
  overhead, plugin-advisory tool-result clearing with deterministic fallback
  (`agent-loop.ts:1069-1078`), A2A-on-existing-trust. All verified in code, not marketing.
- **durability 5**: solo author (lvndry), MIT, SECURITY.md, trusted publishing, active
  HEAD; bus factor 1, no institutional backing; history evidence-limited (shallow).
- **docs-dx 8**: 74 in-repo md pages spanning concepts/design/security/surfaces,
  link-validity CI-gated (`ci.yml` docs job). Docked: eval design specs referenced as
  "local-only" (`evals/README.md:4`) - documented-but-absent.

## Verdict

Weighted total **74.75 -> B**. Both named hinges resolve in jazz's favor (interop stays
7; token-economy is genuinely strong at 8.5) and the verification-8 ceiling holds by the
errata's own terms, but the independent pass lands below the provisional 76.5 on
architecture (3.3k chat handler and other heavy surface files) and keeps the ceiling
gates closed. **A floor not crossed.** The delta from provisional is carried entirely by
architecture 7.5 and safety 6.5; neither is a judgment call against evidence - line
citations above.
