# Boundary re-review: kolkrabbi (independent, 2026-09-30)

Provisional 77.0 B (1.0 under A floor). Contested axes: architecture 9-vs-8, verification
8-vs-7, safety 7-vs-6. Verdict: **77.0 B CONFIRMED; A floor NOT crossed.**

Closest anchor: **pi** - solo-vision, contract-first layering, exemplary safety honesty;
kolkrabbi's structural gating is codex-grade in mechanism but pi-grade in scale and backing.

Independence note: concept-dedup grep over `findings/*.jsonl` incidentally surfaced two
existing kolkrabbi records (kolkrabbi-5 sandbox-escape-proof, kolkrabbi-7
architecture-contract-test) before I read the prohibition boundary. All verdicts below were
formed from my own file evidence gathered first; boundary findings are limited to evidence and
claims NOT in those two records, and no provisional lane values were consulted.

## 1. Architecture 9 - CONFIRMED

Do the gates BLOCK? Yes.
- `make arch` (AST-enforced layering) and `make purity` run in the `guardrails` job of
  `.github/workflows/ci.yml:59-63` on every push/PR; repo-wide grep for
  `continue-on-error` across `.github/` returns ZERO - every named gate is red-or-green.
- The rules are data, not grep: `internal/arch/layers.go` encodes an L0-L6 layer ladder
  (`mayImport`, :52-61), an unlisted package FAILS the test (:68-72), the engine cannot import
  adapters (:47-50), and `thirdParty` is a per-package import allow-list.
- The ratchet is genuinely bidirectional: `knownViolations` fails both on unlisted violations
  AND on fixed-but-still-listed entries (`layers.go:9-13`, `arch_test.go:539-572`), so it cannot
  rot into exemption - and it is now EMPTY (`layers.go:242`): the debt is fully paid, which is
  the ceiling the ratchet was built for. Unused-allowance decay is also tested
  (`arch_test.go:596-616` - "a budget that pre-approves what nobody asked for is not a budget").
- Purity: engine touches no OS - no GOOS-suffixed files above the platform layer, `os/exec`
  and home-dir lookups have exactly one owner (`scripts/check-purity.sh`, run at ci.yml:62).

Zero-god-file claim: VERIFIED by measurement. Largest non-test Go file = 1,692 LOC
(`internal/engine/agent.go`); next: tui/model.go 1412, tui/controller.go 1012. Total 55,164
product LOC across 782 files, 67,548 test LOC in 442 files. The ERRATA >5k-LOC product-file
docking factor is not remotely in play - kolkrabbi is an order of magnitude under, better than
codex 9 (residual 3.1k files) and vastly better than pi 9 (6.8k per errata).

8-vs-9 substance per the errata ("loop/state model separated from every product surface"):
holds. All surfaces drive ONE engine - `engine.New` appears only in `cli/run.go:382` and
`slash.go:333`; REPL, TUI (`tui_repl.go`), saga (`saga_prompt.go:18`) and slash commands all
dispatch through `RunTurn` (`agent.go:1397`), no parallel re-implementations. Surfaces wire at
L6; the engine injects nothing concrete (L4-cannot-reach-L5 rule).

Honest caveats kept out of the score, recorded in findings: there is NO direct per-file LOC
gate (unlike ouroboros's 1600-line CI ratchet); smallness is maintained indirectly by the
layering/purity structure plus the budgets job (binary size ratchet, cold start, test-count
floor 3217, dep count - `scripts/check-budgets.sh:1-31`, "These FAIL, never warn"). Under the
errata's own terms (pi and codex hold 9 with 3-7k residual files; the gap is separation, not
absence of gates), kolkrabbi's separated single-loop core + blocking arch-as-data + paid-off
ratchet is at or above both anchors on this axis. **9 stands.** Docking to 8 would need a
defect, and I found none on this axis.

## 2. Verification 8 - CONFIRMED (case for 7 rejected; case for 9 rejected)

Against 7: this is not crush/thin territory. 442 test files / 67.5k test LOC vs 55k product;
`make test` runs every module (the nested-module trap is explicitly handled, ci.yml:36-38
comment); golangci-lint pinned; test-count FLOOR is a blocking budget number bumped per
release (check-budgets.sh:27-30 - old floor of 22 "could not trip", so they rebuilt it at 90 %
of 3,575); kernel escape tests run natively on both CI OSes (test matrix ubuntu+macos,
ci.yml:21) with three-way refusal discrimination. That is the codex/cline/pi 8 corpus shape.

Against 9 (ERRATA: 9+ needs evals-in-CI or fuzzing, with the workflow line that BLOCKS):
- Fuzzing: 6 `func Fuzz*` targets exist (`provider/stream_fuzz_test.go:26,85`,
  `redact/json_test.go:106`, `redact/scrub_test.go:248`, `tools/execute_fuzz_test.go:24`,
  `xid/xid_test.go:265`). Their inline seed corpora execute under `go test` in CI, but grep of
  `.github/workflows/` and `scripts/*.sh` for "fuzz" returns ZERO: no `-fuzz` discovery run, no
  committed corpus (`testdata/fuzz` absent), no regression-seed replay. Seed-only execution is
  seed-corpus EXISTENCE, not fuzzing enforcement - smelt's ledger entry (9) had discovery
  feeding CI-replayed corpora; kolkrabbi has neither.
- KolkBench: pre-registered 12 tasks + `verify.sh` exit-code scoring (`bench/README.md`), and
  a pilot DID run (2026-09-05, 3 harnesses x 3 tasks x 2 runs, 0/18, model too weak -
  `bench/results/2026-09-05-local-8b-pilot.md`) - but the README itself declares "No
  comparative run has happened" and no workflow references bench/. The pilot validated the
  MODEL, not the harness ranking.
- Live model in CI: `smoke.yml` is weekly/dispatch-only, never on push, and SUCCEEDS
  (notice) when the API-key secret is absent - honest opt-in, but by the gptme rule refinement
  it cannot gate anything.
No ledger entry. **8 stands** (identical to the anchors' 8-ceiling, and honestly instrumented
at that).

## 3. Safety 7 - CONFIRMED (6 rejected)

The 6 rung is "tested approval, NOTHING underneath." Kolkrabbi has something underneath, and
it is tested where it runs:
- Hardline floor checked FIRST and binding at every tier including full-auto:
  `permission.go:11-14` ("no tier removes the floor below. `--yolo` used to be that off
  switch"), `judgeWith` floor-before-rules-before-tier ordering (`permission.go:79-81`),
  tested at `permission_test.go:36` (all tiers), `:79 TestFullAutoStillHasAFloor`, credential
  reference matching catching the `cat ~/.ssh/id_ed25519` bypass (`permission.go:227-256`).
  Defaults are the most conservative tier: `DefaultPermission = PermissionAsk` (`permission.go:30`).
- Kernel jail: Landlock (`sandbox_linux.go`, 323 LOC) + Seatbelt (`sandbox_darwin.go`),
  escape tests demanding the platform's own refusal phrase run in the ubuntu+macos test matrix
  (pre-enforcer commits go red - sandbox_escape_test.go:13-22). Fail-closed at the boundary:
  Landlock ABI<4 network-deny is "refused rather than approximated" (`sandbox.go:14-19`);
  platforms without an enforcer refuse rather than run unconfined.
- Honesty is first-class: jail is opt-in, default off, labeled as the owner's decision
  (`config.go:53-56`, `settings.go:65`); the word-match blocklist is explicitly "a floor, not
  a perimeter; the jail and confirmations remain the primary control"
  (`docs/plan/13-tools-permissions-sandboxing.md:317`); SECURITY.md states what kolk touches.

Why not 8: the default posture is approval + word-match floor; the kernel layer binds only
when the user opts in (unlike codex, and unlike gemini-cli whose enforcement ships on). Why
not 6 like nanocoder: nanocoder's jail fails OPEN to plain `sh`; kolkrabbi's refuses, is
escape-tested against real kernel refusals on both CI OSes, and its floor survives full-auto
with tests. Hermes precedent (floors + fail-closed, tested, no always-on kernel layer = 7,
kept under 8 for zero shipped-by-default kernel enforcement) maps exactly. Tested-but-opt-in
here also includes a floor that is NOT opt-in. **7 stands.**

## Weighted recomputation (this review)

arch 9x15 + ver 8x15 + safety 7x10 + token 8x10 + orch 7x10 + interop 7x10 + oper 8x10 +
orig 7x10 + dur 6x5 + docs 9x5 = 770/10 = **77.0, band B.** Non-contested lanes held at
provisional after spot-verification (token: cache-aware user-turn injection `dirtytree.go:22`,
cache accounting `provider/client.go:78-82`; oper: `checkpoint.RewindLastTurn`
`checkpoint.go:161`, `RewindTask` `task_snapshot.go:92`, auto-resume `cli/resume.go:35`;
docs: 20 numbered design plans with a CI plan-vs-tick-marks drift gate, ci.yml:92).
No lane survives a raise or drop; the A floor is not crossed. B-ceiling plateau member,
consistent with the dispatched cluster note.
