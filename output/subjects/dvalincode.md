# dvalincode — T1 review

Subject: dvalincode v0.21.0 (`/data/samples/agents/dvalincode`)
TypeScript security-engineering agent ("every repair carries its own offline-verifiable
proof") with a general coding-agent core: CLI/TUI/web+server/Bun GUI/VS Code surfaces,
MCP server + governed MCP client, GitHub Action, headless harness mode.

## Anchor question

Closest anchor: **crush** -- same shape of genuinely tested safety machinery plus
distinctive runner-level ideas (stall detection, budgets) with thin orchestration;
dvalincode sits above crush on safety/originality and below on
orchestration/operability, landing two points above crush's 69.0.

## Census sanity

- `non_test_loc: 56530` is inflated ~2x. Measured first-party code: `src` 19,395 +
  `web` 6,631 + `editors` 1,333 + `integrations` 495 + `eval` 639 = **28,493**. The gap
  is consistent with counting `package-lock.json` files (10,241 lines), `docs/*.md`
  (5,718) and the html assets. `test_loc: 11300` matches measured 11,559 (`tests/`).
- Shallow clone (`.git/shallow`, 1 commit) -- activity from HEAD alone not asserted;
  HEAD is 7 days old and merged PR #252 (`git log`) indicates substantial upstream
  PR history. T1 tier stands under either count.
- Provenance: census says original; manifest name/repo/mcpName all self-consistent, no
  fork markers, no corpus look-alikes. Accepted.

## Architecture (8)

Clean layering, no god files (largest src file 652 LOC, `src/mcp/server.ts`):
state-machine turn orchestration (`src/agent/loop.ts:132-280`), inner tool loop
(`src/agent/runner.ts:138-390`), surface-agnostic session assembly
(`src/agent/session.ts:166-290`), providers behind adapters, and one tool chokepoint
(`src/tools/registry.ts:104-156`) where policy, approval, and audit taps all bind.
Loop/state model is separated from every product surface (the errata 8->9 test), but
docked from 9 for: dead pass-through states (`TurnState.SAVE`/`RESPOND` do nothing,
`loop.ts:243-251`), and a text-regex `@tool(name, {...})` fallback parser
(`runner.ts:399-411`) alongside native tool calls.

## Verification (8)

11.5k LOC of vitest specs that assert properties, not existence:
- policy algebra: "a repo source can only narrow, never widen, the machine source"
  (`tests/core/policy.test.ts:36-42`), order-independence (:44-49), unattended rank
  narrowing (:66-77).
- loop semantics: turn-wide tool budget not per-iteration
  (`tests/agent.test.ts:331`), mid-loop compaction keeps the turn alive (:287),
  first-edit deferral (:110), interrupted-turn state preservation (:349).
- crash recovery: `recoverSession` closes the dangling turn (`tests/sessions/journal.test.ts:53-55`).
CI (`.github/workflows/ci.yml`): test, Playwright web e2e, native shell + sandbox tests
on a 3-OS matrix (shell-cross-platform), MCP interop driving `dist/` with the reference
MCP client over a real pipe with `DVALIN_REQUIRE_MCP_INTEROP=1` so a missing build
cannot pass vacuously (ci.yml mcp-interop; `tests/mcp/clientInterop.test.ts:27-33`),
self-scan dogfood of the GitHub Action with `version: local`, VS Code job, CodeQL,
OpenSSF Scorecard, plus a weekly real-harness interop job
(`.github/workflows/harness-interop.yml:1-13`). SWE-bench eval harness exists
(`eval/swebench/`) but is not run in CI. Ceiling per errata: no in-CI model evals, no
fuzzing -> 8, not 9.

## Safety-enforcement (7)

Above the 6-rung "tested approval, nothing underneath", below codex:
- Narrowing-monotone org policy (`src/core/policy.ts:9-20` keystone docstring):
  every combinator only restricts (`narrow()` :198-231), canonical sorted resolution
  so the tamper hash is source-order independent (:152-157). Enforced at the single
  registry chokepoint -- tool deny, then per-target command/path checks before side
  effects (`src/tools/registry.ts:111-131`), mode/provider/model + `maxToolCalls` at
  session level (`src/agent/session.ts:203-279`).
- Unconditional hard blocks no policy can waive -- `rm -rf` roots, raw device writes,
  curl-pipe-shell, fork bombs -- matched per compound-command segment
  (`policy.ts:249-290`), plus allowlist-disqualifying constructs (substitution,
  redirection) at :293-307.
- Kernel subprocess network isolation via seatbelt/bwrap that **fails closed** when
  isolation is required and the platform can't provide it (`src/core/subprocessSandbox.ts:172-203`);
  sandbox binaries resolved from fixed system paths, never PATH (:207-215); same
  decision reused for long-lived stdio MCP sessions (:122-168).
- Approval gate binds in the registry (`registry.ts:134-141`) and is audited; policy
  denials are audited with rule names (`registry.ts:158-168`).
Honest about limits: policy is tamper-evident, not tamper-proof against a hostile local
admin (`policy.ts:16-19`); default posture is permissive (`permissivePolicy()` :104-117),
sandbox only activates under restrictive network policy. `bypass` permission mode exists.
Docked from 8: file writes are pattern-gated, not kernel-sandboxed; enforcement is
default-off in the out-of-box posture.

## Token-economy (7)

One compaction strategy but disciplined plumbing:
- Trigger estimates the *full serialized request* (messages + system + tool defs)
  (`src/agent/compact.ts:10-17`), checked at turn entry (`loop.ts:151-162`) and at every
  iteration boundary mid-tool-loop (`runner.ts:176-190`) with a monotonic anti-thrash
  guard (new compaction requires +1,000 tokens over the last one).
- Structured summary contract (Goal/Completed/Key decisions/Pending) with 64k transcript
  budget keeping first entry + tail (`compact.ts:25-84`); fallback keeps last 20 on
  summarize failure (`loop.ts:300-311`).
- Prompt-cache discipline: Anthropic `cache_control` ephemeral breakpoints on system,
  last tool def, last message block (`src/providers/anthropic.ts:49,70,77,220`); nudge
  messages are appended, never rewriting history, with the cache rationale stated in code
  (`runner.ts:231-233`). Tool results byte-capped before entering history
  (`runner.ts:281`). Per-run usage with cached/cache-miss/cache-write split recorded in
  `run_end` (`loop.ts:266-280`).
Not 8: single strategy, summary replaces all history (no tail graft), no warming,
char/4 estimation uncalibrated.

## Orchestration (5)

No subagents, no queues, no fork. What exists is durable and tested: append-only per-session
JSONL journal with turn_start/turn_end/interrupted, idempotent replay of completed turns,
crashed-turn recovery surfaced on resume, linked to the audit chain by ids + checkpoint
hash only (`src/sessions/journal.ts:5-46`; `session.ts:124,155-157`). Unattended headless
runs bound by policy ceilings -- permission-mode ceiling, iteration cap, wall-clock cap
(`src/harness/run.ts:196-222`, `policy.ts:63-69`). The dvalin remediation flow is a
pipeline (scan -> worktree -> fix -> verify), not multi-agent (`src/security/workflow.ts`).

## Interop (7)

- MCP server with 11 tools (`src/mcp/server.ts:259`), published to the MCP registry
  (`server.json`, `publish-mcp-registry.yml`), and a governed stdio MCP *client* that
  applies `checkCommand` itself because the registry chokepoint doesn't cover spawn
  (`src/mcp/stdio.ts:14-18,90`), running servers under the same sandbox plan.
- CI interop with the reference MCP client over a real pipe (see Verification); weekly
  real-harness job installing current Claude Code/Codex CLIs (`harness-interop.yml:21`).
- GitHub Action `action.yml` dogfooded in its own CI; VS Code extension with its own CI
  job; web UI + Bun GUI + TUI surfaces; headless harness contract with structured
  `HarnessRunResult` including explicit "unknown scan coverage" semantics distinct from
  "not scanned" (`src/harness/run.ts:38-52`).
- Cross-harness import: memory importer reads `~/.claude/CLAUDE.md` and
  `~/.claude/projects` (`src/memory/importers.ts:43,71-76`).
No ACP, no published SDK -> below cline's 8.

## Operability (7)

Session store + journal resume/recover; `/undo` executing tool-declared reverse ops
(`runner.ts:99-135`); tamper-evident audit per run with offline `verifyChain` and a
`report` command (`src/audit/log.ts:191`, `src/commands/report.ts:37`); `dvalincode trust`
prints the install's own posture -- resolved policy hash + sources, per-surface network
enforcement status, MCP server permissions, dependency list (`src/core/trust.ts:22-90`);
checksum-verified self-update distinguishing binary vs npm installs
(`src/core/selfUpdate.ts:8,108,139-157`). No git-state rewind/checkpoint; crash-recovery
posture is journal-scoped only.

## Originality (8)

Verified in code, not marketing:
- **Re-derivable Verified Fix Records**: verdict rests on observed exit codes + re-scan,
  carries the gate (threshold+mode) so a third party can reach the same verdict;
  schema-versioned with v1 verifying under v1 rules permanently; executor-agnostic
  (dvalin/codex/claude-code/copilot/human) (`src/security/fixRecord.ts:13-58`).
- **`trust` self-report** -- tool issues its own install-specific security posture
  (`trust.ts:22-32`).
- Narrowing-monotone canonical policy with un-raisable hard blocks (see Safety).
- Action-pin consistency meta-test with a concrete incident narrative -- one digest
  under version-comment drift caught by an offline test
  (`tests/actionPinning.test.ts:7-20`).
- Honest-interop discipline: comments in ci.yml/harness-interop.yml explaining that
  hand-verified claims went stale and why the automated version exists.
Not 9: ideas are novel but narrow-scope; no subagent/cache-warming class invention.

## Durability (4)

Contributors: 1 (census); shallow clone hides history, but 252+ PRs, dependabot,
CODEOWNERS, 8 workflows, release/GUI/registry pipelines and recent activity (HEAD
2026-09-22) indicate an active solo project, not dead. Bus factor 1, no institutional
backing, SECURITY.md exists but early-stage (self-declared) -- nanocoder's 4 rung.

## Docs-dx (5-dim: 8)

20+ in-repo docs that match code (THREAT-MODEL, EGRESS-THREAT-MODEL, POLICY-REFERENCE,
DURABLE-SESSION, AUDIT-TRAIL, GOVERNED-MCP, RECIPES-UNATTENDED, SKILLS), bilingual
README (EN + zh-CN), SECURITY.md, CONTRIBUTING, vitepress site with deploy workflow,
docs/APPROVABILITY-PLAN readable as the design spine. Docked from 9: no install-troubleshooting
/ error-message guidance pass, some docs are plan-shaped rather than reference-shaped.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | loop.ts:132-280 state machine; registry.ts:104-156 single chokepoint; max file 652 LOC |
| verification | 8 | policy.test.ts:36-42; ci.yml mcp-interop + 3-OS matrix; actionPinning.test.ts:7-20 |
| safety-enforcement | 7 | subprocessSandbox.ts:172-203 fail-closed; policy.ts:249-290 hard blocks; registry.ts:111-131 |
| token-economy | 7 | compact.ts:10-17 request estimate; runner.ts:176-190 anti-thrash; anthropic.ts:49-77 |
| orchestration | 5 | journal.ts:5-46 + journal.test.ts:53-55; harness/run.ts:196-222 ceilings; no subagents |
| interop | 7 | mcp/stdio.ts:14-18; ci.yml mcp-interop; harness-interop.yml:1-13; action.yml + CI dogfood |
| operability | 7 | trust.ts:22-90; audit/log.ts:191 verifyChain; runner.ts:99-135 undo reverse-ops |
| originality | 8 | fixRecord.ts:13-58 re-derivable proof; trust self-report; narrowing policy + hard blocks |
| durability | 4 | contributors=1 (census); CODEOWNERS/dependabot/PR#252; shallow-clone caveat |
| docs-dx | 8 | 20+ matched docs incl. THREAT-MODEL/POLICY-REFERENCE; zh-CN README |

**Weighted total: 71.0 -> band B.** Strongest: verification/originality tie at 8 (call it
originality -- the fix-record and trust-report ideas are corpus-unique; verification's 8
is the standard errata ceiling). Weakest: orchestration (5).

Boundary check: 71.0 is not within 2 pts of 65/78/88/45.

Calibration: no rule (a) applicable (not a fork). No demotions.
