# ipsupport-code — T1 review

Go, single static binary, Bubble Tea TUI coding agent for any OpenAI-compatible
provider (local-first). Module `github.com/ipsupport-llc/ipsupport-code`, MIT
(file present). Self-learning loop: reason→act→observe over fat tools
(`file`, `run`, `git`, `web`, `calc`), post-task reflection writes "pitfall"
lessons that are re-injected into later tool-error results.

## Census sanity

- **test_loc: 350 is badly wrong.** Actual: 46 colocated `*_test.go` files,
  23,520 lines (largest: `cmd/agent/main_test.go` 9,469;
  `internal/agent/agent_test.go` 3,910). The census glob missed colocated Go
  tests entirely — same failure class as the crush/cline anchors.
- **non_test_loc: 46,949 overstated.** Actual Go non-test: 25,743 lines
  (111 .go files minus tests). Whole tree incl. docs/html/py: ~61k.
- Shallow clone (`.git/shallow`, single visible commit) with head_date
  2026-09-25 (4 days old at review): **no activity claims either way**; CI
  badge, dependabot merge at HEAD, and 4 workflows are the only liveness
  signals, all consistent with active.
- Tier: T1 stands on the corrected numbers (5k–100k).

## Provenance

Census: original, no fork flag; module path matches repo name; no vendored
upstream blobs found; no identity collision with corpus twins (not in the
claw-code/kimi/grok/forge/nanocoder/codex confusion list). License MIT file —
porting allowed in principle, but findings stay concept-level per protocol.

## Anchor question

**Which anchor is this closer to, and why?** Crush (69.0, B): same shape —
single-binary funded-by-nobody TUI agent with a tested approval policy,
loop-detection breaker, and sqlite-free file persistence, and the same missing
rungs (no prompt-cache discipline, no in-CI evals) — but ipsupport clears two
rungs crush stays on: kernel sandboxing that CI actually exercises, and
property-spec test density closer to the 8-rung per LOC; it remains below
cline's SDK-first multi-surface architecture.

## Dimensions

### architecture — 7.5 / 10 (weight 15)

Clean, small, single-purpose packages: `internal/agent` (loop, UI-free,
`agent.go:920 Run` over an `llm.Chatter` interface), `policy`, `sandbox`,
`risk`, `tool`, `knowledge`, `reflect`, `mcp`, `usage`, `trace` — the loop
takes interfaces and is testable without a product (`agent_test.go` drives it
with a scripted chatter). This is cleaner loop/product separation than crush's
fused `agent.go` (anchor 7). Docked hard on the product surface per the pi
errata: `cmd/agent/main.go` **5,981 LOC** and `cmd/agent/tui.go` 3,375 LOC —
above cline's 2.7k docking threshold (anchor 8). `main_test.go` at 9,469 LOC
is the same god-file signal on the test side.

### verification — 7.5 / 10 (weight 15)

23.5k LOC of tests that read like property specs, not existence checks:
policy evasion (`policy_test.go:160 TestRunArgvFloorResistsEvasion`,
`:186 ...SeesThroughWrappers`, `:320 TestResolveRejectsSymlinkChainDeeperThanChaseLimit`),
compaction survival (`agent_test.go:814 TestCompactPreservesActionDigestsVerbatim`,
`:846 TestCompactDigestSurvivesSecondCompaction`), trim overflow fallback
(`agent_test.go:263`, failure-signal preservation `:299`), race-semantics TUI
tests (`tui_e2e_test.go:354,471,531` no-race specs) via charmbracelet/teatest
(go.mod:15). E2E against fake providers via httptest across 9 test files
(e.g. `internal/e2e/e2e_test.go:312 TestE2E_Binary`). CI (`ci.yml`): gofmt,
vet, `go test -race`, host + darwin/windows cross-builds, **a macos-latest job
running real Seatbelt confinement tests** (`ci.yml:60-68` "sandbox tests (real
Seatbelt)"), and a committed-dataset-staleness gate (`ci.yml:33-46`).
Not 8: race matrix is ubuntu-only (crush anchor 7 runs 3-OS), no govulncheck,
and no in-CI model evals / no fuzzing — the documented 8-ceiling.

### safety-enforcement — 7 / 10 (weight 10)

Default-ask policy engine binds in-loop and is tested end-to-end
(`internal/e2e/e2e_test.go:220 TestE2E_PolicyDeniesDestructiveShell`;
run-tool gate at `internal/tool/run.go:97-100`). Beyond the "approval only"
6-rung: argv-aware hard floor that sees through `xargs/env/nohup` wrappers and
is reorder-proof on `rm -r` (`policy.go:190-212`), allow-globs must match
**every** chained segment and refuse command substitution and redirections
(`policy.go:134-157`), symlink-chasing jail with dangling-target and
chain-depth tests, secret-read floor that config cannot disable
(`policy.go:249`), TOCTOU re-check of deny-globs after async approval
(`policy.go:234-241 DeniedWrite`), and external CLI agents (codex/claude/
aider…) as a separate trust class that always re-prompts and is excluded from
spawn session-grants (`cmd/agent/external_agent.go:18-22`). Plus real kernel
containment: Seatbelt SBPL write-jail (`sandbox.go:60-86`) and Landlock
re-exec shim that **fails CLOSED** on kernel miss (`wrap_linux.go:60-61`),
Seatbelt tests run on a real macOS runner. Why not 8: sandbox default is
`off` (`config.go:222-226`), and auto-mode on an unsupporting kernel warns and
runs unconfined (`main.go:2728-2729`) — enforcement-under-approval is opt-in.
Honesty about this is exemplary (config comments say "off (default)" and the
warn is loud), which keeps it above the nanocoder 5 (opt-in + fail-open +
regex-denylist core); its core policy layer is not a regex deny-list.
Note: `sandbox.go:3` still says "Landlock on Linux is planned" while
`wrap_linux.go` fully implements it — doc drift, opposite direction of
misleading.

### token-economy — 7.5 / 10 (weight 10)

Two mechanisms, both tested. (1) Cross-task LLM compaction whose replacement
message re-embeds a **deterministic actions-digest** (files touched, commands
run, failure reasons) verbatim under a shared marker so it survives N further
compactions (`agent.go:473-535`, `agent.go:557`, tests above). (2) Mid-task
deterministic trim at 85% of window shrinking only large old tool results,
with an overflow second pass (`agent.go:632-688`), and — the standout —
`tokenScale` (`agent.go:666-680`) calibrates the 4-bytes/token estimate
against the server's own `prompt_tokens` from the *as-sent* request, clamped
0.5–4, born from a documented live blowout (32.8k window, HTML estimating at
half true density). Plus persistent usage ledger by day/provider/model with
pricing (`internal/usage/usage.go:1-5`), live context meter tested to read the
main-turn figure (`tui_e2e_test.go:573`). No prompt-cache discipline at all
(client.go has zero cache handling) — cline's 7 docks for the same; pi's 8
adds cache warming and branch summarization, which this lacks.

### orchestration — 7 / 10 (weight 10)

Background fire-and-forget sub-agent jobs with their own contexts (esc does
not kill them), kill/list, results folded in at next turn boundary
(`jobs.go:14-40,142`); LLM sub-agents on other model profiles + external CLI
sub-agents; spawn policy with its own ask/exec gates (`config.go:359-364`);
loop-detection breaker (`agent.go:1316-1350` callSig repeat-or-all-fail, one
nudge then stop, reset only on productive turns — same design as crush's
anchor-7 breaker); goal-judge TTL ("returns") with persisted goal + Missing +
Progressed so `/goal go` resumes (`agent.go:1767 judgeOnGiveUp`, Transcript
fields `agent.go:29-50`). No durable queue, no daemon, jobs and rewind
checkpoints die with the process — below codex's 9, level with crush/pi 7.

### interop — 7 / 10 (weight 10)

MCP client (stdio + http) exposed as a proxy tool (`internal/mcp/`,
`config.go mcp_servers`); **external-agent surface**: catalog of non-interactive
launch shapes for codex/claude/gemini/qwen/aider/goose/opencode with PATH scan
(`external_agent.go:34-45`) — consuming rival harnesses as sub-agents is
rare; clean headless contract (one-shot task arg, repeatable `-override`
dotted keys, named sessions, `-it` bridge; `main.go:84-96`). Missing: MCP
server, ACP, published SDK, any IDE surface — crush/nanocoder 7 rung.

### operability — 7.5 / 10 (weight 10)

Session persistence + named sessions + resume-across-restart **tested**
(`tui_e2e_test.go:72 TestSessionPersistsAcrossRestarts`); append-only archive
journal separate from the compaction-shrinking session file, cross-process
file-locked (`history.go:15-56`); `/rewind` per-turn file+history checkpoints
with unified-diff preview and `toobig` handling (`rewind.go:14-60`); usage
ledger with multi-process merge-on-save (`usage.go:36-45`); selfupdate that
refuses unchecksummed binaries (`selfupdate.go:99-107`); atomicfile/filelock/
procgroup plumbing under everything; JSONL trace of every decision
(`trace.go:1-5`). Docked: checkpoints are session-lifetime in-memory
(`main.go:448-450`) — a crash loses rewind state (cline's 8 git-stash
checkpoints survive); no crash-recovery middleware story at the crush-8 level.

### originality — 8 / 10 (weight 10)

Verified in code, not README: (1) **shadow-mode risk classifier** — a
linear SGD model trained offline by a stdlib-only Python script, embedded as
768KB `model.bin`, scoring every tool call and logging disagreements with the
policy while blocking nothing (`risk/shadow.go:40-92`, `cmd/agent/risk.go:19`
on-by-default with env escape hatch, per-workspace learned delta corrections
`risk.go:41-57`); (2) **cross-language feature-parity as a wire format**:
`CallText` and the hash are pinned by identical constants in
`scripts/train_risk.py:20-30` and `features_test.go`, monthly retrain CI that
**refuses to open the PR if held-out precision/recall regresses >0.05**
(`risk-vocab.yml:70-95`); (3) reflection-distilled pitfalls with a distinct
`avoid` kind injected into *tool-error results* on pattern match, never
prose-padded (`knowledge/pitfall.go:12-37`, `agent.go:2238,2263-2284`);
(4) goal-judge semantics "silence is not acceptance" — an unclear verdict
re-feeds rather than ends (`agent.go:1221-1237`); (5) `tokenScale`
calibration; (6) trace-as-training-dataset with FINETUNING.md. Not 9: nothing
here is a field-shaping substrate the way codex/pi's are.

### durability — 4 / 10 (weight 5)

Single contributor, 1 commit visible (shallow — cannot infer history depth;
per appendix rule no dead/low-activity claim; head 4 days old + dependabot
merge + 4 maintained workflows say active). Bus factor 1, LLC branding but no
visible institutional backing, no SECURITY.md. Same rung as the nanocoder
anchor (4); CI/release hygiene (checksums.txt enforcement, nightly,
dependabot) is better than that rung suggests, which is what keeps it off 3.

### docs-dx — 6.5 / 10 (weight 5)

851-line README with the honesty culture matching the code; docs/ is the
website (guide.html 326 lines); FINETUNING.md 188 lines; install.sh/ps1;
error messages are treated as a tested surface (e.g.
`agent_test.go:1028 TestRunNamesTheOutputCapWhenTheModelWasCutOffMidReply`,
`offlineMsg` web.go:48). Missing SECURITY.md, no in-repo reference-doc tree —
crush 6 / nanocoder 7 bracket.

## Totals

| dim | score | w | contrib |
|---|---|---|---|
| architecture | 7.5 | 15 | 11.25 |
| verification | 7.5 | 15 | 11.25 |
| safety-enforcement | 7 | 10 | 7.0 |
| token-economy | 7.5 | 10 | 7.5 |
| orchestration | 7 | 10 | 7.0 |
| interop | 7 | 10 | 7.0 |
| operability | 7.5 | 10 | 7.5 |
| originality | 8 | 10 | 8.0 |
| durability | 4 | 5 | 2.0 |
| docs-dx | 6.5 | 5 | 3.25 |
| **weighted total** | | | **71.75** |

**Band B.** Strongest dimension: originality (8). Weakest: durability (4).
Boundary check: 71.75 is >2 from 78/65 — no boundary-risk flag.

## Calibration notes

- Not a fork; no rule (a) constraint. No archived/dead status (shallow clone,
  recent head) — no cap applied.
- Verification held at 7.5 per the errata ceiling note (no in-CI evals/fuzzing
  caps at 8; 3-OS matrix and volume keep it from 8 outright).
- Architecture docking applied per pi errata (product-mode file >5k: main.go
  5,981).
