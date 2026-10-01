# waveloom — T2 deep review

Go terminal coding agent, DeepSeek-native, "engineered for cache economics."
Module `github.com/Menfre01/waveloom`, Apache-2.0, solo maintainer, shallow clone
(1 commit visible), HEAD 2026-09-14 (fresh vs session date 2026-09-29; no activity
claim needed). Version v0.9.1 per CHANGELOG.en.md:1.

## Census sanity

- `test_loc: 0` WRONG (same artifact class as crush/cline). Actual: 145 colocated
  `*_test.go` = 73,104 LOC.
- `non_test_loc: 109,605` overstated vs measured Go non-test 60,954 (Go total
  incl. tests = 134,058; all md/json/jsonl = 7,436). The 109k figure is not
  reproducible from any obvious glob; treat non-test as ~61k.
- Shallow clone confirmed (`.git/shallow`), commits=1, contributors=1. HEAD date
  is recent so no remote check required for the inactivity rule.

## Anchor question

Closer to **crush**: same shape (Go single-binary TUI agent, bwrap/seatbelt OS
sandbox, rule-engine permissions, no MCP-server), and crush itself sets the 69
rung; but waveloom's module boundaries (loop/guard/sandbox/compaction cleanly
separated vs crush's 2,393-LOC fused `agent.go`) and its cache-economy depth
push it up toward the cline 76.5 side, landing between the two.

## Mandatory line-level reads (T2)

Core loop `pkg/agentloop/loop.go` (1,171), `execute.go` (1,545); compaction
`pkg/compaction/compaction.go` (1,137) + `compactor.go`; permission
`pkg/permission/guard.go` (840) + `bash/security.go` (1,642) wiring;
`pkg/sandbox/{manager,config,linux_bwrap,windows_stub}.go`. All cited below.

## Dimension scores

### architecture — 7

Separation is better than crush and approaches codex-style interface
injection: `Config` takes `permission.Guard`, `SandboxMgr`, `compaction.Compactor`,
`UserResponder` as interfaces/optionals (`loop.go:33-127`); loop emits a typed
event channel (`Run → <-chan StepEvent`, `loop.go:346`) documented with six
numbered invariants incl. message-pairing integrity and cancel priority
(`loop.go:337-345`). Per-step state (`TurnState`) is separated from session-level
loop fields (`loop.go:130-137,207-233`). 28 single-purpose `pkg/` packages.
Docking factors: `cmd/waveloom` is a ~13.5k-LOC single package dominated by
`tui.go` 5,406 LOC (the errata rule: product/TUI-mode file >5k LOC docks),
plus `tui_renderer.go` 2,762. Same rung as crush (7), different docking shape:
crush fuses loop+policy, waveloom fuses the product surface.

### verification — 7

- 73,104 LOC of colocated Go tests, 117 test funcs in `agentloop/loop_test.go`,
  101 in `permission/guard_test.go`, 105 in `skill/skill_test.go`; they assert
  properties, not existence: e.g. `agentloop/sandbox_chain_test.go:48-60`
  regression-guards that per-command sandbox status survives the timeout-ctx
  rebuild (the exact bug class that would silently disable sandboxing).
  Dozens of `TestRegression_*` named after internal review findings (二审/五审).
- CI `.github/workflows/ci.yml`: build+test+lint+cross-compile+Windows matrix;
  notably installs real bubblewrap and re-runs a specific bwrap kill regression,
  **failing on silent SKIP** (`ci.yml:23-38`) — anti-vacuous-green gate, better
  than any anchor showed for this.
- Provider contract tested against mock HTTP transport with real API shapes
  (`pkg/llm/client_test.go:19-54`, `adapter_deepseek_test.go:1912`).
- Gaps keeping it off the 8 rung: no `-race` anywhere (Makefile:29-34, ci.yml,
  scripts — none) in a heavily goroutine-concurrent loop; no fuzzing; the
  advertised eval layers are not in CI and partially reference deleted paths
  (`eval/llmedit/README.md:8-16` cites `pkg/hashline/eval` and `cmd/editbench`,
  neither exists in the tree).

### safety-enforcement — 6

Real, tested enforcement above the 6-rung floor, but the defaults and two trust
holes cap it:
- 8-step Check flow with documented short-circuit order (`guard.go:186-215`):
  deny rules > safety hard blocks > allow rules > session memory > bypass.
  Under headless autoAllow, only DENY survives: ask-rules→allow, safety-ask→allow,
  but deny rules, RiskHigh commands, PathDangerous writes stay fail-closed
  (`guard.go:220-260`) — honest and tested.
- Step 0.5 binary-hijack hard block (LD_PRELOAD/DYLD_*/NODE_OPTIONS) deliberately
  placed before ask-rules and skill whitelists because those short-circuit the
  strip-then-check ("只见 git status、执行时注入" laundering;
  `guard.go:236-255`) — unusually well-reasoned placement.
- Parser-differential defense: `pkg/bash/security.go` runs 23 detectors over a
  real `mvdan.cc/sh/v3` AST incl. CR mid-token, mid-word `#` divergences
  (`security.go:3,813,1000`); parse-differential → ASK, never silently allow
  (`guard.go:429-435`).
- OS sandbox: bwrap (cap-drop ALL, path masking, credential env stripping) and
  darwin seatbelt with capability probe + smoke test
  (`sandbox/linux_bwrap.go:74-185`); violations annotated into tool output, not
  swallowed (`tool/shell.go:656-662`). Headless entries auto-activate the sandbox
  even when `sandbox.enabled=false` (`cmd/waveloom/acp.go:128`,
  `sandbox_setup.go:50-55,92`) — but if bwrap is missing, degrade-to-bare is the
  default (`failIfUnavailable=false`, `sandbox/config.go:104-112`), so the
  no-interaction autoAllow path can end up fully unsandboxed with only a warning;
  default network mode on with zero read-masking produces only a stderr warning
  (`sandbox_setup.go:96-108`).
- Safety holes (findings waveloom-5/-6): project-directory hooks from
  `.claude/settings.json` / `.claude/settings.local.json` /
  `.waveloom/settings.json` are loaded and executed with **no trust prompt**
  (`cmd/waveloom/main.go:770-789`) — cloning a repo and running waveloom in it
  is RCE; skill `allowed-tools` bash whitelist persists "until the next skill
  load" and bypasses the Step-3 RiskHigh hard block for subsequent unrelated
  bash calls (`guard.go:105-127,273-292`).
- Windows: no sandbox, WSL2 advice (`sandbox/windows_stub.go:8-14`) — honest.
- `SECURITY.md` exists with PGP disclosure; checklist is honest about bypass risk.

Net: crush-level "tested approval" plus genuine OS machinery, minus crush on
defaults and hook trust → 6, at the top of that rung.

### token-economy — 9

The corpus's second-strongest cache story after codex, and strongest on
prefix-cache *discipline*:
- Four-tier watermark compaction (45/65/85/98%) triggered on real API usage,
  thresholds recalibrated from 148 real sessions because the old 60/80/95 were
  never reached (`compaction/compaction.go:31-47`); Tier 1 snip + Tier 2 prune
  cost zero API calls; Tier 3 incremental LLM summarization chain.
- Monotonic decision set: once a message is snipped/pruned the decision never
  regresses within a session (`compaction.go:64-115`, O(log N) canApply) — this
  is cache-monotonic compaction: compaction never rewrites the already-cached
  prefix. Persisted/restorable across resume (`compactor.go:145-193`).
- Fork subagent first request is byte-aligned with the parent's cached prefix,
  including the tools array ("实测:同 messages 换 tools → 零命中",
  `execute.go:69-76`, `loop.go:110-118 ToolsOverride`); over-window fork
  requests truncate at whole-turn boundaries and refuse rather than 400
  (CHANGELOG.en.md v0.8.2).
- Injection discipline quantified from data: repeated todo snapshots skipped
  because they measured ~26% of all cache-miss tokens
  (`loop.go:1085-1090 injectTodoStatus`); Append-only, never Update, for
  injected messages (`loop.go:432-436`).
- Cost visibility: cache-hit/miss token accounting per step to TUI
  (`loop.go:706-716 StepStats`), DeepSeek peak/off-peak + cache-tiered pricing
  in CNY/USD (`pkg/pricing/pricing.go:19-30`), image budgets per official
  DeepSeek scaling rule, 1024 tok/image (`compaction.go:330-342`).
- Missing vs codex's 9-rung: no no-LLM token-budget fresh window, no cache
  warming as a feature, cache-awareness is defensive rather than proactive.
  Still 9: other subjects should copy the monotonic-decision + fork-alignment
  pattern wholesale.

### orchestration — 7

Fork/cold subagents with three specialist types; recursive-fork detection via
fork-boilerplate tag scan (`subagent/agent.go:227-230,352-367`); fork budget
guard truncates at turn boundaries; subagent transcripts persisted with cache
hit/miss stats (`subagent/agent.go:90-103`). Background task registry with
turn-internal completion injection so the model stops writing sleep-pollers
(`loop.go:88-95 BackgroundCompletions`, `pkg/task/registry.go`). Loop-detection
backoff ladder 3→5→8 with tool-specialized guidance text (edit→re-read only;
rate-limit→wait, not switch tools; rate-limit never escalates fatal to avoid
killing crawl tasks) (`execute.go:760-830`, `loop.go:152-166`). No queues, no
cron, no durable workflow journal, crash-recovery is session-level only.
Same rung as crush (7): comparable breaker quality, weaker durability plane.

### interop — 7

ACP v1 server with full session lifecycle incl. load/resume/delete
(`pkg/acp/handler.go:28-433`), used by Zed/JetBrains via cc-connect
(`AGENTS.md:17`). MCP client importing Claude's own config surfaces:
`~/.claude.json` top-level and `projects.<cwd>.mcpServers`
(`pkg/mcp/config.go:39-53,180-134`). Skill/command/plugin discovery from
`.claude/skills`, `.claude/commands`, `.claude/plugins` alongside native paths
(`pkg/skill/skill.go:116-148`). Hooks read `.claude/settings.json`
(`main.go:774-781`). Multi-provider (DeepSeek/Kimi/OpenAI) with runtime
`/provider` switch. LSP diagnostics after edit/write (`execute.go:716-721`).
No MCP server, no published SDK, no IDE surface of its own — crush/nanocoder 7
rung.

### operability — 8

JSONL sessions with compaction-aware resume ("若 resume 优先加载 JSONL,会把未压缩
原文重新喂给 LLM,导致上下文翻倍" — handled, `session/session_persist.go:185`);
conversation rewind to any user message paired with file-history backups
(`session/context.go:796 RewindConversationTo`, `filehistory/backup.go:10-40`);
structured `type=event` journal lines (compaction/model_error/tool_timeout)
excluded from replay but aggregated into stats (CHANGELOG v0.8.2); cumulative
cost persists across resume (`session/context.go:955`); startup failure paths
print actionable diagnostics including the self-masked-settings.json sandbox
footgun (`main.go:109-114`); panic guards at loop and per-tool level convert
crashes into TurnDone/fatal tool errors (`loop.go:326-344`,
`execute.go:185-200`). Bilingual UX throughout. Short of 9: no fork/archive
matrix, no debug-prompt-input equivalent, `logging` package is 151 LOC.

### originality — 8

Ideas verified in code, not marketing: cache-monotonic tiered compaction
calibrated on the author's own 148 sessions; fork prefix-cache alignment with
parent-only tools registered as explicit error stubs; ThrottleStore — model-facing
rate-limit feedback encoding earliest-retry-time in tool errors so the model
stops UA-switching and hammering (`execute.go:793-803` + CHANGELOG v0.8.2);
preview-suffix continuation detection with a 16-codepoint Unicode colon-variant
regression history (`loop.go:1041-1114`); proplan model routing (plan-mode→pro,
200k-context guard, never-leak sentinel invariant, `loop.go:895-935`);
`experiments/` dir with attention-order/injection-priority/tool-cache A/B
harnesses. Docked for the fact that the overall product shape (plan mode, hooks,
todo reminders, subagent types) is a Claude Code pattern language. crush 7,
nanocoder 7 — waveloom is clearly above them; below codex/pi's 9.

### durability — 5

Single contributor, shallow clone (bus factor 1, no institutional backing, no
CLA/funding evidence). Counterweights: fresh HEAD (2026-09-14), disciplined
bilingual CHANGELOG through v0.9.1, SECURITY.md with PGP + 48h SLA, release
workflow + Homebrew tap + install scripts, CI discipline. Between nanocoder 4
(one-person, thin) and pi 7 (strong community). 5.

### docs-dx — 8

12 topics × 2 languages in-repo (prefix-cache, acp, mcp, settings, lsp, usage,
faq, install, system-prompt, tool-descriptions), CONTRIBUTING.{en,}md, AGENTS.md,
settings.example.json, .env.example, bilingual help text, setup wizard. Docs
match observed behavior everywhere I checked except one stale artifact:
`eval/llmedit/README.md:8-16` documents a Layer-1 eval suite at `pkg/hashline/eval`
plus `cmd/editbench`/`cmd/llmedit` commands that no longer exist at those paths.
Between cline's 8 (docked for confusion) and pi's 9; 8.

## Totals

| dim | score | weight | w.score |
|---|---|---|---|
| architecture | 7 | 15 | 10.5 |
| verification | 7 | 15 | 10.5 |
| safety-enforcement | 6 | 10 | 6.0 |
| token-economy | 9 | 10 | 9.0 |
| orchestration | 7 | 10 | 7.0 |
| interop | 7 | 10 | 7.0 |
| operability | 8 | 10 | 8.0 |
| originality | 8 | 10 | 8.0 |
| durability | 5 | 5 | 2.5 |
| docs-dx | 8 | 5 | 4.0 |
| **total** | | | **72.5 (B)** |

Strongest: token-economy (9). Weakest: durability (5).

Band-boundary check: 72.5 is not within 2 pts of 65 or 78; no boundary note.

## Provenance & calibration

- Census flag "original" accepted: manifest `github.com/Menfre01/waveloom`,
  Apache-2.0 file, NOTICE lists only third-party libs. Deep behavioral borrowing
  from Claude Code (plan mode, opusplan→"proplan", hooks, `.claude/*` compat)
  is pattern-level, not code-level; no vendored blobs found.
- No rule (a) relationship (no upstream remote in corpus); no demotions applied.
- License Apache-2.0 → concepts and code-adjacent portables allowed.
- Census correction required: test_loc 0 → 73,104; non_test_loc 109,605 → ~60,954
  Go non-test LOC (total Go 134,058).
