# zot -- T1 review

- Subject: `/data/samples/agents/zot` -- `github.com/patriceckhart/zot`, Go, MIT (LICENSE:1).
- Tier: T1. Review is read-only static; nothing executed.

## Census sanity-check

- **test_loc wrong**: census says 47; actual **35,152 LOC across 235 colocated `*_test.go`**
  (`find . -name '*_test.go' | wc -l` = 235; `wc -l` sum). Same failure mode as the crush/cline
  anchor artifacts. All *.go = 105,026, so wc-based non-test is ~69,874 vs census 91,666 --
  census non_test is also overstated (~1.3x), T1 tier itself still holds either way.
- **Shallow clone** (`.git/shallow` exists, single commit HEAD `41873f6`). HEAD merge message is
  "Merge pull request #216 from m1k3s0/..." -- external PRs are being merged, so census
  `contributors: 1` / `commits: 1` are shallow artifacts, not facts. head_date 2026-09-27 is
  fresh; no activity claim in either direction.
- Provenance: census flag `original`; nothing reviewed contradicts it. Go codebase, pi-inspired
  shape (provider-agnostic loop + extensions + skills) but no evidence of copied source. Not a fork.

## Anchor question (one sentence)

Closest to **crush (69.0, B)**: a Go harness with a genuinely separated core loop and heavy
single-host-file heft, tested-but-not-kernel-backed approval, and race-tested 3-OS CI -- zot
sits a shade below crush on safety defaults and durability, a shade above on cache discipline
and distribution originality.

## Structure read

- `packages/core` (provider-neutral loop, session JSONL, compaction, confirm gate) imports only
  `packages/provider` -- no TUI/mode references (`packages/core/agent.go:12`). Cleanest boundary
  in the stack: `core.Agent` loop `runLoop` (`agent.go:446-555`), tool exec with panic recovery
  (`agent.go:899-909`), interrupt-hook ordering guard-before-approval so the user approves the
  effective args, not the model's draft (`packages/agent/cli.go:817-830`).
- `packages/provider` -- 11+ client impls (anthropic/openai/openai-codex/gemini/bedrock/llamacpp/...)
  behind one `Client.Stream` contract; catalog `catalog_builtin.go` (728 LOC).
- `packages/agent` -- CLI/extension/swarm/zotfile wiring; `packages/agent/modes/interactive.go`
  is **7,820 LOC** -- product-surface god file (per errata, a docking factor; loop itself stays
  in core, which is what separates this from the crush 7-rung).
- `packages/tui` -- hand-rolled TUI (view.go 2,833 LOC), no framework dep (AGENTS.md:10-12
  states it; go.mod confirms: no bubbletea/lipgloss).

## Per-dimension scores

### architecture: 8 (weight 15)
- Three-layer split: `core` loop knows only `provider` (agent.go:1-12); modes/extensions/swarm
  consume the loop via hook fields (`BeforeToolExecuteContext`, `BeforeTurn`,
  `BeforeAssistantMessage` -- agent.go:42-77).
- Message queue injected only at safe boundaries with explicit rationale
  (agent.go:153-170; runLoop drains at agent.go:446ff).
- Docked from 9: interactive.go 7,820 LOC (modes/interactive.go:417 (type Interactive) settings/compact/dialog
  policy all in one file) + tui/view.go 2,833. Better than cline's 8-rung docking? No -- cline
  8-rung was docked for 2.7k host files; zot's host file is 3x that. 8 holds because the
  loop/state model is separated from every product surface (pi errata criterion).

### verification: 7 (weight 15)
- 35k LOC property-named tests: `core/session_repair_test.go:14-106` (dangling tool_use repair:
  stub append, merge, no-op-on-valid, server-tool skip), `zotfile_test.go:260`
  (`TestUntarRejectsTraversalAndOversizedEntry`), `zotfile_test.go:134`
  (`TestLoadZotfileRejectsUnenforcedPermissions`), `confirm_lifecycle_test.go:5` (decision
  provenance sources).
- CI: go vet + gofmt gate + `go test -race` on ubuntu/macos/windows matrix
  (.github/workflows/ci.yml:19-46). Release gated on CI success (release.yml:38).
- Ceiling per errata: no in-CI model evals, no fuzzing, no coverage gate, no govulncheck --
  same 7-rung as crush (which additionally has govulncheck) and nanocoder (which has a
  coverage-drop gate).

### safety-enforcement: 5 (weight 10)
- Approval machinery exists and is tested, but **default is yolo**: confirmation only exists
  behind `--no-yolo` ("Defaults off (yolo mode): tools run without asking",
  `packages/agent/args.go:100-108`; gate constructed only if flag set, cli.go:807-810).
  Non-interactive + `--no-yolo` => deny-by-default with instructive reason (confirm.go:119)
  -- honest, but the binding-by-default case is the off path.
- Path jail: symlink-aware containment with `canonicalOrParent` (tools/sandbox.go:40-59) and
  tests (sandbox_test.go:10,60,182), but starts unlocked (sandbox.go:22-25) and is opt-in via
  /jail or `jail_by_default` config (config.go:73-74, jail_setting_test.go:9).
- Bash policy is a substring deny-list ("rm -rf /", "sudo ", "dd if=" ...) with honest
  self-assessment: "We cannot fully sandbox a shell" (sandbox.go:99,112-122). Trivially bypassable
  (e.g. `sud\0`-style variants, aliases, `python -c`); no OS-level enforcement anywhere
  (grep: no bwrap/seatbelt anywhere in packages/).
- Zotfile supply-chain side is genuinely defended: rejects unenforcible permission manifests
  (zotfile_test.go:134), bundled executable extensions (zotfile_test.go:214), tar traversal
  (zotfile_test.go:260), consent receipts (zotfile_test.go:396).
- Placement: nanocoder 5-rung ("real machinery + wrong default"); zot matches -- same wrong
  default, deny-list plus it, without nanocoder's fail-open-to-`sh` (its jail refuses rather
  than degrades). No OS jail keeps it under the 6-rung ("tested approval, nothing underneath"),
  since the by-default-tested layer is yolo.

### token-economy: 7 (weight 10)
- Auto-compact trigger measures **actual last-turn input tokens** vs model context window at a
  configurable threshold (modes/interactive.go:6502-6524), plus a pre-turn condense pass so the
  next outbound request stays under the limit before it is sent (interactive.go:6202-6220).
  Reactive (last turn's usage, not a projection of the pending one) -- below pi's 8-rung.
- Compaction: single tier, structured "context checkpoint" prompt preserving goals/decisions/
  next-steps verbatim-precision instructions (core/compact.go:196-233); keepTail with orphaned
  tool_result repair so Anthropic won't reject the tail (compact.go:127-133, 150-167);
  extension-activated tools survive compaction (compact.go:106-111); explicit compaction
  checkpoint appended to the session journal (core/session.go:711).
- Prompt-cache discipline crush/nanocoder lack: 4-breakpoint budget explicitly allocated on
  Anthropic with the OAuth identity line given its own breakpoint so midnight-date drift in the
  user prompt can't nuke the whole prefix (provider/anthropic.go:253-285); Bedrock
  `cachePoint` markers gated on catalog cache-write price (amazon_bedrock.go:276-280, 362-375);
  cache read/write tokens in the cost tracker (core/cost.go:34).
- Minus: compaction itself token-estimates at chars/4 (compact.go:35); no cache warming, no
  tiered strategies (codex 9-rung absent).

### orchestration: 7 (weight 10)
- Swarm supervisor over headless `zot --swarm-agent` subprocesses reusing its own loop instead
  of reimplementing one (swarm/swarm.go:1-26); durable identity `meta.json` + events.jsonl per
  agent, `Reload()` re-registers orphans as `StatusDetached` on next launch so the dashboard can
  view/resume/remove them (swarm/persist.go:4-16, 33-49; swarm.go:48); inter-agent inbox
  (swarm/inbox.go) with tests incl. e2e (runner_e2e_test.go, persist_test.go 913 LOC).
- Honest limits documented in the package comment: no worktrees, no isolation, "use real git
  yourself" (swarm.go:9-13).
- Queued-message pump at safe boundaries doubles as lightweight steering (agent.go:153-170).
- Absent: budgets (grep "budget" in swarm: none), loop detection, cron. Below cline 8-rung.

### interop: 6 (weight 10)
- Own headless contract: rpc mode with prompt/abort/compact/set_model/get_state method set
  (rpc.go:225-331), docs/rpc.md; print/json modes; Go SDK package (packages/agent/sdk/sdk.go);
  RPC examples in Go/Node/Python/shell (examples/rpc/).
- Extensions = subprocess JSON-RPC in any language (extproto/extproto.go, manager.go spawn/read/
  reload with grace periods), opt-in install, none by default (README:26).
- **No MCP client/server, no ACP**; an `mcp-bridge` exists only as an example extension
  (examples/extensions/mcp-bridge). No published SDK artifact. Same by-philosophy 6-rung as pi,
  telegram bot (modes/telegram/, botcmd.go) adds a surface pi lacks.

### operability: 7 (weight 10)
- JSONL session journal with per-message durable append hooks (`OnMessageAppended`,
  agent.go:94-103), usage rows persisted for crash-accurate cost (agent.go:105-110), compaction
  checkpoints (session.go:711); load-time repair of dangling tool_use pairs (session.go:313,
  session_repair_test.go).
- Resume: LatestSession/OpenSession (session.go:379, 244); session tree dialog with inline fork
  points and branch checkout (modes/session_tree_dialog.go:13-15, session_ops_dialog.go:21);
  empty-session pruning (session.go:611).
- Per-tool panic recovery keeps the loop alive (agent.go:899-909); swarm crash-reload above.
- No git-checkpoint/revert (crush 8-rung recover-middleware breadth unverified in TUI); no
  documented diagnostics command surface beyond ext Diagnostics() (manager.go:1164).

### originality: 7 (weight 10)
- **Zotfiles** -- portable packaged agents from a local dir, `.zot` archive, or a temporary
  public GitHub download (zotfile.go:1-24 imports incl. sha256/tar/zstd; tests: GitHub shorthand
  resolution zotfile_test.go:176, temporary archive download :303, consent key handling :21-96,
  denied-scope summary :375). The permission-manifest-plus-consent-receipt packaging of a
  *downloadable agent* is a shape no anchor has; enforcement is real (see safety) -- mechanism,
  not marketing.
- Hand-rolled TUI + JSON theme system incl. extension-supplied themes (docs/themes.md,
  manager.go:501) with zero TUI framework deps.
- First-class telegram bot mode (modes/bot/, botspec.go).
- Loop, skills, extensions are otherwise recognizable pi/cline lineage -- caps at 7-rung
  (crush/nanocoder level).

### durability: 4 (weight 5)
- Single-author project (MIT, author Patric Eckhart); no SECURITY.md (absent; grep of
  README/CONTRIBUTING finds none); no governance doc; 2 workflows total.
- Shallow clone means census contributors=1 understates (external PR #216 merged at HEAD), but
  institutional backing is still thin. nanocoder 4-rung ("thin, young churn"); pi's 7 needs a
  visible community, not yet evidenced here.

### docs-dx: 7 (weight 5)
- Six topic docs matching implemented behavior (docs/: extensions, providers, rpc, skills,
  themes, zotfiles); AGENTS.md working agreement with ownership map (AGENTS.md:1-30);
  install.sh verifies SHA-256 against release checksums.txt (README:41-49); flag reference
  table in --help (args.go:457).
- Below pi's 9 (30+ docs incl. security/session-format): no SECURITY.md, no architecture doc;
  docs/superpowers/ is a stray two-file process-plans directory, not user docs.

## Strongest / weakest

- **Strongest dimension: architecture (8)** -- core loop is fully provider- and surface-neutral
  with a documented hook contract; among reviewed Go subjects this is the cleanest loop/host
  split (better than crush's fused agent.go).
- **Weakest dimension: durability (4)** -- solo maintainer, no SECURITY.md, governance absent;
  verification and safety machinery are strong for the project's age but inherit single-point
  risk.

## Calibration notes

- Weighted 67.0 => band B; exactly 2.0 above the 65 boundary -- flagged per dispatcher rule.
- Census corrections: test_loc 47 -> 35,152 (235 files); non_test_loc overstated (~91.7k vs
  ~69.9k wc-based); contributors/commits are shallow-clone artifacts (external PR merges
  present). No tier change: still mid-T1.
- No calibration rule (a) applicable (not a fork). No license restriction (MIT).
- Shallow clone: no activity claims made from HEAD.
