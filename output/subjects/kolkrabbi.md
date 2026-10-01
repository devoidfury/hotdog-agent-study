# kolkrabbi — T2 deep review

Subject: github.com/onembyte/kolkrabbi (Go, Apache-2.0, binary `kolk`)
Census row: non_test_loc 141,596 / test_loc 1,256 / contributors 1 / head 2026-09-09 / shallow / commits 1 / tier T2.

## Census sanity (corrections)

- **test_loc is wrong: 1,256 -> 67,548.** 442 colocated `*_test.go` files, 67,548 lines, ~2,147 `func Test*`
  (find/wc over the tree). Colocation confirmed: tests sit beside sources in `internal/engine`, `internal/shell`,
  `protocol/`, etc. The 1,256 figure looks like only a subset of `protocol/` tests was counted.
- **non_test_loc is inflated: 141,596 -> ~55,164** actual non-test Go (122,712 total `.go` minus 67,548 test).
  The census evidently swallowed docs (1.7 MB), site/, spec/ markdown. By the LOC rule (5k-100k) this is a T1-sized
  tree; dispatched T2, reviewed at T2 depth regardless -- the depth was warranted, the size was not.
- **Shallow clone (1 visible commit)**: no activity or dead claims made; head_date 2026-09-09 is fresh (study date
  2026-09-29), so no remote verification required by the 10-month rule.
- Provenance: original per census and nothing contradicts it -- distinct module path, distinct CLI vocabulary, no
  upstream sync machinery. Identity hygiene: no near-namesake in the corpus.

## Anchor question

**Closer to pi than to codex**: a solo-vision, provider-agnostic agent with the loop cleanly separated from every
surface and exemplary honesty about what its protections do not cover -- but where pi left sandboxes and policy to
extensions, kolkrabbi ships a kernel-enforced (opt-in) sandbox and CI-enforced architecture gates, while trailing pi
on cache discipline and breadth of interop surfaces.

## Mandatory line-level reads (what I actually read)

- Core loop: `internal/engine/agent.go:1397-1693` (`RunTurn` dispatch, paused check, dirty-tree preamble as user-turn
  injection to protect the cached system prompt `:1420-1424`, mode split), `runLoop` `:1522-1692` (stream -> tool
  calls -> guard -> doom-loop check *before* execution `:1622-1634`, empty-completion recovery capped at 2,
  effort-governed `MaxRoundsFor` `:99-127`, overflow recover-once `:1571-1578`, one terminal bus event per turn
  `:1481-1510`).
- Compaction: `internal/engine/compact.go` full file (three escalating stages tool-results -> tool-calls -> summary,
  stop at first that fits `:66-105`; tool results emptied not removed to keep call/result pairing valid `:37-39`;
  `keepRecentTurns=2` `:194`; compact-to-half-window `:199-202`; turn-boundary-only with the orphaning rationale
  `:226-231`; in-memory + on-disk archive and `RestoreCompaction` undo `:262-293,388-401`) and
  `internal/engine/context.go` (trigger on the provider's measured prompt-token count, char estimate only as the
  pre-first-turn floor `:16-46`; unknown window never compacts `:48-56`).
- Permission/sandbox: `internal/engine/permission.go` full file (tier model `:14-30`; ordering floor -> user rules ->
  tier `:73-118`; hardline credential/system-dir/command floor `:126-232` with the self-aware "short list, not a
  perimeter" comment `:150-154`; credential bypass catch for shell spellings `:212-232`), `internal/tools/confine.go`
  (symlink-resolved path jail incl. deepest-existing-ancestor resolution `:14-63`), `internal/shell/sandbox.go` +
  `sandbox_darwin.go` + `sandbox_ladder*/escape tests` (one policy struct, seatbelt SBPL inline argv
  `sandbox_darwin.go:31-46`, Landlock on Linux, network-deny gated on ABI 4 with refusal-not-approximation
  `sandbox.go:12-20`, fail-closed `Refusal()` `:70-75`, `sandbox_other.go` refuses everything on unenforced platforms),
  subagent guard converting Ask to Deny `agent.go:1000-1014`.

## Scores (10 dimensions)

### architecture — 9 (weight 15)
Structural rules are *data enforced by CI*: `internal/arch/layers.go:1-32` defines an 8-layer dependency ladder and
the ratcheting `knownViolations` list that fails when a violation is added **or** when a listed one is fixed but not
removed; CI runs it (`Makefile:58-60` `arch`, `purity` -- "the engine touches no OS" --
`.github/workflows/ci.yml` guardrails job). No god files anywhere: largest non-test file is
`internal/engine/agent.go` at 1,692 LOC; TUI (`tui/model.go` 1,412), CLI, serve, daemon are all thin L6 surfaces
consuming the engine through a bus (`internal/bus/bus.go`) and port interfaces (`saga_ports.go`, `ChatBackend`
`agent.go:393-396`). Loop/state fully separated from every product surface, which is exactly the 9-rung criterion in
the errata -- better on file size than either 9-anchor. Held at 9, not higher: single binary, one process model, the
`Options` struct at ~150 lines of optional seams is a lot of knobs on one type.

### verification — 8 (weight 15)
67,548 test LOC / 442 files / ~2,147 test funcs, and the tests *prove properties*: doom-loop spec read as
behavior-named cases (`doomloop_test.go:16-154`: "a changing result is progress even when the command is the same",
"reserialization is not a different call"); sandbox escape tests assert the refusal came **from the kernel**, not
from kolk declining or the pre-exec guard (`sandbox_escape_test.go:61-75` `refusedByTheOS` distinguishes exit -1 /
exit 125 / real kernel phrase); `mockagent` fake vendor CLIs prove the SIGINT->SIGTERM->SIGKILL spawn ladder rung by
rung (`internal/mockagent/mockagent.go:1-40`); fuzz targets exist for the SSE reader and tool dispatch with
must-not-happen invariants (`provider/stream_fuzz_test.go:9-46`, `tools/execute_fuzz_test.go:9-40`). CI: test matrix
(ubuntu+macos), golangci-lint pinned by SHA, plus ~14 named guard-rail gates (`ci.yml` guardrails job) and a budgets
job (binary size, cold start, test-count floor) that "fails, never warn". Ceiling per errata: fuzz targets run only
seed corpora in CI (no coverage-guided phase in any workflow) and the KolkBench harness is pre-registered with **no
comparative run** (`bench/methodology.md:3`); the weekly live smoke (`smoke.yml:1-50`) is a drift canary, not evals.
No 9 without a workflow that runs evals or real fuzzing.

### safety-enforcement — 7 (weight 10)
Above the 6-rung ("tested approval, nothing underneath") because there *is* something underneath and it is kernel-
tested: default tier is `ask` (`permission.go:28-30`), a hardline floor binds every tier including `full-auto`
(`:77-80`), path jail resolves symlinks through the deepest existing ancestor (`confine.go:49-63`), seatbelt (macOS)
and Landlock (Linux, network-deny needs 6.7+ and *refuses* rather than approximates, `sandbox.go:14-19`) with the
escape-proof above; unsupported platforms refuse to run sandboxed commands at all (`sandbox_other.go:4-10`);
subagents cannot ask, so anything ask-scoped is denied with instructions for how to widen intentionally
(`agent.go:1000-1014`); kept rules are session-scoped and visible in `/permissions` by design (`agent.go:1020-1041`).
Not 8: the OS sandbox is **opt-in** (`/sandbox on`; owner decision, honestly documented
`docs/plan/13-tools-permissions-sandboxing.md:106-108`), Windows has no enforcer, and the command floor is
word-matching -- trivially bypassable by indirection, which the code itself says (`permission.go:145-149` "not an
attempt at a perimeter"). Real machinery, safe default at the tier, weaker default at the jail.

### token-economy — 7 (weight 10)
Three-tier escalating compaction stopping at the first stage that fits (`compact.go:13-18,66-105`) is the codex
shape; the trigger uses the provider's **measured** prompt-token count and refuses to compact on an unknown window
(`context.go:16-56`) -- measuring what the model actually received, like pi's projected trigger, and arguably stricter
(no compaction on a guess at all). Overflow refusal triggers one compact-and-retry (`compact.go:356-371`), compaction
is announced and undoable with an on-disk archive (`:233-260`), and cheap work (summaries, titles) routes through a
"fast lane" that is cache- and cost-aware (`fastlane.go:27`). Short of pi's 8: no cache-write discipline at all --
cache-read tokens are recorded from usage metadata (`provider/client.go:487`) but there is no breakpoint control or
warming, and the dirty-tree preamble-in-user-turn trick (`agent.go:1420-1424`) is the only cache-preservation move.

### orchestration — 8 (weight 10)
Agent mode: planner decomposes into dependency-linked tasks, per-task subagent processes each with its own vendor
backend and zero context cost to the parent (`orchestrator.go:45-70`), width governed by effort (1..8,
`:26-43`), per-task git-worktree isolation for parallel writers (`Isolator`, `shell/worktree_isolator.go`, plan 36),
run-level cost ceiling `MaxRunCostUSD` that stops an orchestrated run (`agent.go:236-240`), saga posture with
chaptered progress persisted atomically as `SAGA.md` (`saga_persistence.go:14-30`) and its own doom threshold, and
the standout: **subscription continuity** -- a limit pauses the session durably, a resume monitor confirms the lift
with a token-free probe, and auto mode walks an equivalence chain mid-turn (`agent.go:1462-1467`, `resume.go:12-49`,
plan 35). Crash semantics of the monitor are unproven against kill -9 in what I can see statically; no external job
queue. That plus the doom-loop breaker puts it above crush/pi's 7.

### interop — 7 (weight 10)
MCP client with namespaced tools routed through the same permission rules (`agent.go:302-305`, `internal/mcp/`); a
language-neutral wire contract with a committed OpenAPI 3.1 spec and conformance/inventory/catalog tests
(`spec/kolk.openapi.yaml`, `protocol/openapi_test.go:76,222`, `protocol/contract_inventory_test.go`, `spec.yml` CI
workflow gating changes); headless `kolk serve` over HTTP/SSE plus stdio (`serve/stdio.go`, `sse.go`) with device
pairing and token auth (`serve/auth.go`, `pair.go`); `kolk-mock` ships as a test-binary provider. No ACP, no IDE
surface, no published SDK, no MCP server export. Same rung as crush/nanocoder with different strengths.

### operability — 8 (weight 10)
Sessions with title management and `kolk sessions` (rename marks user-owned titles: `compact.go:296-330`), turn-level
checkpoint + `Rewind` of file changes (`agent.go:1687-1692`, shadow-git per plan 32), compaction undo, pause/resume
with `/resume` policy, coalesced transcript saves with named durable boundaries that never cross a pre-write flush
(`agent.go:1086-1096` `preWrite`, `save.go` boundary table), dangling-tool-call repair on resume (`agent.go:799-833`),
`cmd_doctor` diagnostics, secret scrubbing on every error that crosses the wire (`secret.Scrub` at
`agent.go:1500`, `internal/redact` fuzz-tested for idempotence + valid UTF-8), budgets on cold start and binary size.
No crash-recovery posture doc as explicit as crush's recover-middleware-everywhere, hence 8 not 9.

### originality — 8 (weight 10)
Verified in code, not marketing: (1) multi-subscription continuity across vendor CLIs + gateway + local endpoints --
pause on limit, token-free lift probe, chain-walk to an equivalent model -- unique in the corpus
(`continuity/pause.go`, `resume.go`, `cooldowns.go`, plan 35); (2) the effort dial as a *routing* dial: each level
selects model tier, tool-round budget, shell timeout, and orchestration width simultaneously (`agent.go:99-148`,
`orchestrator.go:26-43`); (3) 100% local model-rating dashboard fed by per-call stats (`dash/`, `recordAtEffort`
`agent.go:839-875`); (4) KolkBench: a pre-registered, published methodology for measuring **the harness with the
model held constant** (`bench/methodology.md:5-20`); (5) arch-as-data ratchet with the fail-on-shrink rule
(`arch/layers.go:8-12`). Two dependencies total (`go.mod`) with hand-rolled PTY/TUI is a distinctive constraint
choice. Not 9: individually several ideas echo anchors (compaction tiering from codex, purity from pi), the combo is
what's novel.

### durability — 5 (weight 5)
Bus factor 1 (census contributors: 1; shallow clone hides the rest, and I make no claim on history depth). Against
the 4-rung (nanocoder): kolkrabbi has a SECURITY.md with a real disclosure path and machine-touches inventory, cosign
-signed releases (`release.yml:41-52`), SHA-pinned actions *with a CI check that they stay pinned*
(`ci.yml` workflow-pin-check), an explicit non-goals doc (plan 23), and head_date 20 days before study date. Young,
solo, institutionally unsupported -- but the engineering-governance signals are well above the 4 rung. 5.

### docs-dx — 8 (weight 5)
37 numbered design-plan docs that track *shipped state with versions and dates* ("shipped V34.1e, 2026-09-05",
`docs/plan/13...:107-108`), plan-check CI gate asserting the tick marks match documents (`ci.yml` plan-check),
SECURITY.md, CONTRIBUTING, install one-liner, docs site. Docs describe behavior, not aspiration -- the escape-test
section admits the marketing pages were *correct* about having no sandbox until the tests went green
(`docs/plan/13...:153`). Slightly under pi's 9: no equivalent of pi's how-pi-works/session-format reference set.

## Totals

| dim | score | weighted |
|---|---|---|
| architecture | 9 | 13.5 |
| verification | 8 | 12.0 |
| safety-enforcement | 7 | 7.0 |
| token-economy | 7 | 7.0 |
| orchestration | 8 | 8.0 |
| interop | 7 | 7.0 |
| operability | 8 | 8.0 |
| originality | 8 | 8.0 |
| durability | 5 | 2.5 |
| docs-dx | 8 | 4.0 |
| **total** | | **77.0 (B)** |

Strongest dimension: architecture (9). Weakest dimension: durability (5).

**Boundary note: 77.0 is within 2 of the 78 (A) boundary.** The two swings are safety 7->8 (if one counts
kernel-tested enforcement as defended despite opt-in) and token-economy 7->8 (measured trigger vs missing cache
discipline). I held both at 7 deliberately: the sandbox default and the absence of *any* cache-write discipline are
each a real gap, not a judgment call.

## Provenance / calibration

- Provenance: original, solo, Apache-2.0 (file present, LICENSE is the standard text). No fork delta to state.
- Calibration rules touched: none (not a fork; no archived claim). No demotions.
- License discipline: Apache-2.0 permissive; findings describe concepts as usual for the corpus.
