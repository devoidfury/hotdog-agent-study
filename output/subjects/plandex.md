# plandex — T1 review

Go CLI + server agent ("Open source AI coding agent. Designed for large projects").
Census: 87,141 non-test / 2,408 test LOC, contributors 1, head 2025-10-03, shallow clone.

## Census sanity

- **LOC overcount**: actual Go non-test = 74,240 LOC (`find *.go ! -name *_test.go`), tests = 2,511 across 6 `*_test.go`. Census 87,141 is ~13k high (glob artifact). Tier T1 still holds.
- **Shallow clone confirmed** (`.git/shallow`, commits=1). Remote-verified per protocol: GitHub API `pushed_at: 2025-10-03T21:49:58Z`, `archived: false`, latest commits list agrees with HEAD (`e2d77207 "link to cloud wind down post"`; prior commit `4577abc "updates for Plandex Cloud wind down beginning 10/3/2025"`).
- **Activity**: 361 days since last push as of 2026-09-29 — under the 12-month rule (b) trigger by 4 days. Rule (b) cap NOT applied per dispatcher instruction; flagged for synthesis (repo will cross 12mo on 2026-10-03, and hosted product is winding down — practically dead, single author Dane Schneider).

## Provenance

Original (`plandex-ai/plandex`, GitHub API `fork: false`). No fork delta to state. Rule (a) n/a. License: MIT (file + GitHub API), so code-adjacent portables are permitted; no license-risk finding.

## Anchor question

**Closer to crush** than any other anchor: Go, client-CLI/server split with all agent state server-side in a DB, plan/session persistence, approval-gated exec with no sandbox — but plandex separates loop/policy/state better than crush's fused `agent.go`, while crushing crush on verification and interop, so it lands a band lower (C).

## Dimension scores

### architecture — 7/10 (w15)
Clean layered Go layout: `app/cli` (TUI/commands), `app/server` (plan state machine + providers), `app/shared` (data models), `app/server/syntax` (edit application engine). The tell loop is a stream state machine split by concern (`app/server/model/plan/tell_exec.go:86` `execTellPlan` with per-stage state in `tell_state.go`, `tell_stage.go`, stream processing in `tell_stream_processor.go`), build pipeline separate (`build_exec.go`, `build_validate_and_fix.go:44` `buildValidateLoop`). Residuals keep it off the 8 rung: largest files are dispatch/wiring rather than loop god-files but still chunky — `app/cli/api/methods.go` 2,454, `app/cli/cmd/repl.go` 1,401, `app/cli/lib/context_update.go` 1,063; state model is server-side plan rows + per-branch active-plan map with channel-based stream-done signaling (`tell_exec.go:113-124` panic recover into `active.StreamDoneCh`).

### verification — 3/10 (w15)
6 `*_test.go` files, 2,511 LOC against 74k non-test (3.4%). The tests that exist are real property tables, not existence checks — `app/server/syntax/structured_edits_test.go:12` `TestStructuredReplacements` covers ellipsis-anchor application incl. "bad formatting" cases; `tell_stream_processor_test.go:9` `TestBufferOrStream`; `unique_replacement_test.go:7`. But **no test CI at all**: `.github/workflows/` contains only `docker-publish.yml`. Promptfoo eval harness exists only as an unwired POC (`test/evals/promptfoo-poc/README.md`, `evals.md`). Between the codel 0 ("nothing runs") and the 4 rung ("happy paths"); the ratio and CI absence dominate. Reuses `test-suite-without-ci`.

### safety-enforcement — 4/10 (w10)
Approval-gated exec with automode presets (`app/shared/plan_config.go:22-36` AutoModeFull..None; `:130-143` Full sets `CanExec=true, AutoExec=true`), and defaults to Semi (`:492` `DefaultPlanConfig.SetAutoMode(AutoModeSemi)`) — so defaults are honest. But nothing binds underneath approval: shell commands run via `exec.Command(shell, "-c", scriptPath)` (`app/cli/lib/apply.go:399`) with no sandbox, no path containment, no deny policy anywhere in tree (grep for sandbox across cli/shared: nothing). `docs/docs/security.md` is honest about the actual posture (secret-file protection via .gitignore/.plandexignore, ephemeral API keys) rather than overclaiming. Above codel's 2 (real binding setting-gate, honest docs), below the 6 rung (untested approvals, nothing underneath). Reuses `approval-only-enforcement`.

### token-economy — 7/10 (w10)
Multiple complementary mechanisms, all real in code:
- Overflow-triggered conversation summarization with summary-graft search: `app/server/model/plan/tell_summary.go:60-133` (finds a `ConvoSummary` timestamped to fit under `GetPlannerEffectiveMaxTokens`, errors if still over).
- Pre-request dry-run token fitting: `tell_exec.go:584` `dryRunCalculateTokensWithoutContext`.
- Per-build pending-changes summary instead of raw diff replay: `app/shared/plan_result_pending_summary.go:10` `PendingChangesSummaryForBuild`.
- Static prompt-cache breakpoints: `CacheControlTypeEphemeral` on system prompt parts at each phase boundary (`app/server/model/plan/tell_sys_prompt.go:70,91,99,141,149`).
Model-role separation (planner/build models, `app/shared/ai_models_roles.go`) adds cost routing. Docked from 8: triggers are reactive (overflow) not projected like pi's usage-measured compaction; cache marking is static, no warming/stat discipline.

### orchestration — 6/10 (w10)
Server-side plan state machine with statuses, branches (`app/cli/cmd/branches.go`, `db/git.go` per-branch shas), subtask lists fed back into prompts (`tell_subtasks.go:14-24`), auto-continue loop, and a capped auto-debug repair loop: on exec failure, rollback plan + re-tell with output + rebuild, recursive up to `AutoDebugTries` (`app/cli/plan_exec/apply_exec.go:26-168`). Panic recovery at loop entry (`tell_exec.go:113-124`) + error notification fanout (`notify.NotifyErr`). No subagents, no queue/daemon plane, crash recovery of in-flight streams is via status rows, not journals. Standard design, unproven crash semantics → 6 rung.

### interop — 3/10 (w10)
No MCP anywhere (grep hits in `app/` are unrelated substrings), no ACP, no published SDK, no structured headless output contract (`plandex tell` exists non-interactively, `app/cli/cmd/tell.go:25`, but output is human TUI). Server API is private to its own CLI/web. Above codel's 2 only because self-host is a real documented server (`app/server/Dockerfile`, `docker-compose.yml`) with a broad CLI surface.

### operability — 8/10 (w10)
Strong plan lifecycle: every plan versioned in its own git repo server-side (`app/server/db/git.go:37` `InitGitRepo`, `:95` `GitRewindToSha`, `:197` commit history), rewind/checkout/diffs/branches/archiving CLI verbs (`rewind.go`, `diffs.go`, `checkout.go`, `archive.go`), `AutoRevertOnRewind` in full-auto (`plan_config.go:86`, `:144`), concurrent-plan locking (`app/server/db/locks.go`, 668 LOC), usage report (`app/cli/cmd/usage.go:39-50`), model packs/providers config. Matches the crush 8 rung (recover middleware, locks, datadir discipline); short of 9 — no rollout-journal/debug-tooling story, and TUI-first resume.

### originality — 8/10 (w10)
The ellipsis-anchored structured-edits format ("// ... existing code ...") resolved against tree-sitter nodes with fallback matching (`app/server/syntax/structured_edits_tree_sitter.go:119` `findNextAnchor`; `structured_edits_generic.go`, `unique_replacement.go`) — historically field-defining and the format other harnesses adopted; genuinely defended by the corpus's best test file. Also distinctive: plan-as-server-checkpoint (git per plan), request-scoped ephemeral API keys, planning-phase vs implementation-phase split with model-driven `load` context gathering. Not 9: no loop detection, no novel caching or subagent model.

### durability — 1/10 (w5)
Single contributor (remote-verified, all recent commits by Dane Schneider), 15.7k stars but activity is a wind-down: cloud shutdown commits at HEAD, no CI tests, no SECURITY.md. Not scored 0 only because repo is not archived and the >12mo remote check hasn't tripped yet (361 days). Will be a rule-(b) candidate at synthesis within days of this review.

### docs-dx — 7/10 (w5)
Full Docusaurus tree in-repo (`docs/docs/`: cli-reference.md, quick-start, repl, environment-variables, core-concepts, hosting incl. self-hosting, models, security.md) that matches code behavior (security.md claims match what I verified in code). Install script (`app/cli/install.sh`). Docked: no SECURITY.md, some cloud-mode drift after wind-down.

## Totals

| dim | score | weight | weighted |
|---|---|---|---|
| architecture | 7 | 15 | 10.5 |
| verification | 3 | 15 | 4.5 |
| safety-enforcement | 4 | 10 | 4 |
| token-economy | 7 | 10 | 7 |
| orchestration | 6 | 10 | 6 |
| interop | 3 | 10 | 3 |
| operability | 8 | 10 | 8 |
| originality | 8 | 10 | 8 |
| durability | 1 | 5 | 0.5 |
| docs-dx | 7 | 5 | 3.5 |
| **total** | | | **55.0 (C)** |

Strongest dimension: **operability** (8; tied raw with originality, but the plan-versioning machinery is the deepest implementation here). Weakest: **durability** (1).

Note vs anchors: lands below nanocoder (60.5) despite better architecture/originality — the verification and interop gaps (no test CI, 3.4% test ratio, no MCP/ACP/headless contract) are decisive at these weights, which matches the "score against anchors, not goodwill" rule.

## Calibration notes

- Rule (b) not applied: remote-verified last push 2025-10-03 = 361 days < 12 months. No silent demotion. Synthesis should re-check after 2026-10-03.
- Census LOC corrected in-report: 74,240 non-test / 2,511 test (census 87,141 / 2,408).
- MIT license: portables may be code-adjacent.
