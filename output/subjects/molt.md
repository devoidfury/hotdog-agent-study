# molt -- T1 review

**What it is (honesty check first):** a coding-agent harness, not an app-builder. Terminal
coding agent (`molt run`, tools read_file/write_file/edit_file/bash/grep/list_dir at
`src/engine.ts:2172-2320`) with an Electron desktop shell riding the same engine
(`electron/`, `src/app.tsx`, `test-e2e/drive.mjs:1-8` drives "a real turn, through the real
window"). Manifest `molt-desktop` names the repo; the CLI ships separately as `@solvyx/molt`
(`.github/workflows/release.yml:1-3`). The full rubric applies; no carve-outs needed.

**Census sanity:** shallow clone (`.git/shallow` present, 1 commit visible) -- HEAD date
2026-09-23 is fresh and remote-verified activity was not needed for any claim here.
`test_loc: 21087` undercounts: `test/` alone is 27,483 LOC across 67 files plus
`test-e2e/drive.mjs` (~27.5k+ real test LOC, ratio to source ~1:1). Non-test 34,352 checks
out (src 27,435 + ui 3,061 + electron 2,657 + scripts). Tier T1 stands either way.

**Fork/provenance:** census says original; nothing in-tree suggests otherwise (own protocol
docs, own receipts format, single author). Apache-2.0 in file and manifest, consistent.
Identity check: no collision with any of the listed confusion pairs.

## Anchor question

Closest anchor: **crush** -- same shape of one big file fusing loop, policy and wiring
(`engine.ts` 5,452 vs crush's agent.go+coordinator) and a "tested approval, nothing
underneath" security posture -- except molt's verification corpus and token economy reach
toward pi rather than crush.

## What the thing actually is

Molt's thesis is that completion is a claim requiring proof: `.molt/done.yml` acceptance
checks, sealed before the work (`Object.freeze` + seal journalled before the first request,
`src/engine.ts:3615-3637`), run against disk when the model says done, refusal receipts
written either way, and a hash-chained tamper-evident journal of everything in between
(`src/journal.ts:1-16`). Compaction is mechanical "shedding" (no LLM call, full history
archived to `.molt/exuviae/`), explicitly positioned against summary-compaction in
`docs/shed.md:26-38`: verification needs the original, not a summary of it.

## Dimension scores

### architecture 6 (crush-adjacent, below it)
- Loop is `Engine.runTurn` (`src/engine.ts:3597`), a ~1,400-LOC generator: sealed criteria
  preflight, proof-attempt loop, ceiling accounting, elide/shed, step loop, salvage.
- `engine.ts` = 5,452 LOC fusing loop, budgets, shedding, tool dispatch, bar wiring, and all
  three backend paths (HTTP, Claude Agent SDK, ACP). Per the errata, >5k LOC docking factor
  applies doubly: this is not a host/TUI file, it is the core itself. crush's 2393+1877 split
  reads better than one 5.4k class (errata: 8->9 gap is separation; 6->7 gap is god-file
  severity, and molt's is the worst god file on the anchor ladder).
- Credit where due: backends are a real seam -- subprocess steps produce "the same three
  things the HTTP path produces" (`engine.ts:4015-4020`); `autonomy.ts`, `journal.ts`,
  `receipts.ts`, `bar.ts`, `redact.ts`, `watchdog.ts` are genuinely single-purpose; product
  surfaces (TUI `cli.tsx`, desktop `app.tsx`) consume the engine event stream, not its guts.

### verification 8 (anchor ceiling, honestly reached)
- 67 test files + e2e, ~27.5k LOC, named like incident reports, not features:
  `vacuous-pass.test.ts`, `stats-honesty.test.ts` (header `:1-7` enumerates four real
  misreported numbers), `turn-loss.test.ts`, `loop-close.test.ts`, `wire-validity.test.ts`,
  `journal-order.test.ts`, `partial-evidence.test.ts`.
- Faux provider: `test/helpers.ts:21` `scriptedProvider`; e2e stubs only the model --
  "engine, bar, receipts, IPC and DOM are all the shipping ones" (`test-e2e/drive.mjs:1-8`).
- Meta-verification no anchor has: `src/tests-real.ts:1-24` ships a builtin bar check that
  refuses completions whose new tests are tautologies, assertion-free, or import-disconnected
  -- with a stated conservatism rule (false positive "refuses correct work").
- CI runs typecheck+test+CLI smoke (`check.yml:14-30`). Ceiling holds: no in-CI model evals,
  no fuzzing, CI is macOS-only, and the window e2e/self-check explicitly do NOT run in CI
  (`check.yml:19-20` comment). 8, not 9, per errata.

### safety-enforcement 6 (the "tested approval, nothing underneath" rung)
- Mechanical deny-by-default gate: "No model judges what is safe... pure function of the
  level, the tool call, and the project directory" (`autonomy.ts:11-16`); new tools, odd
  flags, unknown shell all prompt (`autonomy.ts:16-18`).
- Tested at the right granularity: `test/autonomy-probe.test.ts:129-184` -- destructive
  commands at every level, "a chain is only as safe as its worst link", "nothing
  unrecognised is ever assumed safe". The `git stash`/`config`/`tag` postmortem
  (`autonomy.ts:71-76`) documents a real medium-level hole found and closed.
- Boundary containment only where it binds: `mustBeInside` guards walkers but deliberately
  not gated paths (`engine.ts:2340-2353`); `scrubbedEnv()` + timeout + cancel-kill on bash
  (`engine.ts:2300-2315`).
- No OS sandbox, and the honesty is exemplary: rule 3 of the module is literally titled
  "**Not a sandbox**" (`autonomy.ts:20-23`). Same rung as pi/cline/crush; the honesty and
  probe tests do not buy a 7 without anything underneath.

### token-economy 8 (pi's rung, different mechanic)
- Two-tier mechanical context management: elide superseded tool results first ("a smaller
  move than shedding", `engine.ts:3952-3958`), auto-shed above threshold (`engine.ts:3991`).
  Zero LLM calls in either -- compaction that cannot hallucinate.
- Cache discipline is measured, not hoped: elision protects the prefix once the endpoint has
  shown it caches, because mid-conversation rewrites measured 0% hit on the next step
  (`engine.ts:3960-3967`); sticky `cachingUnsupported` / `streamUsageUnsupported` flags stop
  re-offering refused fields (`engine.ts:1310-1318`).
- Learned `tokenScale`: real-tokens-per-estimated-token ratio per endpoint, monotonically
  rising because "the failure it guards against is carrying too much"
  (`engine.ts:1232-1241`, `tokenScale()` `:315`); tool-result budgets sized to the endpoint's
  context window (`resultBudgetBytes` `:429`, comment at `:1289-1296` explains why shedding
  cannot fix an oversized kept message).
- Cost visibility: per-attempt receipts, billed-vs-estimated step counters (`engine.ts:1300-
  1306`), turn ceiling with interactive-only raise channel so `--yes` headless CI can never
  spend past a budget (`engine.ts:3906-3945`).
- Below 9: no cache warming, no branch/session-tree summarization, shed is drop-older-only
  (keep-2-exchanges, `engine.ts:349-373`).

### orchestration 4
- Loop detection inside the turn: repeat-call `answered` map by (call, result-sha) plus
  line-range coverage `shown` map after a real session "walked past the repeat guard for
  thirty-two steps" (`engine.ts:3722-3756`), dry-streak nudges, identical-bar-failure
  breaker (journal kind at `journal.ts:48-53`).
- Budgets are first-class: per-turn token/USD ceilings with progressive warnings and
  double-or-stop semantics (`engine.ts:3860-3945`), `--for` turn timer, watchdog on silent
  streams (`watchdog.ts:1-12`).
- But no subagents, no queue, no workflows, and -- the big one -- no session resume anywhere:
  grep for resume/continue in the CLI surface turns up nothing but stdin plumbing
  (`src/keys.ts:137`). A crashed process loses the working session; the journal records it
  but cannot restore it. crush earns 7 with sqlite resume; molt has no persistence story for
  continuation. 4.

### interop 5
- Headless contract is real: `molt run`/`molt prove` with graded exit codes (0 verified / 1
  not met / 3 no verdict, `cli.tsx:85`) and `--json` on run/prove/stats/receipts
  (`cli.tsx:153`).
- Unusual and well-engineered direction of interop: consumes *rival agents as model
  backends* -- ACP to `grok agent stdio` / `gemini --experimental-acp` (`acp.ts:1-40`) and
  the Claude Agent SDK -- while exposing molt's own tools to them as an MCP server
  (in-process HTTP + stdio bridge, `mcp-bridge.ts:1-15`), denying every non-molt tool at
  `session/request_permission` (`acp.ts:36-40`).
- Missing: MCP client (can't consume third-party tools; `mcpServers: []` for its own
  backends, `acp.ts:337`), no ACP server for IDEs, no published SDK. Desktop + TUI are the
  only surfaces. Above codel's 2, below nanocoder's 7 (which ships a full ACP server).

### operability 5
- Audit operability is best-in-corpus: hash-chained JSONL journal + `molt verify` recomputing
  the chain with honest "tamper EVIDENCE, not prevention" (`journal.ts:1-16`), `molt log
  [--raw|--json]`, receipts incl. refusals, exuviae archive, `doctor` (`check.yml:29`),
  secret redaction closing the `curl -H "authorization: Bearer..."` journaling hole
  (`redact.ts:1-12`).
- Turn cancellation rolls the transcript back to `turnStart` "so a cancellation can leave no
  trace" (`engine.ts:3604`); commands kill their in-flight processes (`engine.ts:2310-2313`).
- But no resume, no fork, no checkpoint-revert of the *working tree*, no rewind of
  conversations. Anchors at 6 (nanocoder) at least have a session manager; molt's
  operability is forensic, not continuative. 5.

### originality 8
- Verified completion as the load-bearing feature, with the parts to make it honest: criteria
  sealed and journalled before the first request (`engine.ts:3615-3637`), "recorded, not
  machine-checked" notes that may never be described as passing (`engine.ts:3693-3700`),
  `tree-accounted` refusing claims when the worktree holds a change no ledger entry explains
  (`acp.ts:28-31`), substance-changed and removed-assertion detection in the write ledger
  (`engine.ts:2216-2226, 2294-2301`), and the anti-vacuous-test builtin (`tests-real.ts`).
- Subscription-honest rival transport: refuses to lift OAuth tokens out of `~/.grok/auth.json`
  and instead speaks the vendor's documented protocol to their CLI (`acp.ts:5-15`) -- an
  ethical line no other corpus subject draws.
- Demo is self-referential and real per README's unedited receipt. Not 9: none of this is
  field-proven beyond one author, and pieces have partial precedents (validated-finish-gate
  x4, Claude Code's microcompact acknowledged in `docs/shed.md:9-13`).

### durability 3
- One contributor, bus factor 1, no SECURITY.md, org-level sponsors (GitHub Sponsors/Polar,
  `FUNDING.yml`). Shallow clone blocks activity claims in either direction (census head
  2026-09-23 is 6 days old at review, so freshness at HEAD is fine). CHANGELOG, versioning
  doc, and dated audit (`docs/audit-2026-09-02.md`) show care, not institution. Below
  nanocoder's 4: no collective, no visible outside contributors.

### docs-dx 7
- 15 in-repo docs, 2,741 lines, that match the code (every claim cited in this report came
  from docs and was then verified in source -- the pi property, at smaller scale):
  `docs/shed.md`, `done-yml.md` (597 lines), `transparency.md`, `autonomy.md`,
  `testing-charter.md` (a testing brief "grounded in defects that were actually found"),
  `prior-art.md` (knows what Claude Code already does).
- README opens with an unedited real receipt. Docked: no SECURITY.md, install path thin vs
  cline's doc tree, no config schema reference.

## Totals

| dim | score | weight | pts |
|---|---|---|---|
| architecture | 6 | 15 | 90 |
| verification | 8 | 15 | 120 |
| safety-enforcement | 6 | 10 | 60 |
| token-economy | 8 | 10 | 80 |
| orchestration | 4 | 10 | 40 |
| interop | 5 | 10 | 50 |
| operability | 5 | 10 | 50 |
| originality | 8 | 10 | 80 |
| durability | 3 | 5 | 15 |
| docs-dx | 7 | 5 | 35 |

**Weighted total 62.0 -- band C.** Strongest: verification (tied with token-economy and
originality at 8, but verification has the CI evidence to hold it). Weakest: durability (3).

C 62 sits between nanocoder 60.5 and crush 69: it out-tests crush, under-orchestrates it,
and shares crush's exact safety rung. Not a band-boundary risk (3.0 from 65; if synthesis
moves orchestration or interop up one, it crosses -- flagging for synthesis's eye, not a
demotion).

Calibration notes: no fork relationship; no calibration rule triggered. Scored against
anchors, ceiling rules from the errata applied (verification capped at 8 absent in-CI
evals/fuzzing; god-file dock applied to architecture).
