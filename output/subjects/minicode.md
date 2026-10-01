# minicode — T1 review

- **Subject:** minicode (`minicode-ai` v0.12.0), /data/samples/agents/minicode, upstream `startupmini/minicode`
- **Identity check:** manifest `minicode-ai` (package.json:2); NOT mini-kode (minmaxflow/mini-kode), mocode, or nanocoder. All facts below from this directory only.
- **Census sanity:** census non_test 51,743 / test 40,790; my cloc: test/ = 41,214, total repo code = 93,118, non-test ≈ 51,904. Census is accurate. Shallow clone (1 commit, "1 contributor") is a clone artifact; CHANGELOG has 48 releases, latest 0.12.0 dated 2026-09-25, HEAD 2026-09-26 - clearly active. No dead/low-activity claims needed or made.
- **Provenance:** census says original; confirmed. Not a fork. Distinctive: the agent kernel ("MiniCore", 1,639 LOC) lives in a sibling repo and is vendored into `vendor/minicore/` via subpath imports `#minicore` (package.json:44-46), pinned by commit + hash (vendor/minicore/VENDOR.md:5-10), with CI enforcing sync (`vendor:check`, ci.yml:44-46). Same-org split, not an upstream derivative; rule (a) N/A.
- **License:** MIT (file + manifest). Clean for concept and code-adjacent consideration.

## Anchor question

**Closest anchor: crush (69.0, B)** - same profile of single-org strong-engineering CLI with tested permission gates, a resume journal, and distinctive adversarial harnesses; minicode sits above crush on architecture/verification density (no 2.4k-LOC fused loop file; test:src ratio 1.25) but below pi/cline on ecosystem surface. Scoring at 72.5, between crush (69) and pi (78.5), nearer crush.

## Architecture (8/15)

- Frozen 1.6k-LOC kernel loop `vendor/minicore/src/core/loop.ts` (355 LOC): snapshot-on-append aliasing guards (loop.ts:214-222 "never write late results back into the shared store"), tool-call/result pairing validation with name-mismatch errors (loop.ts:270-289), dual compaction flags with documented conflation-bug history (loop.ts:44-51), strict stream-contract enforcement (post-finish events rejected, loop.ts:104-110; missing finish reason = retryable error, loop.ts:136-140).
- Product layer with DI from a composition root; layer rules enforced by tests: `src/` non-ui cannot import `src/ui/`, `src/ui/` cannot import `#minicore` (AGENTS.md:20-24, test/ui-boundary.test.ts), plus `architecture-map.test.ts` drift gate.
- No god files: max = src/ui/tui/app.ts 1,411 LOC; loop/state fully separated from every product surface (the errata's 9-rung criterion is structurally met). Docked from 9 because the separation rests on a 3.5k-LOC vendored kernel whose seams were patched from outside (the "frozen + additive seam" is disciplined but the kernel/product boundary is a published-artifact contract, not in-repo evolution) and the presentation stack (adapter 1,041 + reducer 806 + projection 662) is a heavy self-built event pipeline.

## Verification (8/15)

- 203 test files, 41.2k test LOC vs ~33k src LOC (1.25 ratio). Names assert properties, not existence: permission-matrix, permission-adversarial, tool-toctou, live-toctou, ssrf-guard, concurrency-cross-process, journal-verify, recovery-idempotency, task-invariants, sandbox-failclosed.
- CI (ci.yml, PR + push, Linux AND Windows jobs): vendor:check (:44-46) → tsc → lint → bun test → coverage gate (:62-64) → JS-grep-engine rerun (:68-72) → **bash bypass probe in both auto and allowlist modes (:77-83)** → pack gate (:92-94) → fake-provider bench smoke (:96-98) → 60-check harness audit (:101-103) → telemetry gate (:105-107). Windows job reruns the probe (:151-159).
- Nightly (slow.yml): multi-seed seeded-PRNG bash fuzz (:28-34, extreme-bash-fuzz.ts:17-34 with `--seed` reproducibility), shadow-git stress 1500 files x 8 sessions (:35-37), adversarial MCP server (:39-41).
- Eval harness exists (bench/eval-gate.ts, swebench_lite_20.jsonl + era manifest tested daemonlessly, swebench-docker.test.ts:11-30) but real model evals are not run in CI. Per the frozen errata, fuzzing evidence exists (slow.yml:28-34) but it is single-target scripted fuzzing, and no model evals in CI - holds at 8, does not clear to 9.

## Safety-enforcement (7/10)

- Permission handler is data-driven per mode (permission.ts:262-378); kernel calls it before execution and cancellation beats late approval (permission.ts:396-401).
- **Jail binds before every mode including `allow-all`** (permission.ts:379-441, 483-497): realpath-outside-root + sensitive-path rejections run first; allow-all still runs the FULL bash-guard ("izin penuh bukan berarti rm -rf /"); owned-state `.minicode/` writes denied with a realpath double-key against internal symlink/junction escape (permission.ts:443-479, jail.ts:44-60). Deny-reason channel makes denials actionable for the model (permission.ts:220-250).
- Bash guard is normalize-then-match (bash-guard.ts:1-19): quote stripping (:38-46), cmd.exe caret escapes with quote-awareness (:60-88), simple-var inlining with rescan for chained indirection (:90-120); header states its limits honestly ("statical analysis over a Turing-complete language; real isolation needs an OS sandbox").
- Sandbox: seatbelt/bwrap/docker (sandbox/os.ts, sandbox/docker.ts) with fail-closed on explicit request without backend (tools/bash.ts:48-56,253-262; sandbox-failclosed.test.ts:28-40) - the inverse of nanocoder's fail-open. BUT default posture: bash runs direct, guard-only; OS sandbox is opt-in via MINICODE_SANDBOX. No kernel-level enforcement by default anywhere.
- Above the 6-rung (approval + tested + something underneath that binds even under allow-all), below the 8-rung (enforcement under approvals is static/string-level by default, not OS-level). Docs honesty exemplary (docs/security-model.md:1-30: "Tanpa kata 'aman' absolut", explicit allow-all caveat table).

## Token-economy (7/10)

- Budget pressure tiers at 0.75/0.9/1.0 (vendor budget.ts:15-24) include system+schema fixed cost in the estimate (loop.ts:60-66, single estimator loop.ts:349-353).
- Compaction (src/policy/compaction.ts): LLM strategy with sync-mechanical fallback the loop enforces ("an async compactor must never crash the loop", loop.ts:336-355); anti-thrash breaker disables LLM path after 2 sub-10%-reduction rounds (:95-120); prior summary pinned verbatim, never re-summarized (anti drift, :207-217); summary hard cap 1,500 chars with asymmetric truncation directions and lone-surrogate safety (:169-215); compaction input scrubbed of secrets and fenced against prompt-injection persistence (:262-276); compaction LLM spend reported into the session budget bus (:31-40, 293-305); SSRF-checked compaction endpoint (:229-233). CHANGELOG:5-8 documents the measured 53.9x growth fix.
- Prompt-cache marking on Anthropic only: system + last-message ephemeral breakpoints (providers/anthropic.ts:76, 352), cache read/write tokens priced into cost (test/prompt-caching.test.ts:5-26). No cache warming, no cache-aware compaction ordering, nothing for openai-compat. Two strategies + accounting ≈ cline's 7; pi's 8 needs cache discipline as a feature.

## Orchestration (7/10)

- `delegate_task` subagents with an abort-aware concurrency-3 pool (agents/pool.ts:1-42), child sessions isolated via DI-injected session factory; plan mode forces children read-only (permission.ts:270-280).
- Crash/recovery story is real: mutation intent journal (see findings) + verify-first resume, never auto-redo (journal.ts:1-22); recovery-idempotency/recovery-side-effects/redo-recovery tests.
- Budgets: per-run `--budget` in ACP/exec, ratelimit rpm, cost tracking with pricing table. No loop/repeated-action detection anywhere (kernel guards retries only); no queue/daemon plane. crush-level 7.

## Interop (7/10)

- MCP client (stdio + Streamable HTTP/SSE, tools/resources/prompts, dynamic `serverid.toolname` tools) AND an MCP server (src/mcp/, 538-LOC server + mcp-http/mcp-resources-prompts/mcp-server tests). LSP client (diagnostics/definition/references/hover/symbols, didClose cleanup).
- ACP server exists but is an explicitly-documented 4-method subset, honest "BUKAN klaim kompatibel ACP penuh" (cli/commands/acp.ts:20-24) - not nanocoder-grade ACP commitment.
- Headless `exec --json` JSONL + summary (exec-json-envelope.test.ts). No published SDK, no IDE plugin. crush-level 7.

## Operability (8/10)

- sqlite session persistence with incremental save + fidelity tests (reasoning/is_error columns restored on resume, CHANGELOG:33-38); session branch (session-branch.test.ts); checkpoints via shadow-git (see findings) - more principled than cline's git-stash because the user's index/HEAD are provably untouched (shadow-git.ts:23-27).
- Journal-aware resume with degraded-mode advisory path (journal.ts:18-22); doctor, exit-codes, non-tty output contracts, stdin-flow tests; runtime mode switching (Shift+Tab) and config wizards; rich actionable deny reasons.
- 8-rung fit: recover/resume semantics designed AND tested, though no explicit crash-telemetry posture beyond degraded-mode warnings.

## Originality (7/10)

Real, code-verified ideas not seen in anchors:
1. Frozen-kernel two-repo split: kernel published into its own repo, vendored here behind a freeze + additive-seams-only contract with CI hash-pin sync (VENDOR.md:5-14, ci.yml:44-46, no-frozen-runtime-value.test.ts).
2. Mutation-intent journal that answers exactly one question ("is mutation X committed?") with explicit non-goals written down (journal.ts:14-17).
3. Shadow-git checkpoints pinned as tree refs in `refs/minicode/` - invisible to `git log --all`, gc-safe, O(delta) undo (shadow-git.ts:16-27).
4. Bypass-probe as a CI measurement harness: every denylist rule carries paired must-deny/must-allow cases, run on Linux and Windows per PR (experiments/bash-bypass-probe.ts:11-13, ci.yml:77-83,158-159).
5. Evidence-gated task completion: `completed` requires injected verify evidence; failed verify demotes to `blocked` (todo.ts:51, setup.ts:1184).
Not field-defining (9) - each mechanism is a strong variant, none redefines the shape - but clearly above crush's 7 rung? crush at 7 had loop-detection + call-scoped grants; minicode's journal + shadow-git + probe ensemble is at least equal: 7.

## Durability (5)

- Single-org (startupmini) solo-appearing project, but shallow clone hides real contributor counts (artifact noted, not asserted either direction). Very active: 48 CHANGELOG releases, latest 0.12.0 on 2026-09-25; public npm publish workflow (publish.yml); security docs present (docs/security.md + security-model.md, though no root SECURITY.md); no external institutional backing evident. Between nanocoder 4 and pi 7; closer to 4-5: take **4** (solo bus factor, young, no governance artifacts).

## Docs-DX (5)

- 40+ in-repo docs including security-model, TERMINAL_CONTRACT (frozen output contract read before touching rendering, AGENTS.md:5-7), troubleshooting, getting-started; ARCHITECTURE.html kept honest by the architecture-map test (docs cannot drift from files - rarer than pi's manual discipline).
- README explicitly refuses uncountable marketing numbers (README.md:7: machine-countable claims "tidak ditulis di sini"). Install = `npm i -g minicode-ai`.
- Heavy Indonesian for docs/comments/CHANGELOG - real onboarding cost for non-Indonesian contributors; nanocoder's 7 is trilingual, minicode is not. 7.

## Totals

| dim | score | w | weighted |
|---|---|---|---|
| architecture | 8 | 15 | 12.0 |
| verification | 8 | 15 | 12.0 |
| safety-enforcement | 7 | 10 | 7.0 |
| token-economy | 7 | 10 | 7.0 |
| orchestration | 7 | 10 | 7.0 |
| interop | 7 | 10 | 7.0 |
| operability | 8 | 10 | 8.0 |
| originality | 7 | 10 | 7.0 |
| durability | 4 | 5 | 2.0 |
| docs-dx | 7 | 5 | 3.5 |
| **total** | | | **72.5** |

**Band: B** (65-77). Strongest: verification (with architecture at 8; verification named for the 1.25 test:src ratio, oracle probes wired into PR CI on two OSes, and nightly seeded fuzz). Weakest: durability (4, solo bus factor).

Boundary check: 72.5 is not within 2 pts of 65/78. No demotions applied; no calibration records needed.
