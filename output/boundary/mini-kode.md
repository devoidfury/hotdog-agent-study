# Boundary re-review: mini-kode (T1, 2026-09-30)

Provisional 46.5 C (1.5 over the C floor). Dispatch swing: verification 5-vs-4 (zero-CI cap +
runner-blind screening). Full ten-dimension pass re-run; safety holes confirmed at file:line.

**Closest anchor: nanocoder (60.5 C).** Both: real-but-leaky permission machinery with no OS sandbox,
one-standard-summarize compaction, no model evals, solo-team C shape. mini-kode sits ~15 below
because its grant-matching layer is bypassable from inside the threat model (self-grant) and its
orchestration/interop board is thinner. Not close to codel (23): mini-kode's loop, tests, and
enforcement are all real.

## 1. Verification: 5 DEFENSIBLE, 4 and 6 both rejected

**Zero CI confirmed.** No `.github/`, no `.gitlab-ci.*`, no `.circleci/`, no workflows of any kind
(`find` for yml/yaml returns only `pnpm-lock.yaml`). The only test gate is local
`prepublishOnly` (`package.json:48`) - runs at publish time on the author's machine, not per-push.
Zero-CI cap applies (claw-code-agent precedent, applied at ferrum).

**Runner-blind screening: PASSES.** `test` is `vitest run` (`package.json:45`) with no
`vitest.config.*`/`vite.config.*` anywhere, so vitest's default include
`**/*.{test,spec}.?(c|m)[jt]s?(x)` matches all 32 co-located test files (28 `.test.ts` + 4
`.test.tsx`; `src/ui/components/ToolCall.test.tsx` included). Binharic import-graph check also
passes: 32/32 test files import production modules (0 replica tests). 4,851 test LOC, 276
`it/test` cases. This is NOT the groq-code-cli shape (npm test sees everything) - the 4-rung
reading fails.

**Why not 6 (ferrum = top of the zero-CI cap):** the corpus is real but property-thin exactly where
it matters. The project's own documented security example - `pathChecker.ts:33-35` asserting
`isPathUnderPrefix('/home/user-project/file.txt', '/home/user') => false` - is FALSE for the
implementation at `pathChecker.ts:53` (bare `startsWith`), and no test covers the sibling boundary
(grep for `user-project|foobar` across all tests: 0 hits). Loop tests mock the tool runner wholesale
(`src/agent/executor.test.ts:30-70`), so end-to-end loop properties rest on mocks. No fuzzing, no
evals, single runner. 5 = "works on happy paths, known gaps" and matches the claw-code-agent
zero-CI rung; 6 requires ferrum-class property matrices this lacks.

## 2. Safety: 4 HELD (not 3, not 5). Laundering verdict: mechanism CONFIRMED, aggravation REJECTED

Shipped default posture is honest and binding: `default` approval mode prompts for bash
(`src/tools/bash.ts:167-176`), fs writes (`src/tools/fileEdit.ts:122-129`), and MCP tool calls
(`src/mcp/tools.ts:154`); yolo/autoEdit are explicit opt-ins (`src/cli.ts:55-61`); approval prompts
fail closed on 5-min timeout (`src/permissions/permissionRequest.ts:101-110`) and non-interactive
mode hard-fails any prompt (`src/nonInteractive/runner.ts:98-106`). Shipped-default doctrine is
satisfied - which is why 5-rung candidates (octomind-style "machinery exists") were considered.

Why the grant engine cannot reach 6 - four independent breaks, all inside the default mode:

1. **Sibling-prefix FS grant** (`pathChecker.ts:53`): `normalizedFile.startsWith(normalizedPrefix)` -
   a grant to `/foo` admits `/foobar`. The function's own doc example (`pathChecker.ts:33-35`)
   promises `false` for exactly this case; docs/permission.md:104 still advertises "prefix matching:
   verifies target paths are under authorized prefixes". Untested. (b1, concept
   workspace-path-containment, x4 with claii/darce/groq.)
2. **Policy-file self-grant by the model**: `checkFsPermission` auto-approves all writes under
   `cwd + "/.mini-kode"` (`policyResolver.ts:143-145`) to let todoWrite work - and the grant store
   lives at `cwd/.mini-kode/permissions.json` (`utils/paths.ts:33`). fileEdit therefore writes the
   permission policy with no prompt; `fileManager.ts:52-72` accepts any well-shaped grant including
   `{"type":"bash","command":"*"}`, honored at `policyResolver.ts:180` and unioned
   ("If ANY policy grants access, the operation is allowed", `policyResolver.ts:22`). One no-approval
   fileEdit = permanent global bash. No test covers it. (b2.)
3. **Clone-and-run MCP**: project-scoped `.mini-kode/mcp.json` (`config/manager.ts:429`) is read and
   stdio-spawned at app mount (`ui/App.tsx:52` -> `mcp/index.ts:99` -> `mcp/client.ts:184`) and in
   headless runs (`nonInteractive/runner.ts:79`). No trust gate - opening a cloned repo spawns
   attacker-chosen processes. Same sub-family as memcode/waveloom/zap (b3, hook-trust-scoping x4+);
   the repo can also ship pre-baked `permissions.json` grants, accepted ungated (same loader,
   `fileManager.ts:29-77`).
4. **Compound-command grant ride**: `extractMainCommand` returns the LAST non-setup segment
   (`commandParser.ts:28-36`), so under a `git:*` grant, `bash -c 'evil' && git status` matches and
   the whole command executes without a prompt. The blacklist cannot catch it: `bash` is not banned
   and the matcher keys on the first token only (`commandValidator.ts:125-128`) - `/usr/bin/curl` or
   `bash -c 'curl ...'` pass trivially (regex-denylist convergence, x5+). (b4.)

**Laundering verdict (dispatch question):** the mechanism holds as described - the bash blacklist
bans curl/wget/nc explicitly to stop "data exfiltration" (`commandValidator.ts:33-58`), while
fetch.ts is `readonly: true` (`tools/fetch.ts:53`) and readonly tools "should never require
permission approval by design" (`agent/toolExecutor.ts:147`), giving unapproval'd network egress
beside unbounded reads (`fileRead.ts:75-77` reads any absolute path outside cwd, by design, with
only a code comment). But it does NOT pull safety to 3, for two reasons: (a) local reads were never
denied in the first place, so fetch does not "launder around a denied read" - the deny-set was never
closed over the read/egress effects; the honest framing is the deny-set is decorative against
exfiltration, which is already priced into the 4; (b) the posture is documented-honest (fileRead's
comment states the choice), not misleading - per the confirmed honesty doctrine, absence-of-mechanism
caps the rung, and mini-kode's mechanism is real-but-leaky = rung 4 exactly. Same rung as claurst
(4), above forge's hidden-off, below nanocoder's real-if-opt-in jail (5): machinery binds by default,
but any of four paths dissolves a grant boundary silently.

## 3. Ten-dimension recount

| dim | score | basis |
|---|---|---|
| architecture | 6 | Genuinely clean for 9k LOC: UI-agnostic loop `agent/executor.ts` shared by interactive + headless, uniform Tool interface, no god files (largest: executor 494, policyResolver 449). Docks: enforcement distributed inside tool bodies (the `readonly` flag IS the permission model), module-global session-grant singleton (`permissions/session.ts:31`), 862-LOC useAppState. Loop tested (mocked). nanocoder-5 has parallel per-surface loops; crush-7 has god files - 6 fits. |
| verification | 5 | See section 1. Cap held, runner-blind cleared. |
| safety-enforcement | 4 | See section 2. |
| token-economy | 5 | Usage-measured trigger on real API usage (`executor.ts:116`) + full-summarize auto-compact + /compact + token display = the nanocoder-6 shape minus quality: hardcoded 115000 threshold assumes a 128k window while the shipping DEFAULT preset is deepseek-chat (`config/manager.ts:52-58`), whose window is smaller - on the default provider auto-compaction is unreachable and the API errors first (b6). Zero cache discipline, zero cost visibility. |
| orchestration | 3 | No subagents, no queue, no budgets, no loop detection - and an explicit refusal: "No hard iteration limit" (`executor.ts:108-110`), an uncapped LLM loop. Below codel-4 (which at least ships a durable task queue). |
| interop | 4 | MCP client, stdio + streamable-HTTP, grant-gated and tested (`mcp/client.test.ts`), progress state in UI; headless mode with a classified exit-code contract (`executor.ts:57-63` header, `nonInteractive/runner.ts:140`); multi-provider OpenAI-compatible presets. No server, no ACP, no SDK. |
| operability | 4.5 | Config 3-layer priority, slash commands (/compact /clear /init /mcp), error taxonomy, honest KNOWN_ISSUES.md. Dock: sessions persist (`sessions/persistence.ts:28`) but `loadSession` has ZERO production callers and no --resume flag anywhere - orphaned persistence (b7). No rewind, no diagnostics beyond console. |
| originality | 3 | Deliberate educational clone of the kode shape; @-mention fuzzy completion and planModel dual-model split exist elsewhere in corpus. No corpus-new mechanism found in code. |
| durability | 4 | Solo author, MIT, npm-published with git-tag release flow (`package.json:57-58`), remote-verified ~11mo quiet, not archived - rule (b) not triggered (dispatch instruction + HEAD 2025-11-04 consistent). Zero CI, no SECURITY.md, bus factor 1. nanocoder-4 is young-churn; this is settled-dead-ish - 4. |
| docs-dx | 5.5 | 7 topic docs (architecture/permission/tools/config/ui/llm-integration + docs/README), working install path, AGENTS.md/CLAUDE.md, honest known-issues. Dock: permission.md's security claims (":104 prefix matching verifies") overstate a matcher that contradicts its own docstring. |

**Weighted total: 6*1.5 + 5*1.5 + 4 + 5 + 3 + 4.5 + 4.5 + 3 + 4*0.5 + 5.5*0.5 = 45.25 → C.**

## 4. Band ruling and sensitivity (disclosed, not silent)

Strict alternative (token 4.5 for the miswired default-provider trigger, interop 4) = 44.25-44.75,
technically D. I ruled the floor HELD at 45.25 on band-coherence, the plandex-convention logic
("would a reviewer in a normal window call this..."): every published subject below 45 carries a
D-band disqualifier mini-kode lacks - zero tests (codemachine 42.5), fake tests (binharic 40),
runner-blind tests (groq 36.5), dead rule (b) (agentless 42), no loop (codel 23), no safety at all
(darce 32.5, 1). mini-kode ships a working, published, documented agent whose defaults prompt, and
its own provisional reviewer independently landed 46.5 on the same tree. A 0.75 arithmetic gap is
noise at that structure. If synthesis re-derives, note the two decisive ±0.5 lanes: token-economy
and interop.

Rules: (a) n/a - original provenance, no corpus lineage; (b) n/a - not archived, <12mo at review
per remote-verified census. No calibration demotion; straight recount.

## 5. Findings (7 new, ids mini-kode-b1..b7)

b1 workspace-path-containment (sibling-prefix, contradicts own doc, high) / b2
project-settings-self-grant (model writes permissions.json via auto-approved dir, global-bash in
one edit, high) / b3 hook-trust-scoping (repo-shipped mcp.json spawned on open + permissions.json
accepted ungated, clone-and-run x4, high) / b4 permission-policy (compound-grant-ride via
last-segment extraction + first-token blacklist, med) / b5 test-suite-without-ci (32 real files,
276 cases, zero CI; runner-blind check PASSED - positive nuance vs groq) / b6
compaction-trigger-fixed-window (hardcoded 115k vs small-window default provider, sibling of coro
completion-budget miswire, med) / b7 orphaned-persistence (sessions saved, load unwired, low).

**Independence disclosure:** while grepping `findings/` for concept dedup (protocol-mandated), a
`network-egress-approval` grep incidentally surfaced the original review's `mini-kode-3` title
(fetch auto-approved egress) before my fence was complete. My planned laundering finding is
therefore DROPPED as a duplicate - the laundering question is answered only in the report's
verdict (section 2), no new record. No other original-review content read; subjects/scores files
never opened.
