# Boundary re-review: zot (T1, Go, solo, MIT) - provisional 67.0 B (+2.0 over floor)

Independent pass, 2026-09-30. Did NOT read subjects/zot.md, scores/zot.json, findings/zot.jsonl.
Anchors + ERRATA + calibration-notes precedents (octomind shipped-default doctrine, amazon-q
broken-as-configured CI ledger, verification-9 ledger, mocode non-kernel safety ladder) applied.

**Anchor sentence:** closest anchor is **crush (69.0 B)** - solo-ish Go TUI agent with a clean
core loop, subagent plane, and 3-OS race CI; zot sits just below crush because crush's approval
layer binds by default while zot's does not, and crush carries govulncheck + funded durability.

## Q1. Safety rung: 6 or 5? -> **5**

Shipped-default posture (octomind/amazon-q doctrine: "shipped default posture, full stop"):

- **Yolo by default.** `NoYolo` defaults off; "Defaults off (yolo mode): tools run without
  asking" (args.go:95-106). Opt-in only via `--no-yolo` (args.go:236). Worse, the opt-out is
  silently voided in headless modes: "No effect in -p / --json / rpc modes ... tools still run
  freely" (args.go:102-105) - a user who passes `--no-yolo` and scripts with `-p` gets zero
  approval, only a stderr warning.
- **Jail off by default.** `jail_by_default` is "Off by default; nil/missing means disabled"
  (config.go:71-74); the sandbox only locks if the user explicitly set it (build.go:615-617),
  and `NewSandbox` "starts unlocked" (sandbox.go:23-24). Both `CheckPath` and `CheckCommand`
  are hard no-ops when unlocked (sandbox.go:43, 104).
- **Net shipped default: nothing binds.** No approval, no path jail, no bash gate (the deny-list
  lives inside `CheckCommand`, which returns nil unlocked). No OS layer anywhere (no seatbelt/
  bwrap/landlock hits).
- **The bash gate, when enabled, is a substring deny-list** bypassable by construction:
  `strings.Contains(lower, banned)` over literals like `"rm -rf /"`, `"sudo "` (sandbox.go:111-122).
  Whitespace variants (`rm  -rf  /`), shell quoting (`'rm' -rf /`), `$IFS`, and env expansion all
  sail through; only the first `;`/`&&` segment's `cd` is checked (sandbox.go:134-136). The code
  itself is honest: "This is a speed bump for the model, not a security boundary"
  (sandbox.go:131) - honesty credits the deception-penalty axis (hax doctrine) but never buys
  the rung.

Why 5 and not 4 or 6:
- Not 6: the crush/pi/cline 6 rung is "approval **binds** in-loop and is tested." zot's gate is
  real and well-tested (confirm_test.go:39-182 covers allow-once, remember-tool, remember-all,
  refuse-with-reason) - but it does not bind on the default path. That is exactly the nanocoder/
  octomind precedent: "real machinery + wrong default = below the 6 rung," octomind now applied
  twice (octomind honest variant, forge hidden variant). mocode's non-kernel ladder names rung 5
  for "deny-list" shapes; zot is deny-list + shipped-disabled, squarely rung 5.
- Not 4: no phantom UI, no runtime false-safety indicator (claurst), no hidden-off engine
  (forge). Yolo is named in the flag itself, the jail default is documented, errors say
  "use /unjail". Honest-weak variant, same pole as octomind's 5.

Bonus counter-credit noted, not scored: zotfile consent is the corpus's best-shaped defense
against clone-and-run hooks (see zot-b4) - digest-keyed, fail-closed ("refusing to run without
interactive consent", zotfile.go:827). But it protects only `zot run` zotfile agents, not the
default interactive session.

## Q2. Architecture 8 claim with a 7,820-LOC interactive.go -> **7, docked**

The loop/state model IS separated: `packages/core` holds runLoop (core/agent.go:446), the
confirm gate (core/confirm.go), compaction (core/compact.go), session store with fork tree
(`Parent`/`ForkPoint`, core/session.go:52-62, 479-500), and modes/ drives it as a consumer.
That is the right shape and blocks a lower score.

But the ERRATA is explicit: a product/TUI-mode file >5k LOC is "a docking factor on both sides."
`modes/interactive.go` is **7,820 LOC** (192 funcs; `handleKey` alone spans :2212-~3000),
plus tui/view.go 2,833. The 8 rung (cline) is docked already at 2.7k-LOC runtime hosts; the
pi errata made 7.8k-class interactive files THE 9-vs-8 question. An 8 claim with a 7.8k TUI
god-file is the wrong rung by the study's own frozen errata: **8 -> 7** (-1.5 weighted).
Crush rung 7 ("sound design, big fused files") describes zot's position precisely; zot's files
fuse view/input/dialog/settings rather than loop+policy, which is why it does not sink below 7.

## Q3. Verification: 35k test LOC - what actually blocks in CI?

CI is small but real and **blocking** (no continue-on-error): on every push and PR to main, a
3-OS matrix (ubuntu/macos/windows) runs `go vet ./...`, a gofmt diff gate, and
`go test -race ./...` (ci.yml:3-13, 17-48). Nothing else: no govulncheck, no coverage gate,
no evals, no fuzzing, no nightly.

The corpus behind that command is genuinely property-asserting, not existence-checking:
235 files / 35,152 LOC with named-issue regressions (sandbox_test.go:58 "regression for issue
#94"; sandbox.go:63-70 DisplayPath fix cites issue #39), confirmation-gate semantics
(confirm_test.go), compaction event pairing (lifecycle_test.go:130), intercept/policy semantics
(intercept_test.go:28-125), swarm persistence/reload (swarm/persist_test.go 913 LOC,
runner_e2e_test.go), session prune/repair/branch tests. Tests run on all three OSes with -race,
which is the crush-7 shape (crush: 61.7k LOC + -race 3-OS + govulncheck). zot = same matrix,
smaller corpus, no vulncheck, TUI surface tested only at handler-helper level -> **7**, matching
provisional; the 8 ceiling (errata) is not contested - no evals/fuzzing exists.

## Full lane recount (vs provisional 67.0)

| dim | score | weight | note |
|---|---|---|---|
| architecture | 7 | 15 | docked from 8 per errata (interactive.go 7,820) |
| verification | 7 | 15 | blocking -race 3-OS + property tests; crush rung |
| safety-enforcement | 5 | 10 | docked from 6: nothing binds at shipped default; substring deny-list |
| token-economy | 7 | 10 | threshold compaction + thoughtful 4-breakpoint anthropic cache budget (anthropic.go:253-267); no cache-hit validation, len/4 token estimate (compact.go:31) |
| orchestration | 7 | 10 | durable swarm (meta.json + Reload re-registration, persist.go:1-20) tested; no loop-detection, no budgets |
| interop | 6 | 10 | token-auth RPC contract (rpc.go:171-198) + Go SDK; MCP only as example extension (examples/extensions/mcp-bridge); no ACP/IDE |
| operability | 7 | 10 | session fork tree, resume/continue, rename/delete, cost tracker; no crash-recovery posture doc |
| originality | 7 | 10 | digest-keyed consent receipts; jail-aware DisplayPath escape-nudge (issue #39); swarm inbox sockets |
| durability | 5 | 5 | solo, active (HEAD 2026-09-27, external PR #216), MIT full text, goreleaser; shallow clone noted; no SECURITY.md |
| docs-dx | 7 | 5 | 105KB README + six topic docs + install.sh/ps1; rpc docs mirror SDK; no SECURITY.md |

**Weighted total: 66.0 -> B.** Safety 6->5 (-1.0) and architecture 8->7 (-1.5) against a
provisional that (by the dispatch's own account) sat at 67.0 with those rungs contested; other
lanes land where the anchors put them.

## Band verdict

**B floor HELD** at 66.0 (1.0 margin). The two contested rungs both dock, but the blocking
-race CI, the separated core, and the swarm/interop/operability substance keep the total above
65 - distinct from amazon-q, whose B->C move came from four lanes being over-credited, not from
the dispatched hinge. No calibration rule (a)/(b) applies: original provenance, active, MIT.
Shallow clone noted (.git/shallow present); activity judged from HEAD + merged external PR only,
no inactivity claim made.
