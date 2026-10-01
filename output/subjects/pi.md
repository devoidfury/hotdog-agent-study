# Anchor review: pi (earendil-works/pi, TypeScript) — A (78.5, bottom of band)

Tier: T2 (census 248k non-test / 152k test LOC — census sane here; tests are 38% of code). Full history: 311 contributors, 6546 commits. MIT.

**Closest anchor: n/a — is anchor.** Spiritually opposite pole to codex: minimal trusted-context core with everything else as extensions.

## Dimensions

### architecture — 9
- Clean three-layer split: provider-agnostic loop (`packages/agent/src/agent-loop.ts`, 898 LOC; `agentLoop` :37, `runAgentLoopContinue` :127), unified provider/model layer (`packages/ai`), product layer (`packages/coding-agent`), plus separate `protocol`/`server`/`client`/`durable`/`session-backends` packages.
- Extension runner is the spine: providers, tools, approvals, even subagents are extensions (`packages/coding-agent/src/core/extensions/runner.ts`, `examples/extensions/` has custom providers, dynamic tools, permission gates).
- Deduction from 10: coding-agent core still carries 60+ files in one dir (`src/core/`); product concerns (bug-report, footer-data-provider) share the layer with session machinery.

### verification — 8
- 625 test files / ~151k LOC: per-property suite names that read like a spec — `agent-session-tree-navigation.test.ts`, `agent-session-retry.test.ts`, `agent-session-concurrent.test.ts`, `bash-close-hang-windows.test.ts` (packages/coding-agent/test/).
- CI runs build + typecheck + full test on PR (`ci.yml:36-42`); `npm-audit.yml` for supply chain.
- Evals infrastructure in-repo (`packages/evals/src/harness.ts`, `docker.ts`, `report.ts`) but no workflow invokes it; no fuzzing.

### safety-enforcement — 6
- By design there is no built-in approval or sandbox, and pi is the most honest subject in the corpus about it: "the Pi coding agent intentionally does not have a sandbox" (`SECURITY.md:50`), with containerization patterns documented (`packages/coding-agent/docs/containerization.md`, README.md:44-48).
- What does bind: `beforeToolCall` blocking is enforced inside the loop (error result with reason at `agent-loop.ts:739-740`, tested in `packages/agent/test/agent-loop.test.ts`); project trust gates loading project settings/extensions/packages before first prompt (`core/project-trust.ts:24-29`, `trust-manager.ts`).
- Permission prompts themselves are examples, not defaults (`examples/extensions/permission-gate.ts`, `confirm-destructive.ts`). Standard design, tested, honest — 6; the honesty is why it is not 5.

### token-economy — 8
- Compaction with threshold trigger (`compaction/compaction.ts:289 shouldCompact`) and projection-based estimator measuring what the model would actually receive (`:919 estimateProjectedContextTokens`).
- Branch-summarization: abandoned session branches get summarized and attached on switch (`compaction/branch-summarization.ts` + `branch-summarization.test.ts`) — compaction as a branch operation, unique in corpus.
- Prompt-cache discipline: dedicated cache accounting and warming (`core/cache-stats.ts`, `core/cache-warmer.ts`), usage totals surfaced (`core/usage-totals.ts`).
- Deduction: no tiered fallback comparable to codex's token-budget fresh-window.

### orchestration — 7
- `packages/durable` (durable session storage/documents) + remote control plane `packages/{protocol,server,client}`; subagent pattern shipped as extension example (`examples/extensions/subagent/agents/worker.md`).
- Crash handling: `core/crash-log.ts`, compaction-queue tests survive interruption (`agent-session-auto-compaction-queue.test.ts`).
- Deduction: no first-class teams, queues, budgets, or scheduling — orchestration is intentionally left to extensions.

### interop — 6
- Strong headless contract: `json` and `rpc` app modes (`project-trust.ts:12 AppMode`), documented RPC protocol (`docs/rpc-commands.md`, `docs/sdk.md`), HTML session export (`core/export-html/`), rival-rules interop via example extension (`examples/extensions/claude-rules.ts`).
- The gap: no MCP and no ACP anywhere in core or packages (grep finds "mcp" only in vendored highlight.min.js). Deliberate, but interop anchors credit the surfaces, not the philosophy.

### operability — 9
- Session tree as an operating model: `/fork`, `/clone`, `/tree` navigation without losing branches (`docs/sessions.md:20-32`), resume + branch navigation each covered by tests (`agent-session-branching.test.ts`, `agent-session-tree-navigation.test.ts`).
- Diagnostics culture: `core/diagnostics.ts`, `settings-diagnostics.ts`, `crash-log.ts`, `bug-report.ts`, documented session file format (`docs/session-format.md`), keybindings/themes config.

### originality — 9
- Session-tree-with-branch-summarization (nothing else treats abandoned branches as first-class context material); cache warming as a core feature; extension-owned policy/providers/subagents; RPC-first remote operation. All verified in code (paths above).
- Deduction from 10: several of those exist in embryonic form elsewhere; pi does them most purely rather than being first for all.

### durability — 7
- Active full-history repo (6546 commits, 311 contributors), MIT, small institution (Earendil Works) around a strong solo vision; PR gate + contributor-approval workflows (`pr-gate.yml`, `approve-contributor.yml`).
- Bus-factor concern real: direction depends on one maintainer's philosophy (e.g., "no MCP" is a personal stance, not governance).

### docs-dx — 9
- 30+ in-repo docs incl. `how-pi-works.md`, `session-format.md`, `rpc.md`, `security.md`, `containerization.md`, `extensions.md` — reference + conceptual, matching code; SECURITY.md states scope limits honestly.
- Deduction: extension API onboarding still expects you to read example source.

## Verdict
**Strongest: architecture / operability / originality / docs-dx (9).** Weakest: interop (6) — no MCP/ACP by design.
Weighted 78.5 -> **A**, at the band floor. The A-band floor reference: you can reach A with zero sandbox if minimalism is coherent, tested to the teeth, honest about gaps (SECURITY.md:50), and the session/token model is genuinely better than everyone else's. Do not read this as "no enforcement is fine": cline at 76.5 has *more* default enforcement; pi wins on architecture, verification density, and originality.
