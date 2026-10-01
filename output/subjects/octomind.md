# octomind — T2 deep review

Rust, single crate (v0.55.0), Muvon Un Limited, Apache-2.0. Census says T2: accepted
(see census correction; real product LOC ≈111k, tests ≈105k).

**Anchor question:** octomind is closer to **pi** than to codex: a solo-vision,
systematically-designed runtime whose token-economy and cache discipline exceed the
anchor rungs, with safety machinery that exists but is opt-in, and thin institutional
durability -- but it is pi-plus in breadth: octomind adds MCP, ACP, a workflow engine,
and an out-of-band supervisor plane that pi deliberately leaves to extensions.

## Census sanity

- `test_loc: 3976` is wrong the same way the anchor-era artifacts were: inline
  `src/**/*_tests.rs` files (276 files, 101,385 LOC) were counted as non-test. Real
  tests: 101,385 inline + 4,169 in `tests/` = ~105.5k across 3,943 `#[test]`/`#[tokio::test]`
  fns. Real non-test LOC ≈111k (216,394 total .rs). Tier T2 still correct.
- Shallow clone (`.git/shallow` present, HEAD `fceb8cb chore(release): 0.55.0`,
  2026-09-26). No activity claims made; CHANGELOG shows an incremental release cadence
  through 0.55.0.

## Provenance

Original (manifest `octomind`, repo muvon/octomind, no fork flag). The provider/LLM/
evaluation layer is a sibling crate `octolib = 0.40.0` (Cargo.toml:44) also owned by
Muvon -- provider behavior and the fake-provider plumbing partly live outside this
repo, so that layer was audited only through its call sites. Not a sync-fork;
calibration rule (a) N/A. Identity hygiene: no name collision in this corpus.

## Core loop (line-level read)

- Interactive loop: `src/session/chat/session/main_loop.rs:204` (`run_interactive_session`),
  turn dispatch through `api_executor.rs:460-560` -- cost tracked immediately per
  exchange (`:467`), response processing shares one `process_response`
  (`src/session/chat/response.rs:439`) across all five surfaces via an `OutputSink`
  abstraction (`response.rs:37-52`), so interactive / non-interactive / jsonl / WS /
  ACP do not fork the loop. Good state model: `ChatSession` + RAII
  `SessionRuntimeGuards` (`main_loop.rs:42-48`), per-session listener registries.
- Ctrl+C semantics are explicit: `interrupted_call_truncation` (`main_loop.rs:118-132`)
  distinguishes first-call (truncate user msg) from multi-turn (preserve, tool msgs
  make state valid). Non-interactive keeps the session alive on an inbox with batched
  drain (`main_loop.rs:1857-1920`) so piled-up scheduled/background results cost one
  call between them, not one turn each.
- Tool admission: `admit_tool_batch` (`src/session/chat/response/tool_execution.rs:291`)
  runs guardrails sequentially over the ordered batch before any spawn; "Blocked tools
  are NEVER spawned; they return an immediate error result"
  (`tool_execution.rs:366-368`). Batch order feeds `+/-` history conditions
  (`tool_execution.rs:364-366`).
- Largest files: `display.rs` 4025 (product surface), `attention/mod.rs` 2112,
  `main_loop.rs` 2183, `gate.rs` 2003 -- no 5k+ product-mode files, no UI-coupled loop
  (contrast the pi errata rung). Loop separated from all surfaces -> architecture 8,
  not 9 (9 needs the codex-grade crate decomposition; this is one crate, though with
  clean module boundaries and pervasive test-seam docs).

## Compaction / token economy (line-level read)

- `conversation_compression/` is an adaptive controller, not a threshold: one fire
  line + physical hard ceiling entered early ("A cooldown may delay a soft fold, but
  it must never permit an over-window API request", `mod.rs:88-91`), depth computed
  per cycle from measured growth rate (`decision.rs` header, `adaptive_fire_line`,
  `autonomous_runway`, `measured_growth_rate`).
- **Fold economics**: `FoldEconomics` (`decision.rs:31-70`) prices a fold in token
  ratios relative to one uncached input token -- cache-read, folder input/output,
  cache-write -- with conservative stand-ins when a provider publishes no pricing;
  a fold behind the fire line only fires if amortized by predicted remaining turns.
  This is the most rigorous compaction-gate economics found in the corpus.
- **PACT attention controller** (`conversation_compression/attention/mod.rs:15-19,
  55-59`): deterministic evidence selection around the generative fold -- packet
  kinds, provenance enum (RealUser/ToolObserved/ValidatedSummary...), lanes
  (KeepExact/Summarize/ArchiveReference), SHA-256 packet digests. Runtime owns pins,
  model only folds.
- **Cache keepalive** (`src/session/cache_keepalive.rs:15-27, 198`): idle-time
  `max_tokens=1` ping against a frozen conversation snapshot to reset provider cache
  TTL; provider-policy gated (Anthropic only today -- "pinging them would just burn
  money"); ping costs harvested and folded exactly-once into session cost
  (`main_loop.rs:76-110`). Beyond pi's warmer: it extends TTL during idle, and the
  cost of warming is accounted, not invisible. Off by default (`default.toml:64`).
- **Condense** (`src/supervisor/condense.rs:15-33`): oversized tool outputs narrowed
  by ONE cheap-model call selecting LINE RANGES over a numbered copy, reconstructed
  verbatim -- the model never retypes content; full original spilled losslessly to a
  session file first; per-result fail-open ("no spill -> no condensation").
- Per-turn cost everywhere incl. `external_spend.rs:15-21`: subagent/layer/supervisor
  spend banked per session and folded exactly-once "including across the monotonic-max
  merge that persistence applies on resume". Workflow runs carry `max_cost` USD
  ceilings (`config-templates/workflow-graph.toml:13`).

## Safety / permissions (line-level read)

- **Guardrails DSL** (`src/config/guardrails.rs:15-36`): project-local
  `.agents/guardrails.toml` with four phases -- `[[pipe]]` pre-model input transform,
  `[[guard]]` pre-call deny, `[[hook]]` post-result script, `[[validator]]`
  end-of-turn script. Guard DSL goes beyond regex denylists: `has`/`when =
  ["-filesystem-read"]` temporal conditions evaluated against the session's
  capability call log (`src/session/guardrails.rs:15-60`, per-validator cursors).
  Denied calls return as `[guardrail]`-prefixed errors and never spawn.
  Tested (`src/session/guardrails_tests.rs:58` "authorizer_denied_calls_never_enter_
  history_and_preview_is_read_only").
- **Authorizer** (`src/supervisor/authorizer.rs:15-20`): bounded tool-free LLM batch
  admission on user-intent ground truth -- persisted `AuthorizationState` (user
  instructions, immutable parent boundary for delegated sessions) survives
  compaction/resume; only exact grounded denials memoized; policy/memory/tool-evidence
  changes invalidate.
- **Kernel jail**: Landlock (Linux) / seatbelt (macOS) applied to self before any
  child spawn, inherited by shells/MCP/subagents (`src/sandbox/mod.rs:15-22`,
  `main.rs:249-259`). Write-only isolation: read-only on `/`, RW on cwd + XDG data
  home; `~/.ssh` writable-protected but readable, and the code says so
  (`sandbox/linux.rs:20-27` "True read isolation on Linux requires
  namespaces/seccomp -- out of scope here"). Opt-in (`sandbox = false`,
  `default.toml:33`), fails open on unsupported kernels with a logged warning
  (`linux.rs:73-79`), no-op off-platform (`mod.rs:36-40`). No network egress control.
- **Supervisor gate** (`src/supervisor/gate.rs:15-19, 45-75, 660`): on self-reported
  `done`, an independent LLM verifier judges END STATE against evidence blocks
  (recorded actions, ground-truth diff, read-back rounds); narrative explicitly
  distrusted; prohibitions checked as requirements; bounded re-runs (`MAX_ITERATIONS = 2`);
  a PASS labels the trajectory so only verified work is learned.
- **Self-report protocol** (`src/supervisor/detect.rs:82-95`): every response ends
  with a `<sup>{json}</sup>` status token (state/focus/next/carry/plan/memories),
  injected out-of-band and stripped from display; free counters + self-report fuse the
  loop detector (`detect.rs:506-510` LOOP_THRESHOLD=3, NO_PROGRESS_WINDOW=5 -- fixed
  constants, "not knobs", `supervisor/mod.rs:24-26`).
- Learning evolution defaults off (`default.toml:722-739`); self-generated guardrail/
  skill artifacts promote only via counterfactual shadow-control arms scored on
  pass-rate gap vs noise margin -- a genuinely safety-minded self-modification gate.
- No SECURITY.md. Absent project guardrails there is no default protection (pi-like),
  but it is not misrepresented.
Safety = 7: binds (deny before spawn, tested) AND has kernel machinery underneath --
above the 6 rung -- but the jail is opt-in, write-only, no egress control, which is
what separates it from codex's 10.

## Verification

- ~105k LOC tests, 3,943 test fns; property-shaped names and assertions
  (`gate_tests.rs` 52 fns, `conversation_compression/tests.rs` 114 fns,
  `tool_execution_tests.rs:41-47` asserts loop-advisory content).
- Faux-provider discipline: scripted stub SSE/chat servers + `install_fake_evaluation`
  seams (`src/session/chat/test_support.rs`, `gate_evaluate_tests.rs:15-30` -- a
  refutation call the seam should have replaced reads "SCRIPT EXHAUSTED").
- **PTY end-to-end**: `tests/interactive_pty_e2e_test.rs:15-18` drives the real
  binary inside a pseudo-terminal -- "the only way the interactive main loop,
  reedline input layer, and terminal rendering ever execute under test". Rare in
  corpus. Plus session-restore exactness tests (`tests/session_restore_tests.rs:15-21`)
  and ctrl-C cleanup tests.
- CI: 3-OS matrix + beta + nightly (`ci.yml:44-48`), coverage job publishing real
  numbers (`ci.yml:146-219`), cargo-audit (`dependencies.yml:33-44`), musl static
  builds. NO in-CI model evals: the SWE-bench-Live harness is an out-of-repo box
  process (`bench/README.md`) with committed `baseline.json` -- real eval discipline,
  but not CI-gated. No fuzzing. Verification ceilings at 8 per errata.

## Orchestration / interop / operability

- Supervisor as out-of-band control plane beside the loop (`supervisor/mod.rs:15-33`);
  delegation (`supervisor/delegate.rs`), durable schedules flushed into the session
  inbox (`main_loop.rs:1868-1872`), monitors, webhook + inject listeners per session
  (`main_loop.rs:53-70`).
- Workflows: declarative step graph (sequential/parallel/loop/conditional,
  `workflow/run.rs:23-27`) with `max_cost` ceilings and cost aggregation. No
  crash-resume journal for workflow runs found.
- Interop: MCP client AND server (`src/mcp/server.rs`, rmcp w/ elicitation
  `Cargo.toml:46-53`), MCP OAuth (`src/mcp/oauth`), ACP agent for Zed/JetBrains/Neovim
  (`doc/usage/12-editor-integration.md:13-27`, `src/acp/agent.rs` 1855 LOC), WebSocket
  server, headless `--format jsonl/plain`, `octomind send` to daemons. Homebrew-style
  **tap** registries distributing agents/skills/capabilities
  (`src/agent/taps.rs:15-27`) following the AgentSkills spec
  (`src/mcp/runtime/skill.rs:17-19`). No published SDK.
- Capability auto-activation: on each user message, the intent is embedded (local
  ONNX `muvon/octomind-embed`) and matched against capability triggers (mean-of-top-K
  cosine + margin); a hit registers the MCP servers with no LLM round-trip, LRU
  eviction over the active set (`src/mcp/runtime/capability.rs:21-33`).
- Operability: exact-state resume (tests above), titles sidecar for picker
  (`main_loop.rs:222-234`), session share bridge/upload (`src/session/share/`),
  `/report day|week|month` with project filters, process/terminal title
  introspection, STRICT config schema ("a missing [supervisor] section ... is a hard
  parse error", `supervisor/mod.rs:36-38`). Counterpoint: `panic = "abort"` in release
  (`Cargo.toml:29`) -- crash-bounce favors size over graceful degradation.

## Docs

56 in-repo docs, ~20k LOC, with reference/config/session-command/compression/
supervisor/guardrails/token-efficiency pages (`doc/usage/01-25`...). Install scripts,
shell completions with their own tests. No SECURITY.md.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | one sink-abstracted loop across 5 surfaces `response.rs:37-52`; supervisor out-of-band `supervisor/mod.rs:15-33`; no 5k+ product files; one crate, display.rs 4025 |
| verification | 8 | 3,943 test fns; PTY e2e `tests/interactive_pty_e2e_test.rs:15-18`; 3-OS CI `ci.yml:44-48`; no in-CI evals (bench/ external), no fuzzing -> errata ceiling |
| safety-enforcement | 7 | deny-before-spawn tested `tool_execution.rs:366` + `guardrails_tests.rs:58`; Landlock/seatbelt `sandbox/linux.rs:31-52` but opt-in, write-only, no egress |
| token-economy | 9 | fold economics `decision.rs:31-70`; hard ceiling `mod.rs:88`; idle keepalive `cache_keepalive.rs:17,198`; condense `condense.rs:15-33` |
| orchestration | 8 | inbox batching + schedules `main_loop.rs:1857-1872`; workflow graph + max_cost `run.rs:23-27`, `workflow-graph.toml:13`; no workflow crash journal |
| interop | 8 | MCP client+server, ACP Zed/JB/nvim `12-editor-integration.md:13-27`, WS, jsonl, taps; no published SDK |
| operability | 8 | exact-resume tests `session_restore_tests.rs:15-21`; share; strict config `supervisor/mod.rs:36-38`; `panic=abort` `Cargo.toml:29` |
| originality | 9 | PACT attention lanes; fold-economics gate; embedding capability routing; temporal guard DSL; counterfactual artifact promotion `default.toml:722-739` -- all verified in code |
| durability | 5 | 1 contributor (census), shallow clone; crates.io/Docker/website + systematic release CI; young (0.55.0); no SECURITY.md; bus factor 1 |
| docs-dx | 8 | 56 docs/20k LOC incl. reference + troubleshooting; install/completions scripts; no SECURITY.md |

**Weighted total: 79.5 (A).** Strongest: token-economy (9, with originality 9).
Weakest: durability (5). Boundary note: 79.5 is within 2 pts of the 78 A/B floor;
a stricter durability (4, pi-minus community) lands 78.5, still A; a stricter
architecture (7) would put it at 78.0 exactly.
