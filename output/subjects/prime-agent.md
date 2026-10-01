# prime-agent — T2 deep review

TypeScript monorepo, hard fork of pi (earendil-works/pi), Prime Intellect. Shallow clone (`.git/shallow` present, 1 visible commit referencing PR #2708): no activity or contributor claims possible from HEAD.

## Anchor question

Closest anchor: **pi**, unambiguously -- it is pi's direct child: 91 files hash-identical to /data/samples/agents/pi, same three-layer core (`packages/agent` loop / `packages/ai` provider layer / `packages/coding-agent` product), and README:120 credits pi explicitly.

## Provenance / fork delta vs pi (mandatory, rule (a))

Full-tree hash diff vs the frozen pi anchor copy (excluding node_modules):

- 91 files hash-identical (confirms census note), 435 modified, 713 prime-only, 1394 pi-only files absent from prime.
- Prime-only concentration: 532 files in `packages/coding-agent`, including entire new subsystems:
  `src/core/kernel/` (persistent Python kernel REPL; boot-gate.ts:7 boot-concurrency semaphore),
  `src/core/mcp/` (mcp-manager.ts, 734 LOC), `src/modes/acp/` (acp-mode.ts, 1,224 LOC),
  `src/modes/daemon/` (42 files: supervisor, worker/command recovery journals, rlm-ledger),
  `src/core/refinement/refinement.ts` (self-improvement proposals: prompts/memory/skills/subagents, :35-123),
  `src/core/rlm-runtime.ts` + `rlm-max-depth.ts` (recursive subagent registry with depth budget),
  `src/core/semantic-edges.ts` (causal request-edge ledger, :6-27), `src/core/cron-jobs.ts` (durable file-locked crons),
  `src/core/session-lease.ts`, `src/core/orphan-process-journal.ts`, plus `scripts/evals/` (43 files) and `prime-agent-runtime/` (Python kernel runtime).
- Fork-point lag: prime lacks upstream pi features that anchor pi's scores -- no `project-trust.ts` anywhere (find for `*trust*` = empty; pi anchor cites project-trust.ts:24-29), no `cache-warmer.ts`/`cache-stats.ts` in core (present in pi), and compaction lacks pi's projected-context measurement (`estimateProjectedContextTokens` exists only pi-side per compaction.ts diff).

Verdict: **divergent fork, not a sync-fork.** The delta is structural (new orchestration/daemon plane, ACP+MCP, eval gate, kernel), not cosmetic. Rule (a): scored on merit; ends up below upstream pi regardless (72.0 vs 78.5), so no demotion bookkeeping needed.

## Core surfaces read (line-level)

- **Core loop** `packages/agent/src/agent-loop.ts` (963 LOC): pi's loop plus fork-authored abort hardening -- `raceWithAbort`/`settlePostTurn`/`createAbortedAssistantMessage` (:28-176) make cancellation a first-class terminal state (aborted assistant message preserves partial content and usage). Blocking tool-call gate at :791-810: `config.beforeToolCall` result `{block, reason}` returns an error tool result; enforced before execution. No default policy installed at loop level (same philosophy as pi).
- **Compaction** `packages/coding-agent/src/core/compaction/compaction.ts` (1,005 LOC): usage-anchored token accounting (:186-207, trailing messages estimated after last real usage), threshold trigger `shouldCompact` (:221-224) `contextTokens > contextWindow - reserveTokens`, compaction-as-skill `COMPACT_SKILL_NAME` (:117), branch summarization (branch-summarization.ts, 410 LOC, :94 collectEntriesForBranchSummary). Weaker than pi's current rung-8 design: trigger is raw context vs window, not projected-model-context; no cache warming.
- **Permissions/safety**: no sandbox by default (same as pi); enforcement = blocking `beforeToolCall` hook + extension `tool_call` handlers (extensions/runner.ts:916). New fork-owned guard: destructive-git dirty-tree guard in `tools/bash.ts` -- pattern-detects discard commands (`git reset --hard`, `git checkout -- .`, `git clean -f`, `git restore .`) at :667-719, resolves `cd` chains and `git -C` targets per discard, **refuses when target is unresolvable** (fail-closed, :686-690), probes `git status --porcelain` only on pattern match, explicit two opt-outs (`allowDestructiveGit: true` arg or `PI_BASH_ALLOW_DESTRUCTIVE_GIT`, :156). Tested in `test/bash-destructive-git-guard.test.ts` (:152 refusal message, :176 bypass-and-discard). Differs from nanocoder's regex-denylist anti-pattern: probe-based, explicit-intent bypass, refusal messages enumerate dirty paths (:494-515).
- **MCP safety contract** `test/mcp-service-safety.test.ts:1-21`: adversarial regressions encoding a credential-binding contract (stored MCP credential whose endpoint mismatches the current URL must produce zero probes and zero token reads; cross-client stale-probe must not bless a rotated grant; offline, `globalThis.fetch` denied). Explicitly red-on-baseline discipline with passing control tests.

## Verification

- 338 `*.test.ts`, 145,184 LOC in *.test.ts alone (census test_loc 150,607 plausible once Python tests under `prime-agent-runtime/test` and `scripts/*/tests` are counted -- no glob miss this direction).
- CI `ci.yml`: trust/vouch gate (:19-32), build+check (:34-83), multi-package + sharded coding-agent test jobs (:85-113). `package.json:14` `check` chains biome + tsgo + `check:test-policy` (scripts/check-test-policy.mjs -- diff-based test-presence policy) + installer/push-guard/browser-smoke checks.
- **Behavioral eval gate**: `.github/workflows/behavioral-evals.yml` -- `pull_request_target` + exact `pre-release` label launches a 28-task base-vs-head comparison (15 SWE-bench Verified / 8 Pro / 5 ScaleSWE, scripts/evals/short_swe/README.md:3-5) on pinned models in isolated credential-free sandboxes; fail-closed verdict job (`behavioral-evals.yml:430` `test "$(cat results/verdict)" = pass`); label removal or head/base change revokes both durable statuses (:39-60); `scripts/evals/short_swe/` ships evaluator, oracle harness, gates, and its own tests. Plus `nightly-process-stress.yml` (daily) and `benchmarks.yml`.
- Per errata, this is the evals-in-CI evidence that unlocks 9 on verification; no fuzzing found (repl-kernel-protocol-corruption tests are adversarial fixtures, not a fuzzer), so not 10.

## Dimensions

| dim | score | best evidence |
|---|---|---|
| architecture | 5 | God files dwarf the errata docking factor: `agent-session.ts` 15,101 LOC (class AgentSession :1518) vs pi 4,023; `interactive-mode.ts` 12,614 LOC (:1211) vs pi's flagged 6,852. ~12% of non-test LOC in two files. Loop itself still separated (packages/agent, 963 LOC) and new subsystems are modularly filed, but the session/product core regressed hard vs the pi rung. |
| verification | 9 | 145k LOC tests; sharded CI test matrix `ci.yml:85-113`; test-presence policy gate `scripts/check-test-policy.mjs`; fail-closed base-vs-head SWE eval gate `behavioral-evals.yml:430` + `scripts/evals/short_swe/README.md:1-50`; adversarial security-contract suite `mcp-service-safety.test.ts:1-21`. Breaks the frozen verification-8 ceiling (errata: evals-in-CI observed). No fuzzing. |
| safety-enforcement | 6 | Tested blocking hook `agent-loop.ts:791-810` + dirty-tree git guard `bash.ts:667-719` with tests; but no sandbox, no project-trust gate anywhere (unlike pi), and SECURITY.md replaced pi's exemplary boundary-honesty text with corporate boilerplate (diff vs pi/SECURITY.md removes the trust-boundary paragraphs). Tested approval, nothing underneath = the 6 rung, guards and worse docs cancel. |
| token-economy | 6 | Usage-anchored estimation `compaction.ts:186-207`, threshold trigger :221-224, branch summaries `branch-summarization.ts:94`, compact-as-skill :117. Missing pi's projected-context measurement and all cache warming (`cache-warmer.ts` absent; fork predates them). One standard design, tested -- the 6 rung. |
| orchestration | 8 | Everything pi deliberately lacks: daemon supervisor + `worker-recovery-journal.ts`/`command-recovery-journal.ts` (recovery tested: command-recovery-journal.test.ts), `session-lease.ts` + test, `orphan-process-journal.ts` + test, durable `cron-jobs.ts` (file-locked, :1-15), RLM recursive subagents with max-depth budget (`rlm-max-depth.ts:3-13`) and ledger tests. Crash semantics are journal-tested, which clears cline's 8 rung; no codex-grade queue/SDK plane. |
| interop | 8 | Adds both surfaces pi rejected by philosophy: ACP mode `modes/acp/acp-mode.ts` (1,224 LOC) + docs/acp.md + acp-cold-cli/acp-events tests, MCP `core/mcp/mcp-manager.ts` (734 LOC) + 10 mcp-* tests + service catalog + MCP-over-ACP tool `tools/acp-mcp.ts`; retains pi rpc/json headless (docs/rpc.md). No published SDK under a prime-owned name -- workspaces still ship as `@earendil-works/pi-*`. |
| operability | 8 | pi's session-tree layer + daemon resume/roster (`modes/daemon/daemon-session-list.ts`, `agent-roster.ts` + tests), `native-update.ts`, incident notices (`cli/incident.ts`, incident-notices.ts), `session-resolver.ts`. Crush/cline rung. |
| originality | 8 | Real, tested, not marketing: semantic-edges-v1 causal request ledger with commit-gated edges (`semantic-edges.ts:6-27`), refinement auto-triggered self-edits across prompt/memory/skill/subagent scopes (`refinement.ts:35-123`), label-gated base-vs-head behavioral release gate (unique in corpus at this rigor), destructive-discard dirty-tree guard with cd-chain resolution, persistent Python kernel with boot-gate. Honest composition (comments cite nano-rlm, herdr) keeps it below 9. |
| durability | 6 | Corporate backing (LICENSE:4 "Copyright (c) 2026 Prime Intellect", org CI, security@ address) but shallow clone hides real cadence, no visible external contributors, and deep structural coupling to upstream: workspace packages still published as `@earendil-works/pi-*` (`packages/agent/package.json:2` etc.). |
| docs-dx | 8 | 37 files in `packages/coding-agent/docs` incl. prime-owned daemon.md, rlm.md, acp.md, mcp-integrations.md, long-running-agents.md, usage.md, quickstart.md; install.sh; SECURITY.md's eval-boundary section is precise. Docked for SECURITY.md losing the threat-model and README being mostly pi's. |

Weighted total: 5*1.5 + 9*1.5 + 6 + 6 + 8 + 8 + 8 + 8 + 6*0.5 + 8*0.5 = **72.0 -> B**. Not within 2 of a band boundary.

Strongest dimension: **verification** (9). Weakest dimension: **architecture** (5).

## Census sanity

- provenance_flag divergent-fork:pi: confirmed, and understated -- "91 hash-identical files" reads like near-parity, but the full diff shows 713 prime-only files and 435 modified; this is a substantive fork.
- `contributors: 1, commits: 1`: shallow-clone artifacts (HEAD commit cites PR #2708; `.git/shallow` exists). No dead/low-activity claim made.
- `test_loc: 150,607`: consistent with my count (145,184 across *.test.ts + Python tests). No glob miss.
- README quote "began as a hard fork of pi-mono" not found verbatim; README:120 has the attribution ("built on top of pi"). Substance stands.

## Calibration notes

- Rule (a) N/A in practice: divergent fork scored on merit lands at 72.0, below upstream pi's 78.5; no cap, no demotion to record.
- Verification 9 per errata guidance (9+ requires in-CI evals or fuzzing; behavioral-evals.yml:430 provides the workflow file:line).
- Architecture 5 applies the errata's >5k-LOC product-file docking factor against the subject itself, twice.
