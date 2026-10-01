# memcode — T2 deep review

Repo: /data/samples/agents/memcode (github.com/memcode-ai/memcode, Go, MIT file-license, census provenance "original").
Snapshot: shallow clone (.git/shallow, 1 commit visible, head 2026-09-13 => no activity claims either direction).

## Census corrections

- `test_loc: 0` is WRONG. 477 `*_test.go` files, 82,562 test LOC. Same glob-miss class as crush/cline anchors.
- `non_test_loc: 178,377` includes 41,475 LOC of vendored fork `internal/forks/vaxis` (TUI renderer, see below). First-party non-test Go is ~96.2k. Tier stays T2 either way.
- contributors=1 / commits=1 are shallow-clone artifacts; evidence-limited, not asserted.

## Anchor question (one sentence, before scoring)

Closer to **cline** than to any other anchor: both pair a genuinely property-asserting test corps with a broad product surface and an architecture docked for a central hub, and like cline memcode sits just under the verification-8 ceiling (no in-CI model evals, no fuzzing) — landing fractionally below cline's 76.5 overall.

## Mandatory line-level reads

### Core loop — internal/agent/runtime/loop.go (1339) + runtime.go (1452) + exec.go (2011)

- `runLoop` at loop.go:160 drives model↔tool until stop/iter-cap; per-turn state is a fresh explicit struct (`s.turn = newTurnState()`, :168), and the plan lifecycle is a separate state machine (`planCtl` with BeginTurn/Present/Approve/Phase; testhelper_test.go:22-44 forces tests through the REAL transitions — "raw phase pokes are impossible now, which is the point").
- Bounded-recovery discipline throughout, each bound commented with the failure it kills: overflow reactive-compact-retry ≤2 (loop.go:24-27), stream-cut per-call retry ≤3 with the "history intact because tools run only after a successful call" argument (:253-268), stall resume ≤2 with placeholder assistant to avoid empty-message 400s (:419-433), tool-call-leak self-heal for vLLM streaming bug (:326-340), apply-continuation bounded by todo-progress signature `todoSig` rather than round count (:39-62, :345-366) — progress-bounded, not round-bounded.
- The NO-PROGRESS stall/loop detector was deliberately REMOVED (loop.go:93-95: "the pure runaway/token backstop, since the no-progress stall detector was removed for killing legitimate long work") — iter caps + progress signatures replace a repetition breaker. Corpus-relevant counter-data for loop-detection.
- Safe-boundary doctrine is explicit: steering, eviction, and compaction all fold only after tool_use/tool_result pairing completes (:506-530).
- Iteration cap ladder `loopIterCap` (:96-105): 200 soft; allow-all AND approved-plan apply get the yolo ceiling.
- exec.go wires policy into every tool: readOnly session gates per-tool (:240-262), bash risk-classified via `permissions.ClassifyBash` at :1672, planning/readOnly sessions restricted to Safe bash only (:1685), sandbox wrap at :1734.
- Residual: Session is an acknowledged hub ("carved off the agent Session god-object", ledger.go:1-4; struct itself ~150 lines at runtime.go:76), runtime package is ~130 files, exec.go 2011. No 3k+ god file, but the hub shape is real.

### Compaction — internal/agent/compaction/compaction.go (376) + runtime/compact.go (370)

- Three-layer doctrine (hot raw / warm summary / cold session log, COMPACTION.md); the pure package owns the provable invariant "never split tool_use from tool_result" — cuts only at user-turn starts (`isTurnStart` excludes tool_result carriers, compaction.go:47-62; tested compaction_test.go:67 TestPlanNeverSplitsToolPair, :213 TestToolResultIsNotATurnStart).
- Eviction replaces payloads with TYPED, lossless pointers back to source ("read_file X lines 500-600 — re-read that range to restore", compaction.go:152-177); superseded reads (same path+range re-read later; a later FULL read covers ranges) are offloaded eagerly with range-awareness (:217-260); HOT-path pinning with per-turn decay and a cap of 12 (compact.go:78-118) — the fix for a MEASURED read-evict-reread thrash ("same 9 files re-read 13-14x", :56-63). Keep-count scales with budget (8..32) instead of a constant.
- Budgets are relative to the learned serving lane capacity ×85%, deliberately NO absolute token constants ("every prior magic number aged into a silent clip", compact.go:30-37).
- Compaction back-off `compactWouldHelp`: re-summarize only after ≥20% real regrowth — explicitly to stop prefix-cache-busting "3 compactions in 8 minutes" (:166-175). Proactive path keeps 8 turns; keep=2 reserved for reactive overflow (:252-256).
- Summarizer is force-escalated to the strong model ("a wrong summary silently becomes the session's false memory", :262-266); summary redacted then persisted to the episodic log (:344-350); the compacted-marker tells the model exactly what survives in the cold layer and forbids reconstructing tool output from memory (:281-290).

### Permissions / sandbox — internal/agent/permissions (permissions.go 1508 + deny.go + file.go) + internal/sandbox

- Real shell-AST classification via mvdan.cc/sh (ClassifyBash doc :433-445). Classifier intent: "when in doubt, classify HIGHER; Safe is reserved for operations positively recognized as read-only" (:74-77).
- Depth here is unusual: wrapper value-flag tables per wrapper so `timeout 5 rm` / `ssh -p 22 host rm` land on the inner head (:124-166); `find -delete`/`-exec` and `fd -x` inner-command extraction (:206-228); bare-shell-receiving-pipe detection incl. `-s` bundling and `-- ` positional traps (:169-204); direction-based net risk (POST/upload = exfiltration crossing, :239-242); cloud-CLI verb tables incl. catastrophic bucket-delete `rb` (:244-272); `RiskHead`/`RiskSegment` keep the approval card's "don't ask again" pattern anchored to the HIGHEST-risk simple command, redirects included, so a saved rule can't under-cover (:347-431).
- Denylist is a capability ceiling independent of risk, walked on the same AST with the SAME unwrapper so ladder and ceiling cannot disagree (deny.go:14-22, :75-82); unparseable input is REFUSED, "a denylist that fails open is not a ceiling" (:44-47).
- Catastrophic floor survives allow-all (Decide :55-71); `RecoverableInRepo` downgrades only pure in-repo `rm` to Medium when git can restore it, conservatively false for globs/vars/out-of-repo/.git (:437+).
- Tests are property-shaped, not existence-shaped: bypass regressions (`\rm`, `$IFS`, `$(echo rm)`, `eval "rm -rf"`, pipe-into-any-interpreter, setsid/doas/flock/stdbuf/parallel) each with a paired no-over-classification assertion (classify_bypass_test.go:6-97); the standing battery (classifier_battery_test.go) is explicitly the "whenever the classifier mis-rates a command in the wild, add a case here" guard.
- Approval semantics go beyond yes/no: allow / allow-edited / deny-with-reason-fed-to-model / redirect-continue / interrupt (approval.go:37-47); remembered approvals live in a user-editable plaintext `.memcode/permissions` checked into the repo, not the state DB (file.go:11-31), with scoped remember options and "remember never covers catastrophic" tested (approval_test.go:206).
- Sandbox (sandbox.go 188): seatbelt+bwrap wrapping WITH ReadOnly mode always-on for read-only sessions, Workspace opt-in via MEMCODE_SANDBOX=1, fail-open when no backend — and honest about it ("the sandbox strengthens an existing gate, it must never brick a platform", :12-16). `Supported()` exists so grants that scale to containment (mcp_code_exec sandboxed+no-network=Medium) can tell "asked for" from "got" (:71-84). This is the nanocoder fail-open shape WITHOUT the wrong-default deception, and there is no default-on OS enforcement for normal sessions — above the 6 rung (approval is AST-gated, tested, with a capability denylist and honest defense-in-depth beneath), short of codex's 10.
- Safety hole found: project-shipped `.memcode/hooks.json` is loaded and executed through the platform shell with NO trust gate — hooks.go header (":3-6 project runs after user hooks", commands run via platform shell) and runtime/hooks.go:21 `s.hookSet = hooks.Load(s.root)`; grep for any trust/approval gate in hooks+config returns nothing. Since `.memcode` is the team-shared, portable memory dir, cloning a repo (or accepting a PR) that ships hooks = arbitrary code execution at session_start. (hook-trust-scoping.)

## Other dimension evidence

- **Token economy beyond compaction**: anthropic.go buildWire (:185-232): 1h cache breakpoint on the stable doctrine prefix, volatile per-turn facts split into a SEPARATE uncached block so facts never bust the prefix; last-tool breakpoint; 5-min conversation breakpoint; decoration on CLONES only — with cache_test.go:40-49 asserting the caller's structs are untouched. Cost visibility: ledger per-backend and per-purpose with cache read/write split (ledger.go:26-38); wire.jsonl diagnostics (`MEMCODE_TRACE`) capturing request/response shape per call (wiretrace.go:12-22).
- **Verification**: 477 test files / 82.5k LOC; CI = gofmt+vet+`go test -race`+staticcheck and a real-headless-Chrome browser suite per push (ci.yml:18-36); release gated on `go test -race` (release.yml:22-23). Scripted fake providers drive real end-to-end turns (session_dogfood_test.go:20-45: full StartChat→Submit→EndChat, on-disk log fidelity + agent self-recall; policy_test.go:20-30 recordingProv proves which model each worker ran on). BUT: loop_test.go is largely prompt-string lock tests; CI is ubuntu-only (no multi-OS matrix, no govulncheck); no fuzzing; the `memcode eval` A/B harness is NOT wired into CI. Verification 8 rung by the errata ceiling, at its bottom edge on CI breadth.
- **Orchestration**: sub-agents/scouts with policy narrowing tested (policy_test.go:51-211 — workers inherit the user's pin; explore narrows; the agent tool CANNOT name a model); autonomy package: standing objectives with DelegationPolicy over CONSEQUENCE CLASSES (observe…financial/legal_attestation/destructive) + action/token/seconds/depth budgets + ExpiresAt/Revoked + canonical-hash policy identity (autonomy/policy.go:14-52, policy_test.go); generated-code runner hard-fails when the hardened sandbox is unavailable (runner.go:29-35); gateway scheduler + occurrences (internal/gateway/server/scheduler_test.go, occurrence pkg); checkpoints = per-turn pre-images with honest limits ("agent-edit undo, not a filesystem time machine", checkpoint.go:1-8) + rewind.go. No codex-grade queue/daemon recovery; no loop breaker (see loop above). 7.
- **Interop**: MCP client (internal/mcp incl. oauth + approvals) AND `memcode mcp serve` — an OFFLINE MCP stdio server exposing the repo memory to rival harnesses ("a Claude Code / Cursor / Codex user can give their existing agent memcode's memory… without switching agents", cmd/mcp_serve.go:16-24). Rival-config importers `memcode claw migrate` / `memcode hermes migrate` (cmd/migrate.go:23-107). Claude-Code-compatible hooks semantics (exit-2 veto). Headless `run --protocol stream-json` (cmd/run.go:45,393). Documented extensible wire protocol with 4 named extensions (protocol/PROTOCOL.md:1-30). Twelve chat channels through a gateway with sender pairing (gateway/server/pairing.go). No ACP, no IDE surface, no published SDK. 8.
- **Operability**: episodic session log with search from inside the agent (compactedMarker signposts it), sessions dirs + wire trace diagnostics, `doctor`, wizards, self-update, migrate, /cost /analyze telemetry, checkpoint rewind. 8.
- **Originality** (verified in code, not README): repo-portable memory as THE product with a counterfactual measurement — `memcode eval` runs the same task twice in isolated worktrees, context-pack vs cold, comparing iterations/tool calls/tokens/verification (cmd/eval.go:23-33); mood/focus-adaptive interaction; offline BM25-with-doctrine recall with an explicit "no embeddings until a measured recall eval proves otherwise" stance (recall.go:1-15); lesson distillation at the break-and-repair moment (loop.go:395-401); cross-model plan critic that must cite claim→status→path:line evidence via forced tool schema (loop.go:646-700); two-system-message cache convention exported in the public protocol. 8.
- **Durability**: hosted product + billing lanes + goreleaser release CI; but shallow snapshot, no SECURITY.md, no visible governance; vendored vaxis fork with the upstream version NOT RECORDED at vendoring time (forks/doc.go:8-12) — supply-chain hygiene gap on 41.5k LOC. 5, evidence-limited.
- **Docs/DX**: COMPACTION.md / HOOKS.md / ROUTING.md / PROTOCOL.md / docs/gateway + docs/design, env-var knobs documented in code headers, error paths in the loop print fix-you-can-do messages ("Run /login to reconnect", loop.go:316-319). Thinner than pi's doc tree. 7.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7 | loop.go:160-230 clean turn loop + plan state machine; docked: Session hub (runtime.go:76 ~150 fields; "carved off the … god-object", ledger.go:1-4), exec.go:1-2011 |
| verification | 8 | classifier_battery_test.go + compaction_test.go:67,333,402,454 + cache_test.go:40-49 (no-mutation) + session_dogfood_test.go:39-45; CI ci.yml:18-36; ceiling: no evals-in-CI, no fuzzing, ubuntu-only |
| safety-enforcement | 7 | permissions.go AST classifier + deny.go:44-47 fail-closed + classify_bypass_test.go:6-97 + exec.go:1672-1685 wiring; docked: sandbox opt-in fail-open (sandbox.go:12-16), project-hooks trust hole (runtime/hooks.go:21) |
| token-economy | 9 | compaction.go:47-62,152-177,217-260 + compact.go:56-63,78-118,166-175 + anthropic.go:185-232 + cache_test.go; cost ledger per backend/purpose |
| orchestration | 7 | autonomy/policy.go:14-52 consequence-class grants + budgets; gateway scheduler + checkpoints; no queue-recovery, no loop breaker |
| interop | 8 | cmd/mcp_serve.go:16-28 memory-as-MCP-server to rivals; migrate.go:23-107 rival importers; PROTOCOL.md; run --protocol stream-json; no ACP/IDE/SDK |
| operability | 8 | checkpoint.go:1-36 + rewind.go + wiretrace.go:12-22 + doctor/session tooling |
| originality | 8 | eval.go:23-33 counterfactual A/B of the core claim; recall.go:1-15; loop.go:646-700 evidence-cited plan critic; autonomy consequence classes |
| durability | 5 | release.yml + hosted infra; shallow snapshot, no SECURITY.md, unpinned vaxis fork (forks/doc.go:8-12); evidence-limited |
| docs-dx | 7 | COMPACTION.md/HOOKS.md/PROTOCOL.md, honest sandbox+checkpoint docs; no SECURITY.md |

**Weighted total: 75.5 → band B.** Strongest dimension: token-economy (9) — the only subject so far pairing thrash-measured eviction with tested cache-breakpoint layout and a compaction back-off. Weakest: durability (5) — evidence-limited by the shallow snapshot plus concrete governance/supply-chain gaps.

Calibration: no rule (a) fork relation; no demotions; no boundary within 2 pts of 88/78/65/45 (75.5 sits 2.5 under A). Census test_loc corrected before scoring.
