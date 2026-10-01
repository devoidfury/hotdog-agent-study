# Boundary re-review: codebuff (provisional 67.0, B, floor margin +2.0)

Scope: settle (1) verification rung 5-vs-4 and the general rule for test mass without
execution; (2) mirror covariate for durability/operability/verification; (3) the
token-economy 9 claim. Protocol: read-only; anchors + ERRATA applied; verification-9
ledger (evals-in-CI or fuzzing required for 9+) respected - codebuff is nowhere near 9.

Closest anchor: **crush (69.0)** - a funded company monorepo with solid single-standard
designs and decent-but-uneven verification; the gap is that crush's tests actually run
in CI (`build.yml:29-30` per anchors) while codebuff's entire test corpus is inert in
the public tree.

## Provenance (established first, because it gates everything)

- Single squashed commit: `dfa45d2 "Sync public snapshot from freebuff-private"` - the
  census `commits: 1 / contributors: 1` are export artifacts, not reality.
- `CONTRIBUTING.md:3`: "This repository is a public mirror of the Freebuff/Codebuff
  source tree. The private repository is the source of truth" - accepted PRs are
  hand-ported private and re-exported (`CONTRIBUTING.md:63-67`).
- `pr-hygiene.yml:124-131`: FORBIDDEN paths (`web/`, `packages/internal/`, billing,
  bigquery) prove the mirror boundary is mechanically enforced.
- `docs/testing.md:30-35` describes a real private test matrix and a disappearing-tests
  guard (`scripts/ci/test-with-guard.ts`, `.github/test-baselines.json`) - **neither file
  exists in the public tree**; `scripts/` contains only `tmux/`. So the docs honestly
  describe out-of-tree CI, which per evidence rules earns nothing here.
- Remote-verified (GitHub API, 2026-09-30): repo alive, `pushed_at 2026-09-30T15:10Z`
  (same-day sync cadence), not archived, Apache-2.0, 13,009 stars, 1,390 forks, 317
  open issues, org CodebuffAI, homepage freebuff.com. Product-backed (Stripe/PostHog
  keys in `ci.yml:43-50` placeholders).
- Honest disclosure throughout (`ci.yml:5-9` even admits CI "never reached the public
  repo" while CONTRIBUTING claimed validation). No license-risk; not a leaked snapshot.

## (1) Verification: 5, not 4, and the general rule

Corpus: 553 files / **151,832 LOC** of colocated tests (cli 225/62,334; common
153/31,529; agent-runtime 67/24,334; sdk 76/21,498). Sampled across areas - the tests
are property-asserting, not existence-checks:
- `compact-request-budget.test.ts:47+`: "preserves every call/result in a parallel
  batch and the current prompt" - an invariant, not a snapshot.
- `compact-context-loop.test.ts:46+`: drives the real `loopAgentSteps` with a scripted
  summarizer and asserts what the model actually sees (warm-cache no-op :275, cold-cache
  compact :286, per-agent TTL :324, once-per-turn :355, null-TTL opt-out :478).
- `model-compaction.test.ts:357-402`: fallback ladder + telemetry event assertion.
- `context-pruner-parity.test.ts` (474 LOC): runs the duplicated pruner algorithm and
  the runtime copy over the same histories, fails on drift.
- `agent-dir-trust.test.ts:31+`: trust gate behavior incl. non-interactive refusal.

Execution: public CI runs **build + binary smoke only** (`ci.yml:35-55` - `build:sdk`,
`build:freebuff`, `smoke-binary.ts`), and the root `ci` script is itself build-only
(`package.json:29`). No workflow runs `bun test`. `evals/` (buffbench, runner + eval
JSONs) is wired to nothing. No fuzzing anywhere. `smoke-binary.ts:1-25` is a real
executed check (spawns the compiled binary, asserts the boot screen renders, catches
hangs/segfaults across OSes) - more than nothing, far less than the corpus.

Ruling: **verification 5** per the claw-code-agent row (property corpus + zero CI
execution = 5). Not 4: 4 is for thin/happy-path tests with known gaps - this corpus is
dense, property-asserting, and runnable-in-principle (only 1 test file references the
absent `@codebuff/internal`; preloads and fixtures are in-tree: `cli/bunfig.toml`,
`common/src/testing/`). Not 6: the 6 rung's "tested" means the corpus gates changes; a
test file that no workflow runs provides no enforcement signal in-tree, and mass alone
cannot buy the rung.

**General rule (test mass without execution):** mass buys quality-of-the-corpus credit
(the 4-to-5 axis); execution is what crosses 5-to-6, independent of size. An executed
build+smoke layer raises confidence but not the rung. Private/out-of-tree CI claims are
zero-credit/zero-blame under the mirror covariate - never infer green runs from docs
(`docs/testing.md:32`), and never treat the mirror's silence as evidence the tests fail
either. Broken-as-configured (amazon-q: job runs with a defective filter) is *above*
never-runs, because a defective gate still gates.

## (2) Mirror covariate: needed, but inverted from openhands

openhands: the core (loop, compaction, policy) lives out-of-tree - zero credit, zero
blame, in-tree presence as ranking covariate. codebuff is the opposite case: the agent
substance **is** in-tree and auditable - loop (`run-agent-step.ts`, 1,631 LOC), the full
compaction plane, 40 one-file tool handlers, trust gate, session stores, SDK. What is
out-of-tree is the **process plane**: commit history, contributor graph, review cadence,
bus factor, the test matrix and guard, and production crash behavior.

Ruling: no zero-credit/zero-blame on the core (unlike openhands, most of it is here and
gets scored on file:line). Apply a `disclosed-private-mirror` marker instead:
- durability: activity and backing are honestly creditable from remote checks (same-day
  export, 13k stars, funded consumer product, SECURITY.md, Apache-2.0); bus factor and
  governance are **unassessable** from a squashed mirror - cap at 6, do not extrapolate
  crush's 8 (CLA, release cadence, funded-institution signals are absent/hidden here),
  and do not apply claw's 3 (claw is solo-hobby; codebuff is a company with a live
  product).
- operability: score shipped surface only (chat-history resume screen,
  `process-diagnostics.ts:1-25` snapshot incl. watchdog + active tool pids, BYOK/usage
  commands, 0600 trust store). Crash-recovery posture in the private deployment: no
  credit, no blame. 6.5.
- verification: as ruled above - the corpus is scored in-tree for quality; its (possible)
  private execution is invisible to both sides of the ledger.

## (3) Token-economy: 8, not 9

Real, tested machinery:
- Cache-expiry-gated opportunistic compaction: `promptCacheGapMs` (compact-history.ts:
  1035-1060) measures idle gap last-assistant-to-live-user; `evaluateCompactionTrigger`
  (:1066-1110) fires only past the per-model TTL AND a token floor, because past TTL
  the next request re-reads everything at cold price anyway - compaction shrinks the
  cold prefill for free. Per-model policy in `compaction-policy.ts:21-63` (1h/140k
  default; 15min/40k for the DeepSeek lane). This is field-unique (converged with
  gptme, codebuff-1).
- Ladder: model handoff (`model-compaction.ts:316-384`) -> mechanical pass on any
  summarizer failure (`:336-380`) -> honest no-op; the "a compaction failure must never
  fail the turn" invariant at `run-agent-step.ts:1195-1197` and is tested at both
  `model-compaction.test.ts:357` and `compact-context-loop.test.ts:202,250`.
- Usage-anchored accounting: `token-counter.ts:6-7` (loop anchors context size to each
  response's usage; never BPE-tokenizes); compaction spend attributed, not fed into the
  next request's context (`run-agent-step.ts:1229`); cost aggregation test exists.
- Cache discipline: ephemeral `cache_control` marks (`common/src/util/messages.ts:60`);
  MCP tool names kept byte-identical when legal to avoid prompt-cache churn
  (`mcp.ts:24-33`).

Why not 9 (against codex 9 / jazz 8.5):
- The "measured" part of the claim is half out-of-tree: the tuning numbers (87% strip
  rate, 40x cold-token price, 15-min Luminal TTL) cite `freebuff-costs.knowledge.md
  (private)` (`compaction-policy.ts:47-62`). Documented measurement, not inspectable.
- The tests that would defend the plane never execute publicly (section 1); codex's and
  jazz's comparable tests run in CI.
- Plane depth below codex: no image budgets, no pre/post-compact hooks, no cache
  warming, single summarizer rung (codex has three tiers incl. a no-LLM fresh window).
  The cache-expiry trigger is the one element others should copy - that earns it a
  portable finding (already logged as codebuff-1), not a 9 lane.

**8** = "defended: properties enforced AND tested" holds on the strength of in-tree
readable tests (mirror covariate: zero blame for their non-execution); 9 requires the
codex breadth plus visible proof of running. Delta vs provisional: -1.0.

## Lane summary (boundary recount)

architecture 7.5 (clean packages, 1,631-LOC loop, 40 tool-handler files, no product god
file; docked for chat.tsx 2,317 fuse, the acknowledged context-pruner duplication -
albeit parity-guarded - and the dual codebuff/freebuff product modes). verification 5.
safety-enforcement 6 (tested dir-trust gate naming its own RCE vector,
`agent-dir-trust.ts:12-30`; permission check before tool emission,
`tool-executor.ts:495,733`; zero OS-sandbox in-tree - hosted product). token-economy 8.
orchestration 6.5 (spawn_agents + inline spawn + permission tests, queued-prompt store,
todo-loop; no journals/resume-fork/loop-detection). interop 7 (MCP client with
hash-stable name sanitization + tests, published @codebuff/sdk 0.10.7, composio; no
ACP, no IDE surface). operability 6.5. originality 7.5 (cache-expiry compaction,
parity-guarded duplication, PR-hygiene thresholds replayed against the last 300 real
PRs `pr-hygiene.yml:55-68`, injection-sanitized echo :68-76). durability 6 (capped,
see covariate). docs-dx 5.5 (only 2 files in docs/ but testing.md/CONTRIBUTING are
exemplary; SECURITY.md is an email line; feature docs out-of-tree).

Weighted: 11.25+7.5+6+8+6.5+7+6.5+7.5+3+2.75 = **66.0, B**. Floor held with 1.0 margin;
the token-economy 9->8 is the entire move from 67.0.
