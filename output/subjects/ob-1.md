# ob-1 (@overbrilliant/ob1) -- T1 review

Tier: T1 (census non_test 41,282; src *.ts(x) = 22,366 + scripts = 8,339, rest docs-site/demos).
Manifest: package.json `@overbrilliant/ob1` v0.3.13, Apache-2.0, bun/TypeScript, single loop at
`src/agent/loop.ts` (1,066 LOC). Shallow clone (`.git/shallow`), HEAD 2026-09-25 -> activity claims
evidence-limited per protocol; nothing suggests archived.

## Anchor question (answered before scoring)

Closer to **crush** than any other anchor: both are mid-size agents whose verification is
property-asserting and CI-gated down to a live on-host sandbox-enforcement test, both ship a
loop-breaker and worktree-isolated parallelism, and both sit below the cline rung because enforcement
underneath approval is not the default and there is no SDK/IDE surface. OB-1 exceeds crush on
originality and prompt-cache discipline, undercuts it on corpus size, multi-OS CI, and institutional
durability.

## Loop (`src/agent/loop.ts`)

Single `runTurn` (607-1066) ReAct loop with injected seams (`_callModel`, `_runWorker`, `verify`,
`approve`) -- genuinely host-agnostic (REPL and Ink TUI both consume it; `interactive` flag exists to
avoid mislabeling non-TTY auto-denials, loop.ts:41-45). Termination on *absence of tool_use* rather
than stop_reason, with the 400-cascade rationale spelled out (loop.ts:727-731). Gates per tool call,
in order: loop-breaker identical-call (255-258, 856-877) -> Plan-mode gate (880-887) -> test-edit
guard (889-907) -> policy engine deny/allow/warn (911-927) -> capability-token consumption (929-934)
-> interactive approval with `forceAsk` override even in autopilot, e.g. public tunnels (940-947) ->
PreToolUse hooks (977-988). Post-turn auto-verify with self-fix budget (autofixMax default 3) and a
one-time "no checks ran -> NOT verified" nudge that refuses to equate absence-of-checks with green
(753-812, 826-843). Degenerate-turn guards for tool calls emitted as text/JSON by weak models
(`looksLikeUnsentToolCall`, 524-545) with a single nudge-retry. Step-retry on mid-stream provider
fallback re-issues the step without burning budget (703-712). Runaway backstop at 1,000 steps with a
resume-keeping stop message (245-254, 1053-1060).

## Compaction / context (`src/agent/context.ts`)

Two tiers on one model-scaled budget: `budgetChars = (contextWindow - output_reserve) x 4 chars`
against the *resolved* model (context.ts:49-54; loop.ts:614-618) -- a 1M-window model is not compacted
at 128k thresholds. Tier 1 at 60%: deterministic eviction of stale tool-result bodies, path-aware so
the freshest read of each file is protected (96-135); eviction invalidates the read-dedup cache so a
dedup pointer can never reference evicted content (loop.ts:616-619). Tier 2 at 85%: LLM-summary
compaction whose kept-window cut walks past orphaned tool_results that would 400 the next call
(context.ts:172-178). Manual `/compact [focus]` shares the summarizer prompt (62-68). No
branch-summarization, no cache warming. Prompt-cache discipline is strong: system split into cached
stable block (AGENTS.md, skills, repo map) and uncached volatile tail (semantic memory, date, model
identity) (loop.ts:270-312); retry-only hints are appended as extra uncached blocks so the warm prefix
survives (loop.ts:688-695); a CI test asserts the cached prefix stays task-agnostic
(scripts/parity-harness-smoke.ts:41-49). Cost visibility: per-turn token line incl. cache-read, `/usage`
analytics (usage-smoke).

## Safety enforcement

Approvals bind in the loop and are tested (parity-harness denial scenarios; escalation-smoke). Policy
engine + folder-trust resolver are pure and smoke-tested (safety/policy.ts:62-70, 122-137;
scripts/policy-smoke.ts); empty conditions match nothing by design (policy.ts:43-45). Bash intent
classifier walks shell segments and embedded code (bash-validation.ts:344-517) -- regex-family, not
AST. The OS sandbox is real: Seatbelt profile with quote/newline injection guards on embedded paths
(sandbox.ts:57-79) and bwrap with tiered hardening (`--cap-drop ALL`, `--new-session` vs TIOCSTI,
namespace unshares, capability probed with the EXACT runtime argv, hardened->base->none fallback,
sandbox.ts:26-49, 92-114). Degrade-to-unsandboxed is loud, never silent (sandboxNote, 127-140), and
CI runs a live enforcement proof on real Linux -- network denied, writes confined per mode, with a
baseline check that makes the denials meaningful (.github/workflows/ci.yml:36-73, scripts/bwrap-enforce.ts:25-55).

**But the defaults undercut it**: `permissionMode` defaults to **autopilot** (config.ts:706) and
`sandbox` defaults to **off** (config.ts:708). The mitigating trust gate (index.ts:2433-2455) is
on-by-default and downgrades only an *implicit* autopilot to ask in untrusted folders; one `/trust`
(or any saved setting/env) yields no prompts AND no sandbox. Tested machinery + wrong default = the
6 rung, same shape as the nanocoder annotation ("below the 6 rung, not equal" applies even harder when
the jail is default-off); ob-1 earns 6 over nanocoder's 5 on the trust gate, policy engine, loud
degradation, and CI enforcement proof.

## Verification

Census `test_loc: 0` is wrong -- tests are colocated/in-scripts: 6 `*.test.ts` in src (925 LOC,
incl. exhaustive truth-table on `shouldEscalate`, loop.test.ts:29-33), **71 deterministic smokes**
gated on every push/PR (scripts/ci-smokes.ts:19-88), live bwrap enforcement + MCP interop vs the
official reference server (ci.yml live job), and real-PTY TUI e2e using a pyte terminal emulator
(ci.yml:44-49, tui-pty.py 297 LOC). The MockBrain parity harness drives the REAL runTurn + buildTools
through scripted scenarios asserting wire behavior (parity-harness-smoke.ts:1-55) -- faux-provider
testing done right. Tests assert properties ("denial returns this reason", "escalate iff on && !plan
&& !aborted"), not existence. Ceiling: evals exist (eval/tasks/*.json, eval.ts, docs/evals.md) but
eval.yml is workflow_dispatch + secrets-only, and there is no fuzzing -> verification stops at 7 per
the errata (below the 8 rung, which needs a corpus at anchor scale; ~9.5k test LOC, single-OS).

## Orchestration

`spawn_subagents` (parallel read-only, capped, durable review reports), `spawn_write_subagents`
(disjoint `files` lanes, git-worktree isolation, overlap aborts the batch untouched, merge through the
approval gate -- loop.ts:155-235 + subagents-write-smoke with real git), Fusion best-of-N with
worktree test-scoring and verify-revert, deep mode (Thompson widen-vs-deepen), refute-reviewer, and
the deterministic verified-escalation signal wiring Solo failure into Fusion once per user turn
(loop.ts:120-128, 796-804). No durable queue/daemon, no crash journal; /rewind shadow-git checkpoints
are ON by default (config.ts:709) and revert bash-made changes without touching the user's .git
(checkpoint.ts:1-21). 7, crush-parity.

## Interop

MCP client stdio + http/sse with bearer/custom-header auth and deferred tool loading, interop-tested
live against the official `@modelcontextprotocol/server-everything` in CI (ci.yml:66-68,
mcp-auth-smoke). LSP diagnostics client (context/lsp.ts + lsp-smoke). No MCP server, no ACP, no
published SDK, no structured headless JSON contract (piped stdin falls into the REPL). 6.

## Operability

/resume conversation store per workspace (index.ts:254-277), /rewind checkpoints code+conversation
(1275-1291), settings schema validation dropping invalid fields (config.ts:383), friendly error
translation with action links + recovery recipes (recovery.ts, error-format-smoke), first-run
onboarding, sandbox state surfaced at startup with degrade warnings (index.ts:2508-2509). No session
fork/branch tree. 7.

## Originality (strongest)

Embedded keyless free-model router: ~20 free-tier providers in-process, strategy-ordered candidate
selection, 8-hop failover, escalating cooldown ladders, rate limits self-learned from 429s, bandit
reliability/latency pseudo-counts (providers/free/index.ts:1-35, state.ts:3-6,66) plus a tool-call
repair post-processor for weak free models (tool-repair.ts:280) -- the whole "free tier as a first-class
provider plane" design is corpus-unique and smoke-tested (free-router-smoke, 319 LOC). Verified-failure
escalation as a pure signal (no LLM router), the test-edit guard, the unsent-tool-call recovery,
memory reflection trees with depth caps (memory/store.ts:19-43), skill self-learning with curator
aging (skill-learn/-distill/-curator smokes). Docked from 9 because the claw-code parity lineage (below)
means several features are adopted, not invented. 8.

## Provenance / lineage

Census `provenance_flag: original` holds for CODE (own TypeScript, Apache-2.0, no leaked-source
strings). But 11 modules carry a first-line "parity with claw-code's X" header
(approval-tokens.ts:1, claims.ts:1, green-contract.ts:1, hooks.ts:1, recovery.ts:1,
config-validate.ts:1, git-state.ts:1, lsp.ts:1, eval/parity.ts:1, bash-validation.ts:1,
policy.ts:1) -- the feature list was deliberately modeled on claw-code, which the corpus has already
flagged as a Claude Code derivative (claw-code-1). Concept genealogy for synthesis; not a sync-fork,
so calibration rule (a) does not bind. Identity hygiene: no facts imported from claw-code/-agent.

## Durability

1 contributor, PR count >= 34 at HEAD, npm package + Homebrew tap, signed release workflow with
attestations (release.yml:13-16), fresh-install matrix CI, SECURITY.md with private vuln reporting.
Shallow clone -> growth-rate evidence-limited; no institution. 4.

## Docs/DX

31 structured in-repo docs (ARCHITECTURE, core-concepts, free-models, memory, multimind, evals,
troubleshooting) + separate Astro docs-site, GIF-led README, scripted install, onboarding flow. Not a
generated-schema/docs drift gate as in nanocoder; README's free-tier competitive framing is marketing,
not scored. 7.

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 7 | loop.ts:607-1066 injectable single loop, clean module split; index.ts:1-2521 REPL/dispatch hub |
| verification | 7 | ci-smokes.ts:19-88 (71 CI-gated property smokes), ci.yml:36-73 live bwrap+MCP, bwrap-enforce.ts:25-55; no evals-in-CI, no fuzzing |
| safety-enforcement | 6 | approval+policy tested (parity-harness); sandbox.ts:92-114 real hardening but default off (config.ts:708), autopilot default (706) w/ trust gate (index.ts:2439-2455) |
| token-economy | 7 | context.ts:42-54 model-scaled two tiers; loop.ts:278-312 cache-split; dedup-invalidation coupling loop.ts:616-619 |
| orchestration | 7 | subagents-write disjoint worktrees (loop.ts:196-235), verified escalation (120-128,798-801); no queue/journal |
| interop | 6 | MCP stdio/http/sse+auth, live reference-server interop (ci.yml:66-68); no server/ACP/SDK/headless-JSON |
| operability | 7 | /rewind shadow-git default-on (config.ts:709, checkpoint.ts:1-21), /resume (index.ts:254-277), error explain+recipes |
| originality | 8 | free-model bandit router + tool-repair (providers/free/), verified-escalation, test-edit guard, reflection trees |
| durability | 4 | 1 contributor, shallow; npm+brew+SECURITY.md, no institution |
| docs-dx | 7 | docs/ 31 files + docs-site, onboarding, install script |

**Weighted total: 67.5 -> band B.** Strongest: originality. Weakest: durability.

## Census corrections

- `test_loc: 0` -> actual ~9.5k (6 colocated `*.test.ts` = 925 LOC; scripts ~8.3k mostly smokes
  incl. 71 CI-gated; tui-pty.py + stream-pty.py 661 LOC). Same glob-miss family as crush/cline.
- non_test 41,282 vs 31.6k total `*.ts(x)` in src+scripts+eval+docs-site+demos -- the census likely
  counts .md/.mjs/other; agent code is ~30k either way (src 22.4k). T1 stands regardless.
- suggested_tier T1 accepted (src+scripts ≈ 31k).
