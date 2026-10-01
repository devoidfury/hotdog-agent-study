# Boundary re-review: agentty (T2, C++26, solo, MIT) - provisional 76.5 B, 1.5 under A floor

Independent full-rubric recount, 2026-09-30. Did not read subjects/scores/findings for agentty.

**Anchor sentence:** closest anchor is **codex** - the only cohort shape with kernel-level enforcement on the default path plus this grade of CI discipline; agentty sits at codex's verification methods on a solo budget, with codex-grade coverage gaps (Windows, egress) that keep safety well under codex's 10.

**Snapshot caveats:**
- `.git/shallow` present (1 commit, HEAD 2026-09-27 "release: 0.9.13"). Activity NOT inferred from history; CHANGELOG shows 0.9.12 (2026-09-26) -> 0.9.13 (2026-09-27), near-daily cadence, so alive on artifact evidence.
- Submodules (maya, mcp-cpp, acp-cpp, rag-cpp) are **not checked out** (`git submodule status` all `-`). The fuzz-smoke harness sources live in mcp-cpp (`cmake -S mcp-cpp -DMCP_BUILD_FUZZERS=ON`, ci.yml:501-505) and are therefore **not inspectable** - their oracle character is judged from the workflow only (evidence-limited, noted below).

---

## (1) Verification: 8 vs 9 -> **9 (CLEAN, ledger entry #6)** with recorded nuance

The ledger bar (smelt): oracled fuzz targets + regression seeds replayed in CI, and any 9 must cite the workflow line that **blocks** (gptme refinement, amazon-q adjudication).

**a) The fuzz job (ci.yml:476 "fuzz harness smoke") is CRASH-ONLY and bounded by fixed iteration budget - but it is not the substance of the 9.**
- ci.yml:515-516: `./fuzzbuild/fuzz/fuzz_fuzzy_match 5000` and `fuzz_apply_patch 20000` under `ASAN_OPTIONS=detect_leaks=1:abort_on_error=1`, `UBSAN_OPTIONS=halt_on_error=1` (ci.yml:509-511). Oracle = sanitizer abort; no property assertions visible; the workflow's own comment describes it as "same **crash** coverage on the reachable-from-a-tool locators" plus "hand-picked edge cases" (ci.yml:462-465). Harness bodies are in the mcp-cpp submodule (absent from snapshot) - **evidence-limited**, cannot upgrade to oracled.
- Blocking: no `continue-on-error` anywhere except prune-caches (ci.yml:58); ci.yml:466 says this job "Satisfies the 'fuzz harness smoke' required status check", and ci.yml:20-24 confirms required-check-by-name branch protection exists on master.

**b) The actual 9-qualifying fuzzing is in-repo, oracled, and runs in the primary blocking gate.**
- `tests/frozen_invariant_fuzz.cpp`: randomized property fuzzer with 7 structural oracles (I1-I7, :22-38) over the frozen-scrollback subsystem - lockstep, row accounting, bounds, never freezing the mutable stream tail, conservative trim-commit bound. Deterministic splitmix64, **fixed hardcoded seed family replayed identically every CI run** (main(): 4 widths x 120 walks derived from `0xA11CE5`, :425-433); failure prints `SEED=<n>` + full op trace (:146-155); retired op cases are deliberately kept numbered "so seeds stay comparable across versions" (:334-337).
- `tests/scrollback_wire_fuzz.cpp`: 5 **wire-level** oracles (W1-W5, :8-17) driving production mutation interleavings through the real renderer over a real pipe fd, asserting byte-for-byte agreement with an independent absolute wire model - "a wire row, once committed past the viewport top, is NEVER rewritten".
- Both are unlabeled folds -> correctness set -> run by `ctest ... -LE perf` in build-test (cmake/AgenttyTests.cmake:234-235; ci.yml:192). build-test has no continue-on-error: **this is the blocking line for the ledger**.
- Seed persistence: found failures are promoted to curated in-repo regression tests - the fuzz header names the pipeline ("The curated midrun_* tests pin specific, known-bad scenarios", frozen_invariant_fuzz.cpp:4-5); tests/midrun_freeze_test.cpp, midrun_seam_test.cpp, midrun_wire_test.cpp, scrollback_oracle_test, wire_golden/wire_audit/escape_guarantee/host_escape tests all run every CI. This is the human-curated equivalent of smelt's replayed-seeds corpus: seeds live in the repo and replay forever. There is **no automated corpus-append**; promotion is maintainer-mediated (nuance, not disqualifier - smelt's committed seeds are also repo-persisted).

**c) Sanitizer matrix: BLOCKING, but scope-capped.**
- sanitizers job (ci.yml:216): ASan+UBSan over `-L sanitizer` tests (ci.yml:278) then TSan over `-L race` (ci.yml:319, separate tree, halt_on_error). No continue-on-error -> failing sanitizer output fails the PR. Real motivation on file: the three shipped concurrency bugs all passed ASan and only TSan sees races (ci.yml:282-292).
- Cap: coverage is the sanitizer/race-labeled subset (registry: concurrency_primitives_test, race_harness_test, cred_crypt_test, keystore_test + marks), not the ~68k-LOC / 1081-TEST_CASE corpus, because maya's prebuilt renderer ODR-clashes with instrumentation (ci.yml:206-211). Structural ceiling on 9->10.

**d) The 48-cell compile-time policy proof is a REAL gate.**
- include/agentty/tool/policy.hpp:120-146: exhaustive `constexpr` sweep of `permission(e,p)` vs an **independently re-stated** spec `expected_decision` (:97-118) over 16 effect-sets x 3 profiles = 48 cells, plus a bit-width pin `static_assert(Effect::Exec == 1<<3)` (:144-146) so a fifth effect can't silently escape the sweep. A one-handed policy change fails **compilation** - the build-test job (ci.yml:100) cannot go green without it. Stronger than a test nobody runs; nothing else in the corpus does this.
- Corroborating gates: perf-regression gate `BENCH_ASSERT=1 long_session_bench` with ~6x ceilings blocks order-of-magnitude regressions (ci.yml:194-204); windows MSVC compile gate (ci.yml:326+) and MinGW gate with a native-pipe runtime smoke (ci.yml:440-470) exist because Windows breakage used to surface only mid-release.

**Ruling: verification 9.** Two oracled in-repo fuzzers with 12 asserted invariants run in the blocking primary gate; regression scenarios are persisted in-repo and replay every run; sanitizer matrix blocks; no CI model evals (that plus the crash-only external harness, 2-target fuzz breadth, and the labeled-subset sanitizer scope are what hold it under 10, alongside smelt's stronger 17-target count). Weaker than smelt's CLEAN, comparable to gemini-cli's softest blocker? No - strictly harder than gemini-cli's human-click: every line cited fails the build or the job. Ledger order: smelt > **agentty** > prime-agent > ouroboros > gemini-cli > openhands.

## (2) Safety-enforcement: **7 confirmed** - shipped-default binding, honest gaps

Shipped-default doctrine (octomind/forge): scored on what ships.
- **Default ON**: `--sandbox` empty => `Mode::Auto` (src/runtime/main.cpp:728, 1259-1263) wraps shell/diagnostics/git/process_start, lifecycle hooks (src/tool/hooks.cpp:237), external ACP agents (src/provider/external_acp_backend.cpp:199,205), and mcp-cpp tool runtime via `wire_mcp_runtime` (main.cpp:1290). deb packages **Recommends: bubblewrap** (packaging/deb/control.in:7) = installed by default on Debian-family; macOS sandbox-exec is always present.
- **The probe is a capability probe, not a `which`**: runs the same unshare set against /bin/true and requires exit 0 (src/tool/util/sandbox.cpp:84-100), because issue #21 was "sandbox: active" reported on hosts where userns was kernel-blocked - the false-safety-indicator failure mode, found and fixed. `--sandbox on` refuses to start with no backend (sandbox.cpp:437-443, main.cpp:1270-1274). Auto-degradation prints "unavailable, running unsandboxed" + the actionable kernel/sysctl cause (sandbox.cpp:477-500). Per the claw-code runtime-lie lesson, the degradation message states what is LOST - compliant.
- **Enforcement substance**: workspace rw, system ro, narrow named `/etc` file list and `$HOME` toolchain list with secrets deliberately excluded and a war story on file about a second backend that once bound `/` ("Read + exfiltrate", sandbox.cpp:127-231); fresh tmpfs, user/pid/ipc/uts/cgroup unshare, `--new-session` (TIOCSTI), `--die-with-parent` (:260-276). macOS side has a unit-tested SBPL injection guard with fail-closed clause-omission (sandbox.cpp:24-35; tests/sandbox_escape_test.cpp) and a two-engine parity test (tests/sandbox_parity_test.cpp).
- **Default Ask approval profile** (main.cpp:1659): prompts on Exec/WriteFs/Net; unknown tools fail closed (docs/ARCHITECTURE.md :252-256).
- **Under 8**: network namespace shared by design (documented accepted residual, docs/SANDBOX.md:43-56), **no Windows backend** (honest: fails loudly), bwrap not bundled (codex ships `bundled_bwrap.rs`; Arch/APK ship it as optdepends only, so install-without-recommends runs unsandboxed - but the banner discloses it, so nuance not hole). Read-set is hand-maintained (~20 binds, docs/SANDBOX.md:47). Codex 10 = all-platform kernel enforcement + separate egress approval; agentty is one solo project short of that. 7 stands, matching the hermes rung logic inverted (here kernel enforcement exists on the default path but is platform-partial).
- Security hygiene corroboration: docs/SECURITY_AUDIT.md records PKCE-CSRF-atomic-write-TLS-pin fixes with code locations; CHANGELOG 0.9.13 documents fixing vacuously-passing redaction tests that then caught a real leak in CI - a test-culture signal, not marketing.

## (3) Architecture: **8** (errata check passes, 9 declined)

- Errata >5k-LOC product-file check: largest product file src/provider/openai/transport.cpp **3,595**; src/io/http.cpp 3,276; update/stream.cpp 2,858 - no file over 5k. Header-sprawl check per the C++ gloss (header + paired source as one logical unit): src/provider/openai dir totals 3,806 (<5k); turn dir 5,834 but is 8 separately-headed units (turn.cpp 2,606 largest); update/ dir 16k is 8+ files behind internal.hpp (567). **No logical unit >5k.** (This check would have moved the row either way - it does not.)
- Shape: Elm-style reducer core (src/runtime/app/update/* one-file-per-concern reducers) over a single Model, Cmd-based async, effect-set-driven permission AND parallel-scheduling from one bitset (docs/ARCHITECTURE.md :245-280); TUI, headless `run`, ACP server (src/acp/server.cpp), and airgap all drive the same core through shared seams (main.cpp:1557-1639). No nanocoder-style parallel loops.
- 9 declined: layering is directory-level inside one binary, and cmd_factory.cpp (2,622) + main.cpp (1,864) carry heavy wiring; codex/pi 9 is a multi-package separation story. cline-8 rung is the accurate read (tested, coherent, migration-free, but fused wiring).

## Other lanes (recount, evidence-light but cited)

- token-economy **8**: SOTA prompt-cache breakpoint discipline - quantized 1h anchor + rolling 5m pin inside Anthropic's 4-breakpoint budget with eviction rationale, shape-locked by tests (tests/cache_anchor_test.cpp:1-14; src/provider/anthropic/wire_body.cpp:164-183); compaction threshold math locked as invariants with the cache-reset-cost rationale (tests/compaction_threshold_test.cpp:3-18); 5-rung context-window ladder with ORDER pinned by tests against a live 7,824-model snapshot (tests/context_ladder_test.cpp:1-24). Under 9: trigger is a fixed 95%-fill rule, not a measured projection of what the model receives (pi's pillar), no cache warming.
- orchestration **7**: doom-loop breaker walking the whole run per tool kick, threshold-3 (src/runtime/app/cmd_factory.cpp:94,118); subagents with per-thread nesting depth caps (src/tool/subagent.cpp:11-13); tool-budget test (tests/tool_budget_env_test); race-tested persistence (persistence_race_test, LABELS race). No queue/daemon/journal plane. crush-7 rung.
- interop **7**: ACP server + registry docs + Zed dogfood (README.md:31; src/acp/server.cpp:2,397), MCP client bridge with conformance test, headless `agentty run --events jsonl` machine contract (CHANGELOG 0.9.13). No published SDK - nanocoder-7 rung.
- operability **8**: every user turn pins a git worktree checkpoint with a rewind picker + async diff summaries (src/runtime/app/update/checkpoints.cpp:1-12; tests/checkpoint_test.cpp), fork + rewind mutating a persisted thread store with blob GC (tests/blob_gc_test.cpp; src/io/blob_gc.cpp), cross-process lock test. Under 9: no queue/archive/fork-journal surface.
- originality **8**: SSH airgap sessions (src/airgap/airgap.cpp), compile-time exhaustive trust-matrix proof (unique in corpus), wire-level scrollback fuzz with independent byte-exact wire oracle, context-window ladder - real, code-verified, but narrower than pi/codex 9 idea sets.
- durability **6**: solo maintainer (bus factor 1) offsets near-daily releases (CHANGELOG 0.9.12->0.9.13 in 24h, 36 releases), 5-distro packaging matrix, SECURITY_AUDIT.md. Shallow-clone caveat honored; activity from artifacts, not history.
- docs-dx **8**: 30+ in-repo docs incl. postmortems and corruption-analysis; sandbox doc states its own residuals ("read access plus network is read plus exfiltrate", docs/SANDBOX.md:53-56); curl-install one-liner.

## Totals and band sensitivity

| dim | w | score |
|---|---|---|
| architecture | 15 | 8 |
| verification | 15 | **9** |
| safety-enforcement | 10 | 7 |
| token-economy | 10 | 8 |
| orchestration | 10 | 7 |
| interop | 10 | 7 |
| operability | 10 | 8 |
| originality | 10 | 8 |
| durability | 5 | 6 |
| docs-dx | 5 | 8 |

**Weighted 77.5 -> B (band ceiling).** A floor NOT crossed. The verification 9 is real and earned; it does not carry the row alone because my recount prices durability 6 (solo, no institutional backing) and orchestration/interop 7. Sensitivity: docs 9 -> 78.0 (exactly on floor); arch 9 -> 79.0; both promotions rejected with reasons above. If the dispatcher's provisional non-swing lanes were each ~0.3 kinder, the same verification ruling lands 78.0 - flagged as a fragile-floor candidate, but the boundary recount is authoritative per the claw-code-agent precedent: **B FINAL**. This subject joins the B-ceiling cluster as the ONE member that broke the verification-8 ceiling - the plateau's binding constraint is elsewhere (durability + orchestration), which is itself tier-list material.
