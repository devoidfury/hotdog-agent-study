# jazz (jazz-ai) - T2 deep review

Subject: /data/samples/agents/jazz, manifest `jazz-ai` v0.15.43, lvndry, MIT, TypeScript/Effect-TS, monorepo (`packages/{core,adapters,runtime,cli,plugin-sdk,bot-shared,telegram-bot,discord-bot,imessage-bot,whatsapp-bot,website}` + `plugins/` + `evals/`).

**Anchor question:** Closest anchor is **pi**: like pi, jazz is a provider-agnostic Effect-based core consumed by thin product surfaces (CLI/Ink, four bots, daemon) with spec-named colocated tests and cache-aware compaction; jazz exceeds pi on orchestration/remote-surface breadth but lacks pi's session-tree operability and cache warming, landing just below pi's 78.5.

## Census sanity

- `test_loc: 994` is a glob miss. Actual colocated tests: 358 files `*.test.ts(x)` = **83,139 LOC** (`find packages plugins evals scripts -name "*.test.ts*" | xargs wc -l`).
- `non_test_loc: 213,697` overstates; measured non-test `.ts/.tsx` (packages+plugins+evals+scripts) = **157,272**. Total ≈ 240k. Tier T2 stands either way.
- `commits: 1` + `.git/shallow` present => shallow artifact; head_date 2026-09-27 is fresh. No activity claim either direction beyond remote head date.
- Provenance `original`, no fork flag, no rename suspect; name "jazz" has no corpus collision on the identity list.

## Core loop (read at line level)

`packages/core/src/agent/execution/agent-loop.ts` (1745 LOC) - shared loop behind batch and streaming strategies (`CompletionStrategy` interface :394-460):

- **Budget pressure nudges**: iteration 70/90% (:79-91), cost/token/time 50/80/90% (:100-171), context warn/critical (:199-224), and a **post-compaction counter-nudge** (:231-252) that tells the model to *continue*, not wrap up, after space was freed. All nudges are ephemeral request-only, never persisted - explicitly so they don't "get summarized into the very compaction they warned about" (:1176-1178).
- **Meltdown detection**: `detectMeltdown` (:354-366) - 10-call window, uniqueness on composite `name:arguments` (<0.4 fires) so alternating productive tools don't trip it; recovery message injected *after* tool results, never between a call and its result (invariant tested at agent-loop.test.ts:2219-2269).
- **In-batch dedup**: `dedupeToolCalls` (:382-417) collapses byte-identical calls decided *before execution*, restricted to `riskLevel === "read-only"` tools (:788-799); duplicates aliased onto the survivor's result because providers require one tool message per call-id.
- **Transcript integrity**: `closeDanglingToolCalls` (:581-604) on interrupt; `reportFailedTurn` (:617-640) hands the failing turn's transcript to the caller before it unwinds; parked runs deliberately keep their tool call unanswered (:623-626).
- **Resume-mid-turn**: `pendingToolCalls` (:1573-1590) - a resumed run finishes the tool call it parked on (with the just-granted approval) before the loop iterates.
- Cost caps: soft checkpoint between iterations (:1650+), and `costIncomplete` honesty so cap-enforcers "refuse to enforce a cap they cannot verify" (:553-556).
- God-file scan: loop 1745, `tool-executor.ts` 1151, `agent-runner.ts` 1146; largest files are product-surface (`cli/src/chat/commands/handler.ts` 3326, `adapters/llm/ai-sdk-service.ts` 2439, `runtime/cli-app.ts` 2437) - no fused god-file in the core.

## Compaction / context management (read at line level)

Four-rung ladder with explicit ratios (`context-window-manager.ts:21-41`): **clear 0.50 → warn 0.70 → compact 0.80 → trim 0.95**, applied cheapest-first in `runIteration` (agent-loop.ts:1052-1105):

1. **Offload**: `persistLargeToolResults` writes big tool bodies to disk (placeholder re-fetches); failed write still stubs (:1056-1058 comment).
2. **Clear**: deterministic `clearToolResults` outside the protected window, or a plugin-advised keep/truncate/drop decision (`reduceToolResults`, falls back deterministically - it "never removes a message", :1064-1072).
3. **Compact**: `Summarizer.compactIfNeeded` (:1119-1125) - fixed-schema checkpoint (`## Goal / Constraints / Progress / Key Decisions`, summarizer.ts:78-97), `PINNED_KINDS` guarantees workflow task prompts survive verbatim (summarizer.ts:50-57), memory extraction runs at the compaction boundary gated by `mayExtractMemories` (no sub-runs, no ephemeral; agent-loop.ts:677-684).
4. **Trim** as floor, keyed to 0.95 of the *model window* with an explicit comment that a constant budget made trim preempt compaction and rewrite the cacheable prefix every turn (:1434-1440).

Token counting is **calibrated from authoritative provider usage each turn** (`calibrateTokenCounter`, :1289-1295). Every rung emits before/after telemetry (`logContextRung`).

**Prompt-cache discipline** (`ai-sdk-service.ts`): Anthropic/OpenRouter `cacheControl` breakpoints (:383-404); the conversation breakpoint targets the **last persisted message** so a trailing ephemeral nudge - text that changes every turn - never becomes the cached boundary (:521-523, :550-565); attachment aging-out is accounted as exactly one cache miss (:366-369). Work-state (`work-state.ts:7-30`) is a model-written structured intent doc that survives compaction; a separate append-only work journal records what happened.

## Permission / safety enforcement (read at line level)

- **Capability before approval, fail-closed at execution**: `executeTool` refuses when no `effectiveToolNames` set or name outside it (:96-121 tool-executor.ts); refusal returns a model-actionable tool result, not a failed run (tested tool-executor.test.ts:940-1023). The hidden `execute_*` half is refused when the model names it directly (:974, :1040).
- **Approval pairs**: proposals (`isApprovalRequiredResult`) intercepted at :321-430; "always approve" re-checked at dequeue time via `isAutoApproved` callback so a parallel tool's grant applies (:437-446 comment). Picker-style approvals are never auto-approved even under yolo (:386-390, tested :822).
- **LLM command-risk classifier** (`command-risk.ts`): `execute_command` declares `unknown` risk; a cheap model answers one token (read-only/low-risk/high-risk), 8s timeout, 16 max tokens, anything ambiguous stays high-risk (:33-45, :74-82); plugin policy hook may supply the verdict but "cannot expand the effective tool set, override allowlists or the selected tier" (threat-model.md:34-40); conversation evidence only used when the person being protected wrote it (:365-370 comment).
- **Disclosure tiers** for external doors (peers, webhooks) (`types/disclosure-tier.ts:1-50`): a tier intersects the toolset *down* - "there is nothing outside the tier for a persuasive payload to talk its way into" - and anything riskier than read-only must be named per caller. Tested (`webhook-authorization.test.ts`, `daemon/token.test.ts`).
- **Edit CAS**: `edit_file` digests + per-file lock recheck before write, including across parked approvals (threat-model.md:45-50).
- **Daemon**: refuses to bind non-loopback without a token (daemon/server.ts:149-153); constant-time token compare (:167-173).
- **No OS sandbox.** None found (zero hits for seatbelt/bwrap/landlock). Honesty is exemplary: threat-model.md:7-9 "It is not a sandbox against a hostile model, compromised dependency, or operator"; denylist admitted bypassable (:40). This caps the dimension at 7 regardless of how well-tested the app-level gates are.

## Tests / CI / evals

- 358 colocated test files, 83k LOC, names read as property specs ("refuses a tool the run was never granted, however well the registry knows it", tool-executor.test.ts:940; "injects the meltdown signal after the tool results, never between a call and its result", agent-loop.test.ts:2219). Integration tests: `park-resume.integration.test.ts`, `detach/handoff.integration.test.ts`.
- CI (`ci.yml`): paths-filtered docs+link check; lint+typecheck+catalog rebuild+binary build; **smoke test asserts on binary output strings, not exit status** ("the crash this exists for printed 'Fatal error' and exited 0", :103-104); separate typecheck-tests + test job. Ubuntu only, no coverage gate.
- **Evals not in CI**: `evals/` has pass@1/pass@k, A/B attribution, per-run cost, ambient-LSP and memory-journey acceptance tasks (evals/runner.ts:19-42, README) - no workflow references `evals`. Matches the seeded verification-8 ceiling (no in-CI evals, no fuzzing).

## Orchestration / interop / operability

- `spawn_subagent` (subagent.ts) with persona/model/depth/iteration budgets, live steering messages (:37-43), child cost rollup (agent-loop.ts:729-733); workflows with scheduler + catch-up + run-history (`core/src/workflows/`); job queue; daemon job-worker; Ctrl+B backgrounds an in-flight tool batch without aborting the run (agent-loop.ts:435-447).
- **Park/resume journal**: parked runs persist transcript + pending approval + TTL (`run-recorder.ts:29-60`), answerable from another process via daemon `POST /runs/:id/answer` (docs/concepts/daemon.md:51-52).
- Surfaces: CLI/Ink, Telegram/Discord/iMessage/WhatsApp bots, daemon, webhooks, peer-agent network (`ask_peer` with question-as-parameter and reply-as-quotation, peer.ts:1-15). MCP client with OAuth + elicitation (adapters/mcp/). Headless `jazz run --json` envelope incl. ephemeral multi-turn round-trip (runtime/cli-app.ts:104,169). Plugin SDK released separately (release-plugin-sdk.yml). **No ACP; not itself an MCP server** (keywords claim "mcp-server" - marketing, not evidence).
- Operability: run-store, park/resume, detach-to-remote-host, `jazz update`, per-conversation log groups, OTLP telemetry, config wizards. **No git checkpoint/rewind** - no revert story for agent edits beyond the edit CAS.
- Skills: metadata index (level 1) + on-demand SKILL.md load (level 2), deterministic keyword routing, no embeddings (skill-service.ts:50-115).

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 8 | strategy-split loop agent-loop.ts:394-460; interfaces/ layer DI; no fused god-file in core; product 2-3k files dock it from 9 |
| verification | 8 | 83k LOC property tests (tool-executor.test.ts:859-1060); CI smoke asserts output (ci.yml:100-130); evals exist but not in CI, no fuzzing = anchor ceiling |
| safety-enforcement | 7 | enforced+tested capability boundary and approval pairs, fail-closed classifier, disclosure-tier intersection; zero OS-level isolation underneath (honestly documented) |
| token-economy | 8 | 4-rung ladder ratios 0.5/0.7/0.8/0.95 (context-window-manager.ts:21-41); cache breakpoint targets last persisted msg (ai-sdk-service.ts:550-565); calibrated counter |
| orchestration | 8 | park/resume w/ cross-process approval, budgeted subagents, scheduler+catch-up, daemon, detach |
| interop | 7 | MCP client+OAuth, --json envelope, plugin SDK, 4 bot surfaces; no ACP, not an MCP server |
| operability | 8 | park/resume journal tested end-to-end, failed-turn transcript capture, OTLP; no checkpoint/rewind caps it |
| originality | 8 | disclosure-tier capability ceiling, peer-as-quotation protocol, post-compaction counter-nudge, work-state/journal/memory taxonomy |
| durability | 5 | solo author (shallow-clone), 0.15.x churn, good release/CI infra, no visible co-maintainers |
| docs-dx | 8 | threat-model.md, concepts/ features/ guides/ tree matching code; SECURITY.md reporting policy; Makefile |

**Weighted total: 76.5 → band B (top of band; within 2 of the 78 A-boundary - flagging for synthesis).**
Strongest dimension: token-economy (ladder + cache monotonicity, all tested). Weakest: durability (solo bus factor).

## Provenance / calibration

- Original per census and self-consistent manifest (jazz-ai, lvndry repo, MIT file+manifest). Fork rule (a) N/A. No similarly-named corpus subject; no cross-imported facts.
- No demotions applied. Boundary note: 76.5 sits inside the 2-pt A-boundary window; if synthesis re-anchors interop at 8 (e.g., weighing the published SDK + four surfaces as cline-tier), this crosses to A - I kept 7 because there is no ACP and no MCP-server exposure at a codex-verified depth.
