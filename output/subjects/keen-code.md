# keen-code - T1 review

Go CLI coding agent (`github.com/mochow13/keen-code`, v0.57.0, MIT, bubbletea TUI).
Read-only static review. Shallow clone (1 commit, head 2026-09-25, recent - no activity claims made).

## Census sanity

- `test_loc: 0` is WRONG. 109 colocated `*_test.go` = 35,876 LOC (same glob-miss artifact as crush).
- `non_test_loc: 79,071` also inflated: real Go is 64,285 total, 28,409 non-test (`find . -name '*.go' | xargs wc -l`).
- Corrected size is comfortably mid-T1. Provenance `original` accepted: manifest module matches upstream remote, LICENSE is plain MIT, no fork delta to state.
- Identity hygiene: no name collision; do not conflate with `code` / `codex` subjects.

## Anchor question

Closer to **crush (69.0)** than anything else: same Go single-binary shape, same "tested approval, nothing underneath" safety rung (6), similar CI rigor (race + govulncheck) - but keen lands below crush on orchestration (no loop detection), operability (zero `recover()` middleware vs crush's everywhere), and durability (solo vs funded Charm), and above it on token-economy (real prompt-cache discipline vs crush's plain auto-summarize).

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | Clean UI/core seam: `AgentCore` interface (`internal/agentcore/agentcore.go:12-69`) with a thin adapter (`adapter.go:126-133`); shared tool exec + output compression (`internal/llm/tool_execution.go:39-63`, `compress/output.go:11-40`). Docked: the tool-loop scaffold (maxToolTurns, auto-compact hook, incomplete-exit, pending-state) is re-implemented in all 6 provider clients (`anthropic.go:673`, `bedrock.go:277`, `openai.go:516`, `genkit.go:26` maxToolTurns const, `openai_responses.go:214`, `openai_codex.go:81`). Largest file 1,237 LOC (`command_handlers.go`), no god file. |
| verification | 7 | 35,876 test LOC colocated, 1,386 test funcs. Property-style: compaction is transactional + rejects empty summaries (`auto_compaction_test.go:63,90`; `compaction_test.go:55`); permission resolution flows keyed-tested (`stream_permission_test.go:345-398`). CI: `-race`, coverage->codecov, vet, gofmt, govulncheck (`.github/workflows/go.yml`). Single-OS, no evals, no fuzzing - 8 ceiling per errata. |
| safety-enforcement | 6 | Approval binds at execution layer: plan-mode denies writes via wrapper tools while keeping definitions cache-stable (`tool_execution.go:91-104`, `state.go:188-190`); dangerous bash always prompts and session-allow never covers it (`permissions/requester.go:76,117-119`); guard blocks system paths, all `$HOME` dotfiles, gitignored paths, with symlink-rejecting allow-dirs (`filesystem/guard.go:31-41,44-79,117-133`). Nothing underneath: permitted bash runs in plain shell; project allow-list skips even the dangerous prompt (documented, `docs/permission-system.md`); no sandbox anywhere. Rung 6 = "tested approval, nothing underneath" - exact fit. |
| token-economy | 7 | Trigger measures the actual serialized outgoing request (`estimateAnthropicInput`, `anthropic.go:604-613`) against budget-10% (`core/context.go:30-35`), mid-loop, cancellable (`anthropic.go:628` Cancel handle). Prompt-cache: ephemeral breakpoints at stable-prefix end + last message (`applyAnthropicBlockCacheControl`, `anthropic.go:228-257`); tool registry kept definition-stable across plan-mode switches specifically for cache (`tool_execution.go:90` comment, `state.go:229`). Tool-result aging: TurnMemory strips results from future-turn history keeping placement+status (`core/message.go:19-32`), `/tool-history full` opt-in. Context breakdown rescaled to provider-reported usage (`state.go:392-410`). One tier only - no branch summarization - hence 7, not 8. |
| orchestration | 6 | `delegate_task`: up to 10 parallel bounded subagent tasks with per-profile provider/model/permissions (`tools/delegate_tool.go:16,109`, `subagents/runner.go`). Input queue capped at 5 while streaming (`repl.go:336,380`, `docs/queuing.md`). Pending provider-state survives an interrupted loop (`anthropic.go:75,795-799`). No loop detection, no budgets, no crash journal beyond sessions. |
| interop | 6 | MCP client via official go-sdk with none/api_key/oauth (`mcp/manager.go`); skills discovered from `.agents/skills`, `.keen/skills`, AND `.claude/skills` (`guard.go:236-242`); headless `keen run --format json --completion-signal` (`cli/cmd/root.go:165-169`); npm packaging. No MCP server, no ACP, no SDK, one surface. |
| operability | 6 | JSONL session transcripts with seq numbers, list/resume (`session/store.go:14-29`, `--resume`, `/sessions`); slog file logging (`cmd/main.go:26-31`); self-updater; `/context` breakdown. Zero `recover()` anywhere (grep = 0 non-test hits) - crash posture is bubbletea default; no checkpoints/rewind/fork. crush scores 8 here precisely for recover middleware keen lacks. |
| originality | 7 | TurnMemory cross-turn tool-activity projection (below, unique); MCP-server-as-generated-skill lazy discovery (`mcpskills/mcpskills.go:15-49`); `/adversary` second-model reviewer with a separate client and a registry stripped of write/bash/mcp/delegate (`state.go:259-283`); `/btw` side-channel one-shot (`state.go:243-258`). Hashline editing is openly ported from pi, not claimed. |
| durability | 4 | 1 contributor, shallow clone, no SECURITY.md. Counterweights: fast release cadence (v0.57.0, 1,144-line CHANGELOG), CodeQL + goreleaser + codecov, head 4 days old. nanocoder rung (4). |
| docs-dx | 7 | 16 in-repo docs (~3.7k LOC) that match code, incl. `permission-system.md` with exemplary honesty about what yolo/allow-list bypass and what still binds; TOUR.md, AGENTS.md, script+npm install. |

Weighted: 90+105+60+70+60+60+60+70+20+35 = **63.0** -> **C**. Strongest: token-economy. Weakest: durability.

Boundary note: 63.0 is within 2 pts of the B floor (65) - flag for synthesis.

## Anti-patterns / risks

- Per-provider loop duplication (finding keen-code-9): six copies of the loop scaffold will drift; the shared pieces (`tool_execution.go`, `auto_compaction.go`) prove the seam exists but is never taken.
- Default-on GA telemetry, opt-out via `KEEN_TELEMETRY`/`DO_NOT_TRACK`/CI (`telemetry/telemetry.go:81-92`).
- Project-level `allow: ["bash"]` skips the dangerous-command prompt entirely - documented, but a foot-gun shape (finding keen-code-6).
- Bash danger classifier is tokenized/segment/subshell-aware regex (`bashclassifier.go:81-120`) - better than flat denylists, still fundamentally bypassable (e.g. `find -delete` not covered).
