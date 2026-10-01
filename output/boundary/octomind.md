# Boundary re-review: octomind (independent, full rubric)

Provisional 79.5 (A floor). Fresh full read of /data/samples/agents/octomind: 212,225 LOC total under `src/` + `tests/`, of which ~110,381 LOC are test files and 3,943 `#[test]`/`#[tokio::test]` fns. `.git/shallow` present -- activity evidence-limited from local log, but HEAD 2026-09-26, tag 0.55.0, CHANGELOG and scheduled dependency workflow all current; not treated as dead. Read-only static review; nothing executed.

## Closest anchor

Closer to **crush (69)** than to codex or pi on shape -- one big crate with a 2k+ orchestrator loop, workflow/subagent plane, and no kernel-level enforcement on the default path -- but its scores sit above crush on verification, token economy and docs, below on safety defaults and durability. Weighted result (74.25) lands where crush and pi bracket: firmly inside B, not on the pi-like A floor.

## (1) Token-economy 9 claim -- ruling: 8.5, claim rejected

The two named pillars are real and tested:

- **Fold-economics gate.** `src/session/chat/conversation_compression/decision.rs:31-70` defines `FoldEconomics` as provider-price *ratios* (`from_pricing` reads each provider's published `cache_read/cache_write/output` prices, falling back to documented conservative constants at :44-50 when pricing is absent -- the fallback never silently disables the soft threshold). The mid-turn fold test at `decision.rs:143-146` is a genuine economic inequality: freed tokens x cache-read ratio x pace-projected remaining calls (Lindy horizon, `decision.rs:100-118`) must cover folder input+output+cache-rewrite cost. **Tested**: `amortization_tests.rs:36-172` (monotonicity in accumulated calls, turn-boundary exemption, pricing-derived ratios incl. cheap-folder/pricey-folder cases), `decision_pacing_tests.rs:18-160` (measured growth from usage checkpoints, adaptive fire-line doubling, runway floors).
- **Idle cache keepalive.** `src/session/cache_keepalive.rs:15-30`: a `max_tokens=1` ping against a frozen snapshot on the provider's own TTL cadence, fired only when the resolved provider exposes a `KeepalivePolicy` (Anthropic only today) -- pinging providers without an observable refresh primitive is refused as money burn. Exchanges are harvested for cost accounting. **Tested**: `cache_keepalive_tests.rs:43-224` covers every spawn-refusal condition and the cancel/idle/capture lifecycle.

These MEASURE real cache economics -- ratios from published prices, growth from actual usage counters, realized savings displayed from usage (`cost_tracker.rs:352-378`). But the 9 rung is codex's: deterministic cache-key-gated reuse plus three compaction tiers. octomind's nearest equivalent is the rolling marker frontier with monotonicity test (`cache.rs` + `cache_tests.rs:327` never-advances-behind-frontier), which guards marker movement, not prefix identity; and keepalive/economics are validated as *decision logic*, with no test tying predicted gain to realized cache-hit outcomes. Broad stack (measured-growth trigger, priced fold, TTL warming, savings visibility, recited blocks placed after the cached prefix, `api_executor.rs:269-271`) that equals pi's 8 pillar set and reaches jazz's 8.5 -- not codex's 9.

## (2) Default posture -- ruling: below the pi-6 rung; 5 (nanocoder rung)

Every enforcement layer is a boolean or a file the user must add:

- `sandbox = false` shipped default (`config-templates/default.toml:33`); when on, it is write-restricted only -- reads unrestricted, `~/.ssh`/`~/.aws` readable (`sandbox/linux.rs:22-28`, honest about needing namespaces for read isolation), best-effort fail-open on old kernels and no-op-with-warning on other platforms (`sandbox/mod.rs:44-48`).
- LLM tool-admission authorizer `enabled = false` (`default.toml:750-751`); the pre-call admission path exists and binds when on (`response/tool_execution.rs:301-363`).
- Guardrails (`[[guard]]` deny rules) bind pre-call (`tool_execution.rs:336-347`) and are well tested including fail-closed on a bad registry (`config/guardrails_tests.rs:83`) -- but "missing file = no authored rules" (`doc/usage/18-guardrails.md:54`).
- No human-in-the-loop approval prompt anywhere in the tool path -- unlike pi, which at least has the enforced blocking hook and a project-trust gate, and states its posture plainly in `SECURITY.md:50`. octomind has **no SECURITY.md**; its honesty lives in code comments and config annotations, not a security posture document.

Nanocoder precedent is explicit: real machinery + wrong default = below the 6 rung, not equal. octomind is the honest variant of that rung (no false safety indicator printed -- unlike the claurst runtime-lie case), so it sits *at* 5, not under it. The supervisor gate being default-on (`default.toml:744-745`) does not change this: it verifies completion quality, not safety.

## (3) Verification -- ruling: 8 correct, 9 barred by errata

3,943 test fns / 110k LOC test, module-local `*_tests.rs` specs asserting properties (amortization monotonicity, frontier monotonicity, guard fail-closed, deferral lapse semantics), plus integration suites (`tests/acp_e2e_test.rs`, `cli_run_e2e_test.rs`, `interactive_pty_e2e_test.rs`, `ws_e2e_test.rs`). CI: `cargo test` on ubuntu/windows/macos + beta (nightly continue-on-error) (`ci.yml` Test Suite job), llvm-cov coverage job with badge, musl cross builds, weekly `cargo audit` (`dependencies.yml:33,45`). Nothing in CI blocks on model behavior: the live supervisor matrix is `#[ignore = "live shared supervisor model; requires gateway credentials"]` (`src/supervisor/authorizer_live_tests.rs:21`), and the SWE-bench-Live harness (`bench/README.md`, committed `baseline.json`, `compare_to_baseline.py`) runs on an external box, not a workflow. No fuzzing. Per the ERRATA verification-8 ceiling (now codex/cline/pi), 8 is the correct placement; the bench harness is the cheapest missing wire to 9.

## (4) Architecture -- errata god-file check: pass, 7.5

No product/TUI-mode file exceeds the errata's 5k docking threshold: largest are `display.rs` 4,025 (display-only, single-purpose), `conversation_compression/tests.rs` 3,654 (test), `main_loop.rs` 2,183, `acp/agent.rs` 1,855, `websocket/server.rs` 1,651. The errata's 8->9 gap -- "loop/state model separated from every product surface" -- is *partly* held: TUI (`main_loop.rs`), ACP and websocket surfaces all drive the same `ChatSession` core (`acp/agent.rs:42,56`, `websocket/server.rs:25,59`), so no parallel re-implementations (nanocoder pattern absent). But `main_loop.rs:15` self-describes as "orchestrates all session operations" (listeners, webhooks, keepalive, API execution wiring in a 2.2k-LOC loop module) and `ChatSession` is a god-struct centered on `core.rs:185` (1,593 LOC). Policy is genuinely separated (`supervisor/` plane), which keeps it above crush's 7 fusion; it does not reach 8 (no SDK-first core, no enforced module boundaries). 7.5.

## Other lanes

- **Verification 8** (above; ceiling).
- **Orchestration 7.5**: workflow engine with sequential/parallel/conditional/loop steps, per-step session continuity with resume re-count dedup (`workflow/run.rs:107,143,977`), semaphore concurrency, session/request spending thresholds with a dedicated exit code (`config/mod.rs:357-360`, `main.rs:179-182`), subagent orchestration server + dynamic agents; no durable workflow journal or queue (vs cline 8 / codex 9).
- **Interop 8**: ACP agent over stdio driving Neovim/Zed/JetBrains (`doc/usage/12-editor-integration.md:8-14`), MCP client plus four builtin MCP servers, headless `run --format jsonl` with schema-pinned structured output, websocket server surface. No published SDK.
- **Operability 7**: named/resumed sessions with recent picker (`doc/reference/01-cli-reference.md:159-161`), RAII runtime guards and ctrl-C cleanup test, spending stop; no checkpoint/rewind, no datadir lock, no crash journal or panic-hook surface found.
- **Originality 8.5**: the supervisor plane is a distinctive control-plane design -- verify-gate with a strict evidence-hierarchy prompt and readback rounds judging END STATE over narrative (`supervisor/gate.rs:15-40`); PACT attention lanes where the runtime owns pins, provenance and attribution checks around the fold (`attention/mod.rs:15-52`, 2.1k LOC + 2.4k tests); priced fold economics (no parallel in the anchor set); citation-grounded guard evolution with replay cases and shadow trials, honestly disabled by default (`learning/evolution/synthesize.rs:1226`). Below 9: parts gated off by default, no field-defining breadth yet.
- **Durability 5**: solo bus factor, no SECURITY.md; offset by org CI reuse (`muvon/ci-workflow`), weekly cargo-audit, 0.55.0 release cadence, CHANGELOG. Shallow clone noted.
- **Docs-dx 8**: ~30 in-repo docs incl. an 870-line config reference, architecture doc, troubleshooting and migration guides; config template annotated inline. pi's 9 needs error-message/onboarding evidence not found.

## Weighted total

7.5x15 + 8x15 + 5x10 + 8.5x10 + 7.5x10 + 8x10 + 7x10 + 8.5x10 + 5x5 + 8x5 = 742.5 -> **74.25, band B**.

The provisional A floor does not hold. The gap is not one notch on a single axis as the original reviewer feared -- it is safety at the nanocoder 5 rung (enforcement machinery is real but every piece ships disabled and there is no honest security-posture doc), token-economy at jazz's 8.5 rather than codex's 9 (economics measured, but no prefix-identity/cache-key control and no realized-hit validation), and architecture 7.5 -- together ~5 below the provisional total.
