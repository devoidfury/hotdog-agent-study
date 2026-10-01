# codebuff (CodebuffAI/codebuff, manifest `codebuff-project`) — T2

**Anchor question, in one sentence:** closer to **cline** than to any other anchor -- an
SDK-first separated core (`packages/agent-runtime` + published `@codebuff/sdk`) consumed by
thin hosts, with a genuinely property-style test corpus around the loop -- but below cline on
enforcement (nothing binds on the general agent path) and above it on token-economy
(cache-gated compaction tiers that rival codex's).

## Identity / provenance

- Census row verified in place: manifest `package.json:2` `"name": "codebuff-project"`; repo
  origin `github.com/CodebuffAI/codebuff`. **No relation to the `forge` row
  (`forge-code-evals`, antinomyhq/forge)** beyond the vague word "code"; nothing in this tree
  references forge. Worked only from `/data/samples/agents/codebuff`.
- Shallow clone (`.git/shallow`), HEAD `dfa45d2` "Sync public snapshot from freebuff-private".
  The public repo is an **export mirror of a private tree**: CONTRIBUTING/pr-hygiene comment
  says "Reviewers here port accepted changes by hand into a private source tree"
  (`.github/workflows/pr-hygiene.yml:102-104`), and internal comments reference private paths
  (`freebuff/web/...`, `docs/freebuff-sponsored-local-execution.md` cited at
  `sdk/src/tools/sponsored-sandbox.ts:31-33` but not shipped). Do not read contributors=1 as
  bus factor -- history is one squashed snapshot commit; the hygiene workflow itself replays
  "the last 300 PRs" (`pr-hygiene.yml:63`), so a real contributor community surrounds the
  mirror.
- License Apache-2.0 (file + manifest). Commercial backing: five hosted products, support
  email, BYOK path (`sdk/src/byok.ts`).

## Census sanity

- `test_loc: 122213` undercounts: `find` over `*.test.ts|*.test.tsx|*.spec.ts` = **151,832
  LOC**; non-test `*.ts|*.tsx` = 186,686 (census 184,205, close). Total TS ~339.5k. Tier T2
  stands regardless; record the +30k test correction.
- `head_date 2026-09-27` fresh; no archived/dead question.
- Note: `common/src/constants/freebuff-models.ts` is a 5,207-LOC generated data blob -- counts
  as non-test LOC but is not logic; factor out of "god file" judgments.

## Core loop (mandatory read: `packages/agent-runtime/src/run-agent-step.ts`, 1,632 LOC)

- `runAgentStep` (:204-747) builds one step: drops orphaned tool calls that would make
  strict providers reject every later turn, and writes the cleaned history back so the
  checkpoint is valid too (:322-331); assistant-prefill guard appends "Continue from where
  you left off" for models that reject trailing assistant messages (:352-365); per-step
  context estimate = latest provider receipt + cheap deltas, never re-tokenized
  (:543-548 + `util/context-token-count.ts:20-40`).
- Turn-ending semantics are explicit and mode-dependent: agents with `task_completed` must
  call it; think-only responses continue the turn instead of ending it (:683-706).
- **Two-stage loop-breaker for unchanged to-do spam** (:630-660): at 3 consecutive identical
  `write_todos` calls inject recovery guidance, at 6 throw `TodoLoopError` and end the turn
  (`util/todo-loop.ts:6-8`); the recovery note is contextualized -- if the agent has no file
  tools (plan mode) the message says so, because a GLM stuck-loop was chasing a tool it was
  never offered (`todo-loop.ts:24-31`). Tested: `loop-agent-steps.test.ts` (46 `it` blocks
  driving the real loop over fixture providers).
- `loopAgentSteps` (:749+) is the outer while-loop: step budget (`stepsRemaining`, warning +
  forced turn-end at :283-313), steering hook drained at step boundaries so a host can steer
  a running agent mid-turn (:784-790 contract, :1404-1420 drain), compaction decision before
  each step (:1178-1260), run ledger callbacks by dependency injection (web API in prod,
  no-op for the context-pruner to avoid three round trips per step, :1046-1058). Error paths
  keep accounting honest: even failed/cancelled turns recount the context the user will see
  (:1497, :1587).
- Weakness: `runAgentStep`/`loopAgentSteps` fuse loop, compaction orchestration, telemetry,
  cache-debug snapshots and output-schema retry into one file with a 10-way `ParamsExcluding`
  DI soup (:204-260) -- testable, but the parameter contract is enormous. Not a cline-style
  dual-tree problem though: single tree, clean package direction (cli -> sdk ->
  agent-runtime -> common, per imports in `sdk/src/run.ts:3-58`).

## Compaction (mandatory read: `compact-history.ts` 1,248 + `model-compaction.ts` 384)

This is the strongest module in the subject.

- **Mechanical no-LLM pass** (`compactMessages`, `compact-history.ts:701-841`): rewrites
  history into a `<conversation_summary>` with per-role budgets (50k user / 20k
  assistant+tool, :56-62), tool calls condensed to one-line records that keep the concrete
  facts a coding agent needs to resume (:150-330), prior summaries re-parsed and re-budgeted
  so second compactions don't double-count (:437-460, :747-756).
- **Cache-expiry-gated opportunistic trigger** (`promptCacheGapMs` :1035-1060,
  `evaluateCompactionTrigger` :1066-1113): compacts on ordinary idle turns *only once the
  provider prompt cache has demonstrably gone cold* (last-assistant -> user-prompt gap
  exceeds TTL), because then "the next request re-reads the whole history at full price
  anyway, so rewriting it costs nothing" (:1056-1064) -- with a min-tokens floor so small
  contexts aren't compacted for nothing, which the context-limit trigger deliberately
  ignores (:1101-1103). This is the same family as gptme's TTL cold-cache prediction
  (convergent), and the reasoning is codex-tier.
- **Model-handoff compaction** (`compactWithModel`, `model-compaction.ts:103-287`): map-reduce
  chunking with binary-search fit against the real request budget (:200-225), a JSON replacer
  that names base64 payloads instead of shipping screenshots into the summarizer (the fix for
  a real 800k-token incident, documented with hourly request stats, :71-98, :139-147),
  tolerant argument-unwrap for double-encoded tool JSON (:38-68).
- **Fallback ladder** (`compactWithModelOrFallback` :316-384): model pass fails -> mechanical
  pass -> history left untouched with an over-budget guard; only cancellation propagates.
  Comment at :1145-1148 notes the previous design (LLM summarizer returning full history on
  failure) left context over budget -- the fix is recorded, not just present.
- Manual `/compact` (`compactHistoryNow` :1200-1247) honestly no-ops when the rewrite
  wouldn't shrink the history instead of pointlessly breaking the provider cache.
- The serialized `agents/context-pruner.ts` carries a deliberate duplicate of the algorithm
  (its `handleSteps` is `toString()`-eval'd standalone and bundled into 3 artifacts), kept
  honest by a byte-identical **parity test** (`__tests__/context-pruner-parity.test.ts:1-15`).
- `compact-request-budget.test.ts`, `compact-history.test.ts` (2,699-LOC
  `agents/__tests__/context-pruner.test.ts`), `model-compaction.test.ts` all assert behavior.

## Permissions / sandbox (mandatory read)

- **The general agent path is unprotected by design**: no approval gate anywhere in
  `packages/agent-runtime` or `sdk/src/run.ts` (grep for approval/needsApproval/autoApprove =
  zero hits outside ad/approval-unrelated UI); `resolveFilePath` explicitly honors absolute
  paths to "any file on the system" (`sdk/src/tools/path-utils.ts:43-68`); terminal commands
  run under plain bash (`sdk/src/tools/run-terminal-command.ts:384-399`) with careful process
  group/kill semantics (:141-186, :239+) but no confinement. No checkpoint/revert on the
  normal path -- the only undo is `/ads:undo` for sponsored runs
  (`cli/src/commands/command-registry.ts:282`). SECURITY.md is a 7-line contact boilerplate;
  unlike pi, there is no public statement of "we intentionally have no sandbox."
- **What IS enforced is excellent, but out-of-band**:
  - `.agents`-dir trust gate: repository dirs whose loading would `import()` attacker code or
    spawn MCP stdio servers with this process's env are inventoried (agent files + MCP launch
    commands) and shown to the user once before trust; trust store 0600, realpath'd keys,
    non-interactive runs skip by default (`cli/src/utils/agent-dir-trust.ts:13-35,
    :46-56, :66-79`).
  - Remote-template publisher gate: registry templates carry `handleSteps` as a source string
    that `run-programmatic-step.ts` evals with no isolation; before this gate
    "`--agent anyone/anything` was remote code execution by design" -- now only trusted
    publishers' executable templates load, with the residual gaps (no signing, no pinning,
    unsandboxed eval) explicitly listed as out of scope (`sdk/src/agent-publisher-trust.ts:1-35`).
  - **Sponsored local execution**: advertiser-authored procedures run on the user's own
    machine under real kernel containment -- macOS seatbelt profile + Linux bubblewrap
    (`sdk/src/tools/sponsored-sandbox.ts:1-55`; single bwrap arglist shared with evals),
    `HOME` redirected and dotfiles denied, loopback denied because the orchestrator listens
    there, symlink-aware write containment (:325-530), env allowlist scrub plus value-matching
    secret sweep (`run-terminal-command.ts:341-349`). **"Refuse, never downgrade"**: a missing
    bwrap throws; there is no uncontained fallback (:37-41), and the gap inventory lives in a
    private doc on purpose (:31-33). Windows gets an honestly separate "floor" arm -- scrubbed
    env, redirected profiles, non-interactive git -- that is its own type, so nothing can
    mistake it for a sandbox (:43-49, `common/src/ads/sponsored-local-execution.ts:20-45`);
    the capability grant is *computed from* available containment, never a bare constant.
  - SSRF guard on `read_url`: DNS-resolved IPs classified via ipaddr, everything non-unicast
    blocked, IPv4-mapped v6 normalized, unparseable = blocked (`sdk/src/tools/ssrf.ts:1-51`).
- `spawn-agents-permissions.test.ts` (37 tests) pins which tools subagents inherit.

## Verification

- 151,832 LOC of tests, and they prove things: loop semantics, cancellation
  (`run-cancellation.test.ts` 1,465), compaction budgets, sandbox profile generation
  (`sponsored-sandbox.test.ts` 1,830; `sponsored-containment-probe`,
  `sponsored-windows-floor`), cross-artifact parity. DI-over-mocking doctrine documented
  (`docs/testing.md:1-6`), fixture-env discipline explains silent-suite-death failure modes
  (`docs/testing.md:10-24`).
- **But public CI runs none of it.** `.github/workflows/ci.yml:33-58` = install, build SDK,
  build binary, smoke the binary (`cli/scripts/smoke-binary.ts`); `package.json ci` script =
  builds only. `docs/testing.md:16` asserts "CI runs `cd <package> && bun test`" -- true of
  the *private* CI, not this repo. `pr-hygiene.yml:1-8` and `ci.yml:3-8` both candidly
  document that the public repo previously had *no* CI at all. Evals (BuffBench: real commit-
  reconstruction tasks, 2-judge panel with loud single-judge warning,
  `evals/buffbench/README.md:1-40`) are likewise never run in CI. No fuzzing anywhere.

## Orchestration / interop / operability

- `spawn_agents` parallel subagents with ancestor-run lineage
  (`spawn-agent-utils.ts:314-315, :416`), agents-as-tools (`buildAgentToolSet`,
  `run-agent-step.ts:1013-1019`), programmatic generator steps, steering. No queues, no
  daemon, no crash-recovery journal -- the run ledger is the company web API
  (`AddAgentStepFn` DI); the context-pruner deliberately bypasses it (`:1046-1058`).
- Interop: MCP **client** only (stdio/SSE/streamableHTTP, `common/src/mcp/client.ts:2-4`),
  no MCP server, no ACP, no public headless JSON CLI mode (print-mode events are internal).
  Published `@codebuff/sdk` built by CI; **imports rival-harness config**: skills load from
  `.claude/skills` alongside `.agents/skills` (`tools/handlers/tool/skill.ts:35-38`).
- Operability: chats persisted as JSON per project with truncation-aware listing -- a
  crash-mangled chat is shown as unreadable rather than silently vanishing
  (`cli/src/utils/chat-history.ts:12-19`), `--continue [id]` (`cli/src/cli-args.ts:57-59`);
  exit-sweep kills orphaned command trees (`run-terminal-command.ts:213-247`); diagnostics
  command exposes active PIDs without leaking command text/env (`:196-210`); fatal crash
  reporting (`cli/src/utils/fatal-crash-report.ts`). No checkpoints, no session tree/fork.

## Originality / docs / durability

- The **ad-funded agent economy** is corpus-unique and it is not marketing -- "sponsored runs"
  are advertiser-authored procedures with a capability grant, refusal policy
  (`common/src/ads/sponsored-command-refusals.ts`, 1,183 LOC), acceptance criteria, and the
  local kernel sandbox above; `/ads:undo` exists. Text ads pay for free model access
  (README, corroborated by `common/src/ads/` and `cli/src/ads/`).
- Other original bits: compaction-while-cache-cold (convergent with gptme, independently
  reasoned); the serialized-agent duplication with parity test as a build-constraint-driven
  pattern; per-model agent variant files under `agents/`.
- Docs are thin publicly: 770 lines total (`docs/`), README is product marketing;
  CONTRIBUTING/testing docs are real and candid. `docs/freebuff-sponsored-local-execution.md`
  -- the security prose for the flagship boundary -- is not shipped.
- Durability: funded company, active mirror (snapshot 2 days before sampling), community PRs
  flow but land privately; single visible committer is a snapshot artifact. Public repo's
  usefulness without the company services is partial (BYOK mitigates).

## Scores (against the frozen ladder)

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | clean sdk->runtime->common layering (`sdk/src/run.ts:3-58`); loop fully DI'd and host-agnostic (`run-agent-step.ts:749+`); docked for 1.6k-LOC loop file fusing steps+compaction+telemetry and the `ParamsExcluding` param soup (`run-agent-step.ts:204-260`) |
| verification | 5 | real property corpus, 151.8k LOC, behavior-asserting (`loop-agent-steps.test.ts`, parity test, `spawn-agents-permissions.test.ts`) -- but public CI runs build+smoke only (`ci.yml:33-58`); `docs/testing.md:16` describes private CI, not this repo |
| safety-enforcement | 5 | superb machinery + honesty on narrow paths (sponsored sandbox refuse-never-downgrade `sponsored-sandbox.ts:37-41`; dual trust gates) + SSRF guard, but general loop: no approval, no sandbox, project escape allowed by design (`path-utils.ts:43-68`), no public honesty statement (contrast pi `SECURITY.md:50`) |
| token-economy | 9 | compaction tiers mechanical/model/fallback (`model-compaction.ts:316-384`) + cache-expiry-gated trigger (`compact-history.ts:1066-1113`) + receipt+delta accounting (`context-token-count.ts:20-40`) + binary elision + parent-prompt-sharing subagent cache test (`prompt-caching-subagents.test.ts`) |
| orchestration | 7 | spawn_agents lineage (`spawn-agent-utils.ts:416`), steering (`run-agent-step.ts:1404`), step budgets; loop-breaker; no queues/crash journals locally |
| interop | 7 | published SDK built by CI, MCP client w/ 3 transports, `.claude/skills` import (`skill.ts:35-38`); no ACP/MCP-server/headless-JSON |
| operability | 6 | `--continue`, truncation-aware chat resume (`chat-history.ts:12-19`), exit-sweep + PID-only diagnostics (`run-terminal-command.ts:196-247`); no checkpoint/revert outside ads |
| originality | 8 | ad-sponsored locally-sandboxed runs end-to-end in code; cache-cold compaction economics; serialized-agent+parity-test pattern |
| durability | 6 | funded product company, active snapshot, contributor flow via mirror; development closed, key security docs private |
| docs-dx | 5 | clear install path, candid CONTRIBUTING/testing docs; 770 LOC public docs; flagship security doc not shipped |

**Weighted total: 67.0 -> band B.** Strongest: token-economy (9). Weakest: verification (5).

Calibration: not a fork of any corpus subject; rule (a) n/a. Boundary note: 67.0 sits 2 points
above the B/C line; verification (5, if the 8-rung "CI must run the corps" is read strictly as
4) or token-economy (9, if read next to codex's 9 as an 8) moves it across -- synthesis should
keep an eye on this one either way.
