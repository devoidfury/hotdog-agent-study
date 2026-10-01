# agentty -- T2 deep review

- Subject: agentty (https://github.com/1ay1/agentty), C++26 terminal coding agent, MIT
- Version at HEAD: 0.9.13 (CHANGELOG.md:7, dated 2026-09-27)
- Snapshot: shallow clone (1 commit, `.git/shallow` present)
- Own code: `src/` 90.4k raw LOC + `include/` 37.3k raw LOC; cloc code in src+include excluding tests ~= 73k
- Tests: 239 files, 46,242 LOC (cloc SUM matches census exactly)
- Submodules: maya (TUI engine), acp-cpp, mcp-cpp, rag-cpp -- all same author (1ay1/*), NOT checked out in this snapshot

## Anchor question

**Which anchor subject is this closer to, and why?** Pi. Both are solo-authored,
strongly opinionated single-binary TUI agents with exceptional property-flavored test
corps and deep in-repo docs; agentty trails pi slightly on orchestration and durability
but exceeds it on enforced safety (a real, probed, tested OS sandbox where pi honestly
ships none). It does not reach codex's institutional breadth anywhere.

## Architecture lane: C++ memory discipline verdict

Modern C++ with unusual discipline, not legacy-shaped. Evidence:

- Elm architecture end-to-end: `(Model, Msg) -> (Model, Cmd<Msg>)`, four maya hooks bound
  in `include/agentty/runtime/app/program.hpp`; `docs/ARCHITECTURE.md:8-30` matches code
  (reducer per domain in `src/runtime/app/update/`, view per widget family in `src/runtime/view/`).
- Strong-typed newtypes for ids (`ToolCallId`, `ThreadId`, `OAuthCode`) so swapping ids is
  a compile error: `docs/ARCHITECTURE.md:52-55`, `include/agentty/domain/id.hpp`.
- `std::expected<T, E>` returns through io/http/persistence; closed enum error kinds,
  no `catch(...) { continue; }` (`docs/WHY-NOT-RUST.md:44-52`, spot-checked
  `src/io/http.cpp:1495`).
- 151 `static_assert`/`consteval` proofs across headers ("the build is the test runner"),
  `docs/WHY-NOT-RUST.md:11-16`; independently verified, e.g. the exhaustive 48-cell
  permission matrix proof in `include/agentty/tool/policy.hpp:66-90`.
- Raw `new`/`delete` is rare and deliberate: intentional-leak statics (`src/io/http.cpp:3165`,
  `src/io/tls.cpp:362`) and OpenSSL ex_data lifetime management (`src/io/tls.cpp:442,462,484`).
  119 smart-pointer uses; ASan/UBSan+Leak and TSan run in CI against the labelled sets
  (`.github/workflows/ci.yml:215,278,315`).
- Caveat: the agent loop lives inside the Elm reducer (`phase::Idle/Streaming/
  AwaitingPermission/ExecutingTool` FSM in `src/runtime/app/cmd_factory.cpp:1946-1958`),
  so loop state and UI state are the same Model -- coherent, but the TUI/headless paths
  each consume the loop through seams (`src/acp/server.cpp:8` keeps an own headless turn
  loop "against the same provider + tools + permission policy"). Largest files are
  reducer/view shards, not god files: max `stream.cpp` 2858, `cmd_factory.cpp` 2622,
  `turn.cpp` 2606.

## Core loop (line-level)

`src/runtime/app/cmd_factory.cpp`:
- `kick_pending_tools` (:1835+): dedups re-leaked salvaged tool calls before promotion
  (:1841-1844), then a PURE conflict-aware planner `schedule_parallel_batch` decides the
  concurrent wave; live gate is a thin consumer "so the test exercises the real rule,
  not a parallel reimplementation" (:1852-1858).
- Permission preflight is deliberately non-dispatching: if any sibling in the wave needs
  approval the function returns before marking ANY tool Running, because returning a
  prompt discards accumulated Cmds and mutating first would strand the tool (:1864-1887).
  Late-arrival windows (tool result after Esc-cancel) degrade to no-ops with total phase
  transitions (:1826-1833, :1957).
- Doom-loop circuit breaker `agent_loop_should_break` (:1699-1810, header doc
  `include/agentty/runtime/app/cmd_factory.hpp:154-184`): three triggers -- (1) same
  failing (tool,args) 3x always-on; (2) same byte-identical SUCCEEDING call 6x, applied
  to every model including unrecognised local ids, motivated by a measured 80-repeat
  loop on a 22GB local model (:1712-1724); (3) 25-turn runaway cap gated to weak models
  only, matching Claude Code/aider's no-hard-cap stance for capable models (:1801-1808).
  Fires as a corrective final assistant turn rather than a hard kill. Consulted at the
  continuation point before spending another completion (:2011).
- Tool wall-clock watchdog explicitly REMOVED at user request with the wedge trade-off
  stated in a comment (:1943-1949) -- honesty, but wedged tools hold their card until Esc.

## Compaction / context management (line-level)

`src/runtime/app/update/stream.cpp` + `src/runtime/app/cmd_factory.cpp`:
- Post-turn idle trigger at ~95% of window (`compaction_threshold() = min(95%, window-20k)`),
  and the rationale is cache-aware: ride DEEP because the repeated prefix is a
  prompt-cache HIT (1h Anthropic anchor) while each compaction is expensive and resets
  the cache -- firing seldom minimizes total burn (stream.cpp:1062-1070). This is
  cache-monotonic reasoning in the trigger itself.
- Proactive estimate vs lagging `tokens_in`: `max(tokens_in, est) > threshold`
  (stream.cpp:1086), where est is a byte/3.5 estimate with ~1500/img charge
  (`include/agentty/domain/conversation.hpp:922-956`) calibrated online by EMA
  `0.65*prev + 0.35*real_sample` (stream.cpp:1729-1732) after byte-estimates fired
  compaction with 60k+ headroom on tool-heavy threads.
- Soft-trim degradation ladder (`cmd_factory.cpp:382-460`): wire-only, REVERSIBLE front
  trim that never mutates the transcript; protects head, latest User, newest tool result;
  O(N) single-pass re-pricing after a noted O(N^2) fix (:343-378); trailing guard keeps
  the wire starting with User per Anthropic rules (:454-460). If compaction fails
  (rate limit), the next turn still goes through on soft-trim -- the agent never wedges
  (stream.cpp:1057-1060).
- Compaction is wire-substitution too: transcript immutable, `CompactionRecord` carries
  before/after token pair so "compaction that reclaimed 5 tokens" vs "200k" are
  distinguishable, and a rapid-refill breaker (compact within 2 turns x4 ->
  `autocompact_disabled`) that TEMPORARILY disables and re-arms after ~10 quiet turns
  (stream.cpp:405-428, 794-799; record fields `conversation.hpp:870-920`).
- Summary slice is capped (kCompactionSliceCap=150k) so the summary request stays cheap
  on 1M windows (cmd_factory.cpp:532-543). Five compaction styles behind one prompt map
  (:263-281).
- Prompt-cache breakpoint placement is SOTA-shaped and test-locked: a quantized 1-hour
  anchor + one rolling 5m pin, message pins <=2, total breakpoints <=4 because exceeding
  4 makes Anthropic evict the system pin (tests/cache_anchor_test.cpp:1-13;
  src/provider/anthropic/transport.cpp:346-401).
- Lazy loading: skills are tier-1 catalog line in the system prompt, body loaded on
  demand via the `skill` tool (prompt.cpp:549-552, skills.hpp:9,130-149); RAG
  (BM25+dense, RRF, GraphRAG, rag-cpp submodule) fetches source-tagged slices instead of
  repo dumps; tool-result budgets exist (tests/tool_result_budget_test.cpp).

## Permission / sandbox (line-level)

- `include/agentty/tool/policy.hpp` is THE permission source of truth: pure `constexpr
  Decision permission(EffectSet, Profile)` over (Exec/WriteFs/Net/ReadFs) x
  (Write/Ask/Minimal), no state/IO/model dependency (:37-53); every gating call in the
  runtime goes through it ("no bespoke per-tool lambdas", :5-8). Exhaustive
  compile-time proof over all 48 cells with a spec function + `static_assert` loop
  (:66-90) -- a policy change that breaks any cell is a build failure, no test run needed.
- Sandbox (`src/tool/util/sandbox.cpp`): bwrap (Linux) / sandbox-exec (macOS), on by
  default (`auto`), `--sandbox on` refuses to start without a backend (docs/SANDBOX.md:5-12).
  Availability is a GENUINE capability probe, not `which`: it runs the same namespace
  unshares against `/bin/true` and requires exit 0, fixing false "sandbox: active (bwrap)"
  on Ubuntu 24.04 AppArmor-restricted hosts (issue #21) (sandbox.cpp:66-98).
- SBPL profile injection guard: workspace path escaping with fail-CLOSED on
  unrepresentable control chars, unit-tested with the actual `/tmp/ws")(allow default)("`
  attack shape (sandbox.cpp:24-35; tests/sandbox_escape_test.cpp:1-13). Parity test
  exists (tests/sandbox_parity_test.cpp).
- Hooks are the classic auto-shell exfil surface; agentty gates them behind explicit
  consent: unseen/changed hooks file does NOT run, `hooks approve` prints the file and
  stores its SHA-256, any byte change revokes, and approved hooks run through the SAME
  sandbox wrapper (include/agentty/tool/hooks.hpp:28-48).
- Skill bodies are screened for documented prompt-injection patterns before activation,
  and approval is keyed to body+effects bytes, citing the arXiv 98k-skill study
  (skills.hpp:286-345).
- Stated residuals (honest, verified against code): network namespace SHARED so approved
  commands reach any host -- "read access plus network is read plus exfiltrate"
  (docs/SANDBOX.md:47-55); no Windows backend, fails loudly; Profile::Write allows
  everything (policy.hpp:38) but is opt-in; no separate egress approval (unlike codex).

## Verification

46k LOC / 239 test files against a bespoke agtest harness; tests are named like property
specs (sandbox_escape, doom_loop, cache_anchor, ssrf_guard, wire_golden,
provider_conformance, persistence_race, tool_wedge_liveness). CI (.github/workflows/ci.yml):
- build-test: ctest full suite on GCC-16/C++26 (ci.yml:192).
- sanitizers job: ASan+UBSan run of sanitizer-labelled tests (:278) AND TSan race-labelled
  run (:315) -- tsan coverage of concurrency is rare in this corpus.
- fuzz-smoke job: builds mcp-cpp fuzzers (asan+ubsan) and runs bounded bursts, crash aborts
  job (ci.yml:476-514: fuzz_fuzzy_match 5000, fuzz_apply_patch 20000). In-repo fuzz targets
  (frozen_invariant_fuzz.cpp, scrollback_wire_fuzz.cpp) are registered in the ctest suite
  (tests/agentty_standalone_tests.def:27-28) so they also run in build-test.
- Perf regression gate: `BENCH_ASSERT=1 ... long_session_bench` fails CI on render-path
  regressions (ci.yml:203-204).
- Windows MSVC compile-only gate + mingw runtime smoke (:328, :378, :433).
No in-CI model evals. Fuzzing evidence exists (the errata 9-gate is technically met by
ci.yml:476) but it is bounded-smoke-grade over two harnesses, not continuous coverage
fuzzing -- keeping verification at 8, the observed ceiling.

## Orchestration

- `task` subagent tool with provider-agnostic stream seam; read-only roles
  (explorer/reviewer) auto-route to the cheapest capable model, never cross-provider
  (include/agentty/tool/subagent.hpp:23-63).
- Smart Mode: Strategic/Implementation/Utility role -> (model, effort) resolution, pure
  resolver with degrade-to-parent zero-config (include/agentty/domain/smart_mode.hpp:1-40),
  per-repo learned tuning (domain/smart_tuning.hpp).
- Effect-conflict-aware parallel tool waves (schedule_parallel_batch, pure + tested).
- Message queue during runs (runtime/model.hpp:42-57), fork
  (src/runtime/app/update/fork.cpp, tests/fork_test.cpp), checkpoint rewind picker over
  git worktree+transcript, refusing while the agent works (checkpoints.cpp:130-133).
- No durable queue, no daemon, no crash-recovery journal, no budget caps on subagents --
  below cline's durability tier (sqlite cron, hub journal).

## Interop

- ACP agent side: `agentty acp` serves Zed (src/acp/server.cpp, docs/acp-editor-integration.md);
  plus the reverse direction: external ACP agents as turn backends
  (src/provider/external_acp_backend.cpp, docs/internal-acp-backends.md -- "one turn
  vocabulary for every backend" as the anti-drift thesis).
- MCP client (src/mcp/bridge, HTTP) AND server: `agentty mcp-serve` exposes native tools
  over stdio MCP (src/runtime/main.cpp:632-633).
- Headless `agentty run` with stdin piping (:784-786); 10+ providers incl. OAuth flows
  (Claude Pro/Max, Codex, Copilot, Kimi).
- Rival-ecosystem import: SKILL.md frontmatter (Claude Agent Skills format, incl.
  `allowed-tools`) and CLAUDE.md tiers are consumed (skills.hpp:17-31, provider prompts
  reference CLAUDE.md tiers).
- No published SDK; IDE surface is Zed-via-ACP only.

## Operability

Thread persistence with incremental save + blob GC (thread_save_incremental_test,
blob_gc_test), resume with scrollback rehydration (needs_warmup hook,
docs/ARCHITECTURE.md:32), fork/rewind/checkout-picker, diagnostics via logx with
redaction and rotation (tests/logx_redaction_test.cpp), credential encryption + keystore
(cred_crypt_test, keystore_test), live provider/model switching (^P), 12 packaging
ecosystems (packaging/: deb/rpm/nix/homebrew/scoop/winget/snap/termux/...), install.sh.
No crash-recovery journal; a mid-turn crash resumes from last persisted thread.

## Uniqueness highlights

1. Air-gapped mode: `agentty airgap user@host` runs the agent on an offline box with the
   laptop as relay via `ssh -R 1080` SOCKS5, TLS pinned end-to-end so the relay cannot
   MITM; exec()s into ssh on POSIX (src/airgap/airgap.cpp:1-30). No other subject in the
   corpus does this.
2. "The build is the test runner": 151 compile-time invariant proofs incl. exhaustive
   permission-matrix static_asserts.
3. Online token-estimate calibration (EMA against real usage) feeding the compaction
   trigger.
4. Cache-aware deep-ride compaction philosophy + quantized cache anchors.
5. Honest-docs density: WHY-NOT-RUST.md, RUST-CRITIQUE.md, SECURITY_AUDIT.md,
   corruption-analysis.md, postmortems -- docs that argue with themselves.

## Census sanity

- `test_loc: 46242` -- exact match to cloc on tests/. Correct.
- `non_test_loc: 106217` -- own src+include is ~73k cloc-code (127.7k raw). The census
  figure is plausible only if it counted raw lines or the (here unchecked-out) submodule
  trees; report as suspect-high but within 2x, not a tier-changing error (own code keeps
  it near the T1/T2 boundary; dispatcher-assigned T2 honored).
- `contributors: 1, commits: 1` -- shallow-clone artifact for commits; the CHANGELOG shows
  36 releases with a daily cadence (0.9.10 2026-09-24 -> 0.9.13 2026-09-27), so activity
  is unambiguously alive; solo-maintainer is likely real (single author across repo + all
  4 submodules).
- License: MIT file present, matches census. Original, not a fork; submodules are the
  same author's own libraries.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | Elm reducer core shared by TUI/headless/ACP; pure planner consumed by live gate (cmd_factory.cpp:1852-1858); strong-id newtypes; max file 2858 LOC; loop entangled with UI Model keeps it below pi |
| verification | 8 | 46k LOC property tests + ASan/UBSan/TSan ctest labels (ci.yml:278,315) + bounded fuzz job (ci.yml:476-514) + BENCH_ASSERT perf gate (ci.yml:204) + compile-time exhaustive policy proof (policy.hpp:66-90); no in-CI model evals, fuzzing is smoke-grade -> errata ceiling |
| safety-enforcement | 7 | constexpr policy binding in-loop with non-dispatching preflight (cmd_factory.cpp:1864-1887); capability-probed bwrap/sbx default-on + escape/parity tests; hash-bound hook consent; skill injection screening; residuals honestly documented (shared net, no Windows backend); no egress approval -> below codex |
| token-economy | 8 | cache-monotonic deep-ride trigger (stream.cpp:1062-1070), online calibration (:1729), reversible soft-trim ladder, quantized cache anchors test-locked (cache_anchor_test.cpp), lazy skills + RAG; one estimator basis everywhere (conversation.hpp:958-963) |
| orchestration | 7 | subagent role routing (subagent.hpp:26-40), conflict-aware parallel waves, queue/fork/checkpoint; no durable queue/crash journal/budgets |
| interop | 8 | ACP both directions + MCP both directions (main.cpp:632), headless run, SKILL.md/CLAUDE.md rival-format import; no SDK, one IDE surface |
| operability | 8 | resume+rehydration, fork/rewind picker, blob GC, logx redaction, cred encryption, 12 packaging channels; no crash journal -> below codex/pi 9 |
| originality | 8 | airgap relay, compile-time-proof policy, est_calibration, cache-aware compaction philosophy -- all verified in code; nothing quite field-defining at codex/pi scope |
| durability | 5 | solo maintainer (bus factor 1) but daily release cadence, 36 versions, MIT, Discord, wide packaging; v0.x; above nanocoder's 4 (has SECURITY_AUDIT.md, infra) |
| docs-dx | 8 | ~35 in-repo docs that match code incl. postmortems and residual-list SANDBOX.md; install.sh; usage text in main.cpp:606-640; docked from 9 for design-status docs describing unshipped states (internal-acp-backends.md "Status: design") |

Weighted total: 765/10 = **76.5 (B)**. Strongest: verification (with token-economy and
interop tied at 8; verification named for the sanitizer+TSan+fuzz+perf-gate+
compile-time-proof stack). Weakest: durability (5).

Boundary risk: 76.5 is within 1.5 of the A boundary (78). If synthesis weighs the fuzz +
TSan evidence as breaking the verification-8 ceiling, or scores docs at 9 (pi-style doc
depth), agentty crosses into A. Flagging rather than nudging.
