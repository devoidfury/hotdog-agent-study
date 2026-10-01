# darce-cli — T1 review (2026-09-29)

Subject: `/data/samples/agents/darce-cli` — TypeScript/Ink CLI agent, solo author
(Amer Sarhan), npm `darce-cli` v0.3.5, hosted-account business model (login/register
against api.darce.dev, $20/mo Pro tier). Census tier T1; see census corrections below --
true size (~2.7k LOC) is T0-range, but the T1 depth review was run as dispatched.

**Anchor question. Which anchor subject is this closer to, and why?** Codel (23.0, D):
both share the absence profile that dominates the rubric -- no enforcement under the
loop, no CI, no interop surface -- but darce sits clearly above codel because the loop
is real, provider-agnostic in shape, and backed by 106 behavioral tests; it is nowhere
near nanocoder, which has sandbox machinery, ACP, and a 161k-LOC spec corps.

## License

`package.json:38` claims `"license": "MIT"`; there is no LICENSE file in the tree and
GitHub reports `license: null` (remote-verified). Treated as manifest-only-license
(precedents: coro-code, g3, zap-coding-agent): findings describe concepts only, all
`effort_for_us` values assume clean-room, no code-copy recommendation.

## Census verification

- `test_loc: 0` is WRONG. Root-level `test.ts` (868 LOC, 106 tests, custom mini-runner,
  "Run with: npx tsx test.ts" test.ts:4) was missed by the census glob. Real test_loc
  is at least 868. Dispatcher's suspicion confirmed.
- `non_test_loc: 5763` is INFLATED. Actual `src/` = 2,690 TS/TSX LOC; 5,763 is
  reproducible only by counting `package-lock.json` (2,944 lines) plus config files.
  True project size ~2.7k LOC non-test → T0 threshold (<5k) per protocol tier table.
  Tier downgrade recorded; no rule (b) involvement.
- `head_date 2026-04-01`, `commits: 1`, `shallow: true`: remote-verified via GitHub API
  (repo created 2026-03-31, `pushed_at 2026-04-01`, `archived: false`, 10 stars, 1 fork,
  not a fork). Shallow clone hides nothing direction-relevant here: the repo genuinely
  is ~6 months old with last push at HEAD. No dead claim made; activity is thin, not
  verified-dead (push was ~6mo ago at review date).
- Provenance: original per census; nothing in the tree suggests otherwise (unique
  branding, own auth flow). No corpus-name collision per appendix hygiene list.

## Loop read (core)

`src/core/query.ts:24-164` -- async-generator agent loop, genuinely separated from the
UI: REPL consumes its StreamEvents (`src/ui/REPL.tsx:127-204`). maxTurns default 50
(query.ts:26,43). Compaction hook per turn (query.ts:47-50). Tool dispatch validates
input with Zod `safeParse` and feeds results back as `tool_result` blocks (query.ts:110-121,
152-161). Two defects:
1. `concurrent`/`sequential` buckets are computed (query.ts:98-106) then thrown away:
   every block runs sequentially in `[...concurrent, ...sequential]` (query.ts:130-131).
   Dead classification; the read-only concurrency affordance is unwired.
2. `retriedToolIds` (query.ts:30) is populated on tool error (query.ts:138-144) but
   gates nothing -- no retry is ever performed and no breaker trips; effect is two
   debug log lines. Phantom loop control.

Provider layer is OpenAI-wire-shaped with a single implementation (`OpenRouterProvider`,
src/providers/openrouter.ts:62-271): SSE streaming, per-index tool-call accumulation
(:209-238), 429 exponential backoff (:92-98). Base URL overridable via
`DARCE_API_BASE` (src/config/config.ts:47-49), so any OpenAI-compatible endpoint can
sub in, but there is no second provider.

## Compaction read

`src/core/conversation.ts:4-31`. Trigger: `estimateMessagesTokens(messages) >
MAX_CONTEXT_TOKENS` where the estimate is naive chars/4 (src/utils/tokens.ts:2-4) and
100000 is hardcoded -- `src/config/models.ts` already stores per-model `contextWindow`
(7 models, incl. two 1M-ctx at :42,:49) and compaction ignores it entirely. The
compaction itself is not a summarization: it keeps first + last 6 messages and replaces
the middle with a count-only placeholder ("Context compacted: N messages removed")
(conversation.ts:13-31). No LLM summarizer, no verbatim pinning, no tool-output
pruning tier. `/compact` slash command is a different, harsher path: slice(-4)
(REPL.tsx:80-84). Token-economy positives: per-model cost tracking with pricing
profiles shown live (costTracker.ts:18-31, StatusBar), 30k/10k stdout/stderr
truncation in BashTool.ts:51-58, ReadTool 2000-line cap (ReadTool.ts:30).

## Permission read

There is nothing to read, and I grepped to prove it: no match for
permission/approv/confirm/sandbox/deny/allow across `src/`, `test.ts`, README.
`BashTool` spawns arbitrary commands through `$SHELL -c` with a full env copy and no
prompt (src/tools/BashTool.ts:27-29). File tools resolve user paths against cwd
without containment, so `..`-prefixed and absolute paths escape freely
(ReadTool.ts:25, WriteTool.ts:23, EditTool.ts:27). WebFetch takes an arbitrary URL plus
model-supplied headers with no protocol/SSRF guard (WebFetchTool.ts:25-31). Auth
password is collected via plain readline (echoed) and the API key is written plaintext
to `~/.darcerc` (cli.tsx:73-78,119). Unlike codel, there is no safety marketing to
contradict the absence -- it is honest by omission, which is why it scores 1 not 2.
The only real gate is correctness, not security: EditTool refuses edits to files not
previously Read, enforced via a `readFiles` Set on ToolContext (EditTool.ts:27-29) --
tested (test.ts:329-334) -- but the set resets to empty on `--resume` (cli.tsx:241),
so it does not survive a session reload.

## Verification read

106 `await test(...)` cases in one 868-LOC root file with a hand-rolled runner
(test.ts:38-52). These are real behavioral assertions, not existence checks: SSE framing
edges incl. incomplete frames and colonless lines (test.ts:543-591), EditTool gate +
replace_all semantics (test.ts:325-386), router rule precedence (test.ts:698-731), cost
accumulation (test.ts:642-680). But: no CI at all (no `.github/`), so nothing runs them;
the loop (query.ts), compaction (conversation.ts), and REPL are untested; WebFetch tests
hit live httpbin.org (test.ts:508-511), making the suite network-fragile. Per the
`test-suite-without-ci` pattern this caps at 4.

## Dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | Clean small-module split, loop/UI separated (query.ts:24 vs REPL.tsx:127-204), largest file 367 LOC; docked for dead concurrency split (query.ts:98-106 vs :130-131) and phantom retry set (:138-144) |
| verification | 4 | 106 behavioral tests (test.ts:329-334, :543-591) but zero CI (no .github), loop+compaction untested, live-network tests (test.ts:508-511) |
| safety-enforcement | 1 | No permission/approve/sandbox match anywhere in src; BashTool.ts:27-29 full-env spawn; read-before-edit gate is correctness not safety (EditTool.ts:27) |
| token-economy | 3 | chars/4 estimate vs hardcoded 100k ignoring models.ts contextWindow (conversation.ts:4 vs models.ts:42); middle-drop placeholder compaction (conversation.ts:20-31); cost visibility is the good part (costTracker.ts:18-31) |
| orchestration | 1 | Nothing beyond maxTurns cap (query.ts:43); no subagents/queue/workflow |
| interop | 2 | OpenAI-compatible via base-url override only (config.ts:47-49); no MCP/ACP/SDK/headless-JSON; `darce "prompt"` still renders the TUI (cli.tsx:58,257) |
| operability | 4 | Save-per-turn JSONL-ish sessions + --resume latest-per-cwd (sessions.ts:12-33, cli.tsx:221-231), model switching, Ctrl+C abort wiring (BashTool.ts:66-68); no session listing, no crash logs, restoreCosts never called (costTracker.ts:75) |
| originality | 4 | Conversation-shape-keyed rule router (tool-rounds>=3, token estimate, image presence; router.ts:36-50) exists in code but default `rules: []` makes the README "Smart routing" claim inert (config.ts:7-11) |
| durability | 2 | Solo author, 10 stars, 1 fork, created 2026-03-31, pushed_at 2026-04-01 remote-verified; product depends on proprietary api.darce.dev account backend |
| docs-dx | 3 | Clear install path and decent in-product --help (cli.tsx:10-31); README is marketing tables + fabricated demo transcript; --version hardcoded '0.2.2' vs package.json 0.3.5 (cli.tsx:5) |

Weighted total: (6*15 + 4*15 + 1*10 + 3*10 + 1*10 + 2*10 + 4*10 + 4*10 + 2*5 + 3*5)/10
= 32.5 → **band D**.

Strongest dimension: **architecture (6)**. Weakest: **safety-enforcement (1)**.

## Notes

- Boundary check: 32.5 is not within 2 pts of any band cut (45/65/78/88).
- README marketing table comparing against Claude Code/Cursor and the sample transcript
  are not counted as evidence anywhere above, per protocol.
- Identity hygiene: no appendix collision applies ("darce" is unique in the corpus);
  no facts imported from similarly named projects.
