# Boundary re-review: keen-code (independent, 2026-09-30)

Provisional 63.0 C (-2.0 under B floor; inclusive-2 rule). Go, ~28.4k prod LOC + 35,876 test LOC in 109 colocated `*_test.go`, solo author (mochow13), MIT, HEAD v0.57.0 (2026-09-25, 4d before study; shallow clone noted, HEAD recent so activity proven).

**Closest anchor: crush (69.0 B)** - same shape: solo-ish Go, tested in-loop approval with zero OS enforcement, one auto-summarize compaction strategy. Keen exceeds crush on wire-level cache/permission test assertions and lacks god files; it lacks crush's 3-OS race matrix, loop detection, and LSP. Placing keen ~5 under crush is coherent; codex/pi are a different class entirely.

## (1) Token economy: tested? measured? which rung?

Three claims, verified in code:

1. **Request-serialized compact trigger** - real. `proactivelyCompactHistory` runs before EVERY tool turn in the loop (`internal/llm/anthropic.go:672-674`), measuring `estimateAnthropicInput(*msgParams)` which JSON-marshals the actual outbound messages (`anthropic.go:553-563`) against `ContextInputBudget` = window - max(4096, window/20) (`internal/llm/core/context.go:22-28`) minus a further 10% headroom (`context.go:30-34`). Same pattern in all 5 provider paths (bedrock.go:367, openai.go:608, genkit.go:343, openai_codex.go:209, openai_responses.go:195). **Tested**: threshold boundary is property-tested exactly (`internal/llm/auto_compaction_test.go:105`: 85500 triggers, 85499 does not at budget 95000); AutoCompact itself tested for transactionality, private-replacement, final-run-only summarization, incomplete-stream rejection (`auto_compaction_test.go:33-99`). **Measured?**: request SHAPE is measured (serialized payload), but tokens are a `(len+2)/3` heuristic (`context.go:7-12`) and provider usage events (incl. `cached_tokens`, `anthropic.go:711-720`) are surfaced to UI/headless only - **no usage-feedback calibration back into the trigger**.
2. **Dual cache breakpoints** - real and tested at the wire. `applyAnthropicBlockCacheControl` (`anthropic.go:228-258`) clears all markers then sets system-last + tools-last + message breakpoints at the **stable turn-start boundary** (turnStartLen) and the **last message**, per turn; post-compaction the stable boundary resets (`anthropic.go:644`). Tests capture the actual request params and assert exact marker counts and stale-marker stripping: `anthropic_test.go:984` (3 markers), `:1030` (oneshot 2), `:1063` (stale pending markers stripped, count stays 3). This is genuine cache-prefix discipline, not decoration.
3. **TurnMemory projection** - real. Tool activity lives in a `TurnMemory` sidecar per assistant message (`internal/llm/core/message.go:31`, `agentcore/types.go:47`), outputs pass per-tool compression into `RetainedOutput` (`internal/cli/repl/turn_memory.go:82`; `compress/output.go:10-30` strips read metadata, prefix-folds glob lists, compacts grep), and the projection is what compaction history and the `/context` breakdown estimate consume (`core/context.go:76-84`; `appstate/state.go:392-420`; `cli/repl/context_status.go`). Session replay rebuilds conversation from events incl. TurnMemory (`session/projection.go`). Cost visibility: headless JSON emits `cached_tokens` (`headless_run.go:59,324`).

**Rung placement**: clearly above cline-7 ("no cache discipline") and above the crush/nanocoder 6 rung. Below the pi-8 pillar set: pi's trigger measures projected context precisely and it has cache warming and branch summarization - keen has neither warming nor exact tokens (heuristic estimator). Below the corpus's 8.5-9 cluster: san's cache-monotonic guarantee, jazz's measured 4-rung ladder, waveloom's measured injection economics (26% of cache-miss tokens), octomind's price-ratio amortization all MEASURE realized economics; keen has no realized-cache-hit validation, no warming/keepalive, no compaction tiering (single summarize strategy), no model-call-free budget tier (codex-9 pillar). **Token-economy 7.5** - tested yes, measured at serialized-request granularity yes, but measured-ECONOMIES no. The B path ("only realistic path") fails: even the generous reading (token 8 + verification 8) tops out at ~65.0 exactly on the line with zero credit elsewhere changed; central read is C.

## (2) Verification: what blocks in CI?

`.github/workflows/go.yml`: `go build -v ./...` then **`go test -v -race ./...` blocking on every push and PR (go.yml:29-30)**; plus go vet + gofmt gate (`:60-70`) and a blocking **govulncheck** job (`:79-93`); separate CodeQL workflow. So all 35.9k test LOC DO block. Test quality is property-asserting, not existence: approval-iff-classifier tests including denial paths (`tools/bash_test.go:255,477` - the latter proves the classifier binds WITHOUT the model's `isDangerous` flag), classifier property suites over compound commands, subshells, redirections, env-secret exposure (`bashclassifier_test.go:211,265,145,292`), wire-captured cache-marker counts (`anthropic_test.go:984+`), compaction transactionality (`auto_compaction_test.go:63`). 44 files use table subtests; 1,386 test funcs total.

Ceilings: **zero fuzz targets** repo-wide (grep `func Fuzz` = 0 hits), no model evals anywhere, coverage upload explicitly non-blocking (`go.yml` codecov `fail_ci_if_error: false`), single OS. ERRATA verification-8 ceiling stands: real corps but nothing separating 8 from 10; and the 7-vs-8 call goes 7.5 - cline/pi/codex 8s are 6-10x the asserting mass and codex drives full approval flows against scripted servers; keen's wire-level approvals testing is the right SHAPE at a fraction of the scale, race on one OS vs crush's 3-OS matrix (also 7). **Verification 7.5.**

## (3) Safety: 6-rung or dock?

**6-rung confirmed, no dock, and notably it correctly does NOT repeat neovate's approval-scope amnesia.** Default shipped posture = ModeBuild with the live requester (`cli/repl/repl.go:177`); yolo is opt-in `/mode` (`command_handlers.go:423-1264`); approval binds in-tool (`tools/bash.go:133-162`) and is tested incl. the no-requester fail-closed path (`bash_test.go:294`).

Neovate precedent check: neovate's hole was an always-allow cache keyed on category and replayed without re-checking per-command risk. Keen's session grant IS also toolName-granular (`permissions/requester.go:97`) but is **never honored for elevated risk**: `if !isDangerous && r.sessionAllowedTools[toolName]` (`requester.go:73`) - a dangerous command always re-prompts even after "allow session" for bash; and danger is computed independently of the model's self-declared flag (`bash.go:151`: `isDangerous || IsDangerousCommand(command)`), with the classifier covering privilege escalation, subcommand maps, conditional flags, secret-exposure regexes, and recursive subshell expansion (`bashclassifier.go:60-140`), all property-tested. Correct scope narrowing on the cache key - filed as a positive portable (b4).

Why not 7 (mocode bar: fingerprint grants, fail-closed off-TTY, hash-gated trust, realpath jail, all tested): grants are name-based, path guard is coarse (bash only checks `CheckPath(".")`, `bash.go:133` - command-internal paths uncontained except by classifier), classifier is token-map (interpreter payloads like `python -c "shutil.rmtree(...)"` sail through - corpus denylist-family limitation). Zero OS enforcement (seatbelt/landlock/seccomp/bwrap: zero hits repo-wide). Octomind precedent (all-layers-disabled = 5) does NOT apply - keen's layers ship ON. **Safety 6.**

Sub-floor finding (does not move the score, bounded): **subagents run under an unconditional AutoApprover** (`subagents/tool_factory.go:13-15`) - delegated registries include bash/write/edit/MCP with approvals always granted; containment is guard + profile tool-list only, and a dangerous command inside a subagent never surfaces a user prompt. Filed b5.

## Other lanes (independent read, for the total)

- **architecture 7.0**: no god files (largest product file 1,237 LOC, `cli/repl/command_handlers.go`), clean layer planes (llm/tools/filesystem/session/subagents), event-sourced session store with projection rebuild (`session/store.go`, `projection.go`). Docked: the tool-turn loop + trigger + cache-control is duplicated across 5 provider clients - thresholds/compaction are centralized helpers so the duplication is thin, but it is per-surface loop duplication (sibling of gemini's entry-path axis), and the REPL trio (1,237+1,132+1,051) is drifting toward one UI god-module.
- **orchestration 5.0**: profile-based subagents discovered from `.agents/.keen/.claude/agents` markdown (`subagents/discover.go:111-119`), parallel `RunDelegate` instances, per-profile timeouts (`runner.go:138`); no queues (docs "queuing" = REPL input queue only), no resume journal, no loop detection, no budgets.
- **interop 6.0**: MCP client with OAuth (docs + `internal/mcp`), headless JSON run with usage/cost (`headless_run.go`), rival-harness import surfaces (`.claude/agents`, `.claude` dirs); no ACP, no SDK, no IDE, no LSP.
- **operability 6.0**: JSONL session events with list/load/prune (`session/store.go:53-97`) and replay; no checkpoint/rewind; telemetry GA-based but honors DO_NOT_TRACK/CI and env override (`telemetry/telemetry.go:85-89`).
- **originality 5.5**: TurnMemory sidecar + serialized trigger + stable-boundary breakpoints are all corpus-attested shapes elsewhere; good engineering, no field-defining idea.
- **durability 5.5**: solo, no SECURITY.md, no external-PR governance visible; offset by 126 Keep-a-Changelog releases, semver discipline, head 4d fresh.
- **docs-dx 6.5**: 16-doc tree incl. architecture.md, compaction.md, permission-system.md, turn-memory.md matching code; docs are accurate per spot-checks (compaction.md describes the two forms found in code).

## Verdict

| dim | w | lane | weighted |
|---|---|---|---|
| architecture | 15 | 7.0 | 10.5 |
| verification | 15 | 7.5 | 11.25 |
| safety | 10 | 6.0 | 6.0 |
| token-economy | 10 | 7.5 | 7.5 |
| orchestration | 10 | 5.0 | 5.0 |
| interop | 10 | 6.0 | 6.0 |
| operability | 10 | 6.0 | 6.0 |
| originality | 10 | 5.5 | 5.5 |
| durability | 5 | 5.5 | 2.75 |
| docs-dx | 5 | 6.5 | 3.25 |
| **total** | | | **63.75 / C** |

**B floor NOT crossed.** Token-economy is genuinely tested and request-serialized - the strongest claim of the three - but it sits at cline-plus / pi-minus, below the san/jazz/waveloom measured-economics cluster, and cannot carry the 2.0. Generosity bound (verification 8 + token 8, all else unchanged) lands at 65.0 exactly on the line, which per the study's boundary doctrine (one fresh reviewer decides; claurst-style line-hugging without a mechanism is not a crossing) does not flip the band. C FINAL at 63.75.

Consistency note for the 63-65 comparison set (codebuff +2, zot +2, keen -2): keen's verification is ABOVE codebuff-tier (public CI runs everything vs codebuff's build+smoke-only public mirror), but its orchestration/originality floors sit lower than codebuff's - same weighted neighborhood, band stable.
