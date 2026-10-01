# convergence.md -- concept-convergence across the coding-agent corpus

Built from `output/findings.jsonl` (1,666 canonical records, wave6 merge semantics per `wave6/merge-log.md`: deceptive-default family canon'd under phantom-safety-control, usage-measured-compact-trigger absorbed 3 weak-merge groups, rival-harness-session-import -> foreign-session-import, windows-sandbox-gap merge rejected). Concepts with >=2 subjects form the body below; 1-subject concepts are the appendix. Best-three picks are evidence-ranked (confidence high + test/fuzz/CI presence + defended mechanism > impact-only claims), one record per subject where possible. Per the study's open `verification-9` question (the `wave6/verification9-sweep.md` audit does not exist yet), **no concept below asserts verification-9 membership**; eval-blocking claims are stated only as 'the job blocks'.

## Stats preamble

- Records analyzed: **1,666** across **104** subjects (incl. hotdog's 15 own records).
- Distinct canonical concepts: **678** (712 pre-merge ledger baseline -> wave6 merges).
- Convergence set (>=2 subjects): **161** concepts; unique-material set (exactly 1 subject): **517** concepts.
- Kind distribution corpus-wide: portable 624, nuance 356, anti-pattern 306, unique 230, safety-hole 130, license-risk 20.
- Top 10 concepts by subject count: permission-policy (61); faux-provider-testing (47); compaction-tiering (43); loop-detection (36); sandbox-delegation (32); evals-in-ci (29); prompt-cache-marking (29); checkpoint-revert (27); hook-trust-scoping (25); god-file-loop (25).
- Design-tension sections: **69** of the 161 body concepts ship implementations that contradict; the rest are consensus designs or pure failure censuses (marked as such where doctrine-bearing).

## Narrative: the strongest convergence signals

**What everyone built.** The corpus has converged on a standard harness skeleton, and the convergence counts are the evidence: 61/85 subjects wrote a permission policy of some kind, 47 mock a provider for tests, 43 tier their compaction, 36 detect loops, 32 have opinions about OS sandboxing, 29 wire (or don't) evals into CI, 29 mark prompt caches, 27 checkpoint-and-revert. This is what maturation looks like: the question for nearly every mechanism is no longer *whether* but *which shape* -- and the shape disputes (below) are where the field is actually thinking. For a new harness, none of these clusters is optional; they are table stakes whose absence now reads as a defect rather than a design choice.

**What the field-defining few did differently.** The top of each cluster is not better-configured mediocrity, it is a different mechanism. 3code runs loop detection that spends zero model tokens (fingerprint window + Jaccard near-duplicate clustering with a replay harness) and inverts checkpoint authority so the *model* performs its own context surgery (dmail) while the filesystem stays ground truth. forge-norvialabs and agentty killed the "only giants afford kernel work" claim -- two solo C++/Rust projects ship default-bound sandboxes with live denial tests, and forge-norvialabs refuses to start on a host it cannot confine rather than running unprotected. opencode/kilocode/gemini-cli settled the compaction-trigger argument by *measuring the actual outgoing request* instead of estimating history or lagging on last turn's usage. kolkrabbi paid its architecture ratchet to zero and made the test fail on fixed-but-still-listed violations. smelt fuzzes cache-prefix stability as an invariant. Where the verification question bites, the leaders make their gates block: prime-agent's label-gated behavioral comparison, kilocode's release-needs-eval edge, deepseek-reasonix's anti-grader checkout discipline. (The corpus's verification-9 population audit is still owed -- this document deliberately asserts no 9-rungs.)

**What the crowded anti-patterns say.** The anti-pattern clusters are not about missing ideas; they are about machinery that does not fire. god-file-loop (25 subjects) and god-file-host-wiring (13) say the loop is the thing nobody refactors. test-suite-without-ci (15) is codebuff's 151,832 test LOC behind a build-only CI -- tests as aspirational artifact. quality-gates-unwired (11) plus eval-harness-outside-ci (7) plus broken-as-configured CI detail findings (warn-only "mechanically enforced" invariants, `|| true` lint, skip-message rot) form one meta-finding: **the field's dominant verification failure is wiring, not writing**. phantom-safety-control (15 subjects, 17 safety-holes) is the same disease in safety: permission managers with zero call sites, sandbox toggles read by nothing, TUI approval prompts no backend emits, docs-vs-shipped-default drift. The calibration doctrine prices exactly this shape -- and its mirror image, honest absence (pi, hax, opencode): honest disclosure earns one rung out of the misleading band, never a substitute for enforcement.

**The compaction/cache economics debate.** The most technically alive cluster group in the corpus. Three questions split it. *When to compact:* lagging previous-turn usage vs projecting the actual next request -- the evidence (opencode-e1, kilocode-e1, gemini-cli-e2, keen-code-b1, ferrum's block-the-send rule) lands on projection; hotdog's wire-size estimate is on the right side of the line with weaker arithmetic. *What to spend:* cheapest-first no-LLM tiers are consensus (crab-code's 7-level ladder, gemini-cli's four no-LLM tiers, workground2's fold-economics skip), pushed to its logical extreme by forge-1 (no model ever called to compact) and priced explicitly by octomind (folds fire only when profitable against cache-read/write token ratios) and deepseek-reasonix (the compactor bills itself: "a discarded answer is still a charge"). *What the cache permits:* the cache-monotonic school (waveloom's never-regressing decision set, ouroboros's byte-prefix invariant where every unsanctioned break is *counted*, atomic-agent's byte-identical stable prefix, mocode's cache-hit compaction fork, codebuff compacting only when the cache is demonstrably cold) is rewriting the algorithm, not the tuning. Per the calibration notes, token-economy 9s keep appearing outside the anchor set: this is the dimension where solo projects match institutions -- safety is the opposite (kernel work was assumed unaffordable until two solo projects proved otherwise). hotdog's stake is local-first: KV-warm fleet placement (cache warmth beats all candidate orderings; lane slots held across question waits so a swap cannot thrash the cache the user returns to) -- unique in corpus, counted under its own concept.

**The safety-default-posture debate.** The corpus's longest-running argument, and the one with an explicit doctrinal trail. Stage one: shipped-default doctrine -- octomind and nanocoder established that real machinery + wrong default sits below the 6 rung. Stage two: the hax ruling -- honesty about absent enforcement earns exactly one rung, upward only, out of the misleading (rung-2) band; enforcement always outranks honesty. Stage three: the kilocode collision -- a harness with strong opt-in enforcement and weak defaults broke the tie against opencode, and the study-wide amendment settled it: rung = the binding DEFAULT posture, plus at most ONE rung for an opt-in layer passing three mechanical gates (fail-closed; enforcement tests in a blocking CI job; non-deceptive), with silent fail-open capped below 6 and zero binding default capped at 5. The field's implementations map onto the debate cleanly: default-bind with live denial tests (codex, agentty, forge-norvialabs's confine-or-refuse, deepseek-reasonix's enforce-by-normalization) vs opt-in jails (gemini-cli, vtcode, letta-code, codewhale) vs yolo-defaults (kode-cli, zot, ob-1, bitfun) vs honest absence (pi, hax, opencode). Evidence clusters on default-bind; phantom protection is the only position everyone agrees is worse. hotdog's position is stated in our own findings file (hotdog-8): the best-tested fail-closed gate machinery in the corpus, shipped off, no jail beneath, bash bypassing the one always-on guard by construction -- rung 5 under the amended doctrine, and we scored ourselves 5.

## Body: concepts observed in >=2 subjects (161)

Sorted by subject count desc; hotdog-relevant clusters first within equal counts.

### permission-policy -- 61 subjects / 85 findings
kinds: unique 3, portable 25, nuance 24, anti-pattern 13, safety-hole 20

**Best three implementations** (evidence-ranked):
1. **agentty** (agentty-2, unique) -- Permission policy as a constexpr pure function with an exhaustive 48-cell compile-time proof - the build is the test runner `_(include/agentty/tool/policy.hpp:37-53)_`
2. **crab-code** (crab-code-3, portable) -- Deny-wins-as-security-boundary invariant, cross-layer tested, plus quote-aware per-segment deny for chained bash `_(crates/tools/src/permission.rs:76-115,277)_`
3. **dvalincode** (dvalincode-1, portable) -- Narrowing-monotone org policy with canonical source-order-independent hash, enforced at a single tool chokepoint `_(src/core/policy.ts:9-20,152-157,198-231)_`

**DESIGN TENSION.** approvals-default-on vs opt-in jail -- the field's central safety argument. Default-binding side: kolkrabbi-4/b4 (hardline floor survives yolo), minicode-1 (jail runs BEFORE mode dispatch and binds in allow-all), kimi-code-1, atomic-agent-4, hermes-agent-s1 (yolo flag frozen at import so an injected skill cannot flip it), kilocode-b6 (bash ships ask). Opt-in side: vtcode-s1, gemini-cli-s2, letta-code-s4, zot-4 and kode-cli-5 (yolo factory default), octomind (every layer ships disabled). The amended doctrine settled the scoring, not the design: rung = binding default posture, plus at most one rung for an opt-in layer passing three mechanical gates (fail-closed; enforcement tests in a blocking CI job; non-deceptive; silent fail-open caps below 6, zero binding default caps at 5). The strongest evidence -- proof tables, floor-tested tiers -- clusters hard on default-bind. hotdog-8 is our own entry on the wrong side of this line.

**HOTDOG POSITION.** hotdog-8 (anti-pattern) IS the entry: approvals ship off, Workspace containment covers file tools only. Ranked below all three above on our own admission; under the amended doctrine we are the octomind shape (real machinery, zero binding default -> rung 5) minus any phantom claim. Not competitive; the record exists precisely so this line is honest.

### faux-provider-testing -- 47 subjects / 48 findings
kinds: unique 1, portable 32, nuance 11, anti-pattern 4

**Best three implementations** (evidence-ranked):
1. **ob-1** (ob-1-4, portable) -- MockBrain scripted scenarios drive the REAL runTurn over REAL buildTools in 71 CI-gated no-key smokes, asserting wire behavior `_(scripts/parity-harness-smoke.ts:1-55)_`
2. **SWE-agent** (SWE-agent-3, portable) -- Canned PredeterminedTestModel + swerex DummyRuntime drive full-loop property tests of exit semantics `_(sweagent/agent/models.py:529)_`
3. **cline** (cline-2, portable) -- Loop-level property tests with fake providers assert approval flow and parallel-tool ordering `_(sdk/packages/agents/src/agent-runtime.test.ts:1821,2355)_`

**DESIGN TENSION.** in-process scripted provider vs whole-binary e2e behind an independent oracle. In-process mocks (hotdog-15's stream mocks, open-codex, mini-kode-7) prove loop logic fast but never exercise the shipped artifact. The top evidence drives the real binary against a scripted wire: hax-4/b3 (mock provider compiled into the binary; Python e2e under ASan/TSan on every push), grok-build-s10 (run_headless of the real binary vs mock sampling server), tura-9 (loopback-TCP SSE over the real pipeline + real tool execution), hermes-agent-c8 (loopback server speaking the OpenAI wire, recording every request body), codex-5 (mock SSE suite), deepseek-reasonix-c3 ('component correctness is not proof -- assert what reaches the provider'). The independent-oracle refinement -- hotdog-15's fixtures vendored from the official openai-openapi repo with declared derivation, keen-code-b6 wire-capture -- answers the self-referential risk of tests that regenerate the implementation's own output. Evidence clusters on scripted-wire + real-binary; no-faux-at-all (crab-code-6's closed backend) and silently-skipped integration (kode-cli-7) are the anti-poles.

**HOTDOG POSITION.** hotdog-15: stream mocks + independent-oracle conformance tier (openai-openapi-vendored fixtures, declared derivation). The oracle-independence claim is corpus-unique but everything is in-process -- no subprocess/PTY drive of the real CLI, so we rank below all three (hax/grok-build/tura run the real binary). Honest placement: mid-cluster (~rank 44/48).

### compaction-tiering -- 43 subjects / 44 findings
kinds: unique 3, portable 35, nuance 6

**Best three implementations** (evidence-ranked):
1. **hermes-agent** (hermes-agent-e4, unique) -- Five compaction tiers with cache-cost as the explicit ordering principle: no-LLM proactive prune with reclaim hysteresis, LLM mid-turn summarizer `_(agent/context_compressor.py:2695-2703)_`
2. **jazz** (jazz-b8, portable) -- Four-rung overflow ladder with config-enforced ordering (trim above compact above warn above clear) `_(packages/core/src/agent/context/context-window-manager.ts:21-41)_`
3. **minicode** (minicode-3, portable) -- Compaction tiering with anti-thrash circuit breaker, verbatim-pinned prior summary, hard summary budget, and compaction spend booked into the session `_(src/policy/compaction.ts:44-51)_`

**DESIGN TENSION.** cheapest-first no-LLM ladder vs single summarize-on-threshold. The consensus pole is ordered tiers with deterministic rungs first: crab-code-1 (7-level, no-LLM first, circuit breaker), waveloom-2 (four watermarks recalibrated from 148 real sessions with the rationale written down), oh-my-pi-e1 (five-method ordered ladder by default), gemini-cli-e1 (four no-LLM tiers under a two-pass summarizer), kolkrabbi-2 (sacrificial stages, stops at the first that fits, triggers only on provider-measured tokens), workground2-e1 (fold-economics skip + anti-re-compaction latch). The minority maximalist: forge-1, fully LLM-free deterministic fold, no model ever called to compact -- real and tested, but unproven at corpus scale. cline-1-class single-strategy designs now read as one-rung. hotdog's five-strategy registry with LLM-free overflow rescue rides the consensus pole.

**HOTDOG POSITION.** absent.

### loop-detection -- 36 subjects / 38 findings
kinds: unique 3, portable 13, nuance 19, anti-pattern 3

**Best three implementations** (evidence-ranked):
1. **3code** (3code-2, portable) -- call fingerprints over a bounded window with four signals `_(src/threecode/turns.nim:16-60)_`
2. **crush** (crush-1, portable) -- Repeated-tool-call loop breaker: sha256 call signatures over a sliding window `_(internal/agent/loop_detection.go:11-40,)_`
3. **kimi-cli** (kimi-cli-2, portable) -- Graduated repeat-call ladder (reminders at 3/5/8, hard turn-stop at 12) plus same-step dedup that awaits the original task and copies its result `_(src/kimi_cli/soul/toolset.py:117-167)_`

**DESIGN TENSION.** zero-token fingerprinting vs LLM-assisted checking, and hard-stop vs graduated intervention. Best evidence is token-free: 3code-2/b7 (fingerprint window + Jaccard near-duplicate clustering catching paraphrased thrash, 26 tests + a replay harness), kolkrabbi-3 (identical call AND identical result bytes, 9-slot cycle window, checked before the repeat executes), crush-1 (sha256 call signatures -- the anchor breaker), kimi-code-5/kimi-cli-2 (graduated reminders at 3/5/8, hard stop at 12, same-step await-and-copy). The LLM-assisted pole (gemini-cli-e8's periodic confidence check) adds per-turn cost on weaker evidence. Intervention polarity also splits: maki-2 dedups instead of aborting, opencode-b4 re-enters as a permission ask, ob-1-15 exempts mutating tools deliberately, hotdog-14 escalates nudge -> stronger nudge -> stop at t/2t/3t. Escalate-then-stop clusters on the stronger evidence; three subjects (mistral-vibe-e7, san-11, kolega-code-8) have no detector at all.

**HOTDOG POSITION.** hotdog-14: canonical-arg-hash repeats + strict ping-pong + t/2t/3t escalation ladder, tested. Below the best three (3code's four-signal Jaccard clustering, kolkrabbi's call+result bytes, crush's anchor breaker); the argument-canonicalization and level ladder are our refinements. Roughly #21/38.

### sandbox-delegation -- 32 subjects / 34 findings
kinds: unique 1, portable 7, nuance 11, safety-hole 15

**Best three implementations** (evidence-ranked):
1. **ob-1** (ob-1-5, portable) -- bwrap capability probed with the EXACT runtime argv and tiered hardened->base->none so an old bwrap still sandboxes instead of silently dropping to `_(src/safety/sandbox.ts:26-49)_`
2. **codex** (codex-2, portable) -- OS-native sandbox on all three platforms with denial/violation tests `_(codex-rs/sandboxing/src/seatbelt.rs:115-128,)_`
3. **agentty** (agentty-b3, portable) -- OS sandbox binds on the shipped default path with a capability probe instead of a which-probe, and degradation banners state what was lost `_(src/runtime/main.cpp:728,1259-1274)_`

**DESIGN TENSION.** OS jail binds by default vs opt-in jail vs none-by-doctrine. Default-bind: codex-2 (three platforms with denial/violation tests), gemini-cli-s1 (bwrap/seatbelt/native through one chokepoint), agentty-b3 (capability probe using the EXACT runtime argv, degradation banners that state what was lost), grinta-4, ob-1-5 (tiered hardened->base->none so an old bwrap still sandboxes; CI proves enforcement live). Opt-in: vtcode-s1, grok-build-s4, ipsupport-code-10 (kernel jail fails closed and is tested on a real macOS runner -- and ships default-off), nausicaa-harness-8. Doctrine pole: hax-3 ('because it runs inside the process it guards, it is not a security boundary -- run in a container or VM') and pi-6's honest 'intentionally does not have a sandbox'. The fail-open variants (nanocoder-1 silently degrading to plain sh, trae-agent-2's 3-tool Docker leak) are precisely what the shipped-default doctrine was built to punish. Evidence clusters on default-bind plus live denial tests.

**HOTDOG POSITION.** absent.

### evals-in-ci -- 29 subjects / 32 findings
kinds: unique 3, portable 9, nuance 10, anti-pattern 10

**Best three implementations** (evidence-ranked):
1. **ouroboros** (ouroboros-5, portable) -- Live-model lanes wired into CI: nightly paid E2E stand with explicit $30 cap and honest skip, plus real-provider skill-review smoke at $1.2/run `_(.github/workflows/ci.yml:9,71)_`
2. **letta-code** (letta-code-s8, unique) -- Gating CI runs full tool-calling agent scenarios against live models -- 4 hosted models and 4 local-backend provider models, each across `_(.github/workflows/ci.yml:376-408)_`
3. **gemini-cli** (gemini-cli-s9, portable) -- LLM-based evals run in CI: PR eval gated behind a maintainer environment approval plus nightly behavioral suite with llm-judge `_(.github/workflows/eval-pr.yml:87-92)_`

**DESIGN TENSION.** evals must block vs evals as telemetry. The corpus rule is mechanical and audit-confirmed: an eval counts only if the job BLOCKS on failure; continue-on-error evals are existence of harness, not enforcement. Blocking pole: prime-agent-1 (label-gated base-vs-head 28-task SWE comparison blocks releases), kilocode-b5 (publish needs-edge on a cost-capped real-model smoke against the built release binaries), deepseek-reasonix-s12 (anti-grader design: harness and suite always check out from the default branch so a PR cannot weaken its own benchmark), workground2-s10 (real-provider benchmark hardened against supply-chain abuse), letta-code-s8 (gating live-model behavioral CI, no continue-on-error), jcode-s7 (CI greps the log for skip messages and fails -- 'a skipped test still reports ok'). Telemetry pole: gptme-9 with gptme-b1's three independent off-switches, gemini-cli's env-gated PR eval, and the x7 eval-harness-outside-ci cluster. Evidence clusters cleanly on blocking-with-honest-skip; note the study's verification-9 claims live or die by this rule and the final 9-population audit is still owed, so this document asserts no 9-rung membership.

**HOTDOG POSITION.** absent.

### prompt-cache-marking -- 29 subjects / 31 findings
kinds: unique 2, portable 14, nuance 11, anti-pattern 4

**Best three implementations** (evidence-ranked):
1. **memcode** (memcode-5, portable) -- Stable/volatile system split with 1h doctrine breakpoint, last-tool breakpoint, and clone-only decoration — with a no-mutation property test `_(internal/providers/anthropic/anthropic.go:185-232)_`
2. **goose** (goose-e3, unique) -- Prompt-cache semantics are a declared per-(provider,model) property, not a side effect of the format module: lookback-aware breakpoint placement `_(crates/goose-provider-types/src/cache_semantics.rs:12-46)_`
3. **ob-1** (ob-1-8, portable) -- System prompt split into one cached stable block (instructions, AGENTS.md, skills, repo map) and an uncached volatile tail `_(src/agent/loop.ts:270-312)_`

**DESIGN TENSION.** engineered breakpoint discipline vs modeled-but-never-set. Engineered: kilocode-e2 (memory block pinned 'reused byte-identically' + per-provider placement), codewhale-e2 (pinned SHA-256, drift attribution that keeps counting misses), vtcode-e2 (breakpoint budgets, TTL policy, replay-faithful block order, miss detection), goose-e3 (cache semantics declared per (provider,model), unknown pairs default safe), memcode-5 (clone-only decoration with a no-mutation property test), smelt-4 (prefix stability as a FUZZED invariant), agentty-8 (quantized 1h anchor capped at the 4-breakpoint budget because a 5th evicts the system pin). The telltale anti-pole: claurst-8/b4 and claw-code-6 model cache_control and never construct it -- claw-code even runs a cache-break detector over a cache that was never enabled. keen-code-2's detail (plan-mode denial WRAPS tools instead of removing them so the tool-def prefix survives) is the refinement most subjects miss. hotdog ships no cloud breakpoints at all, deliberately -- local-first KV-warm placement is the substitute.

**HOTDOG POSITION.** absent.

### checkpoint-revert -- 27 subjects / 28 findings
kinds: unique 2, portable 12, nuance 13, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **minicode** (minicode-6, portable) -- temporary GIT_INDEX_FILE + write-tree pinned under refs/minicode/ as bare trees, undo = tree-diff restore of changed paths only, user index/HEAD `_(src/session/shadow-git.ts:12-27)_`
2. **3code** (3code-1, unique) -- the model reverts its own conversation to a [checkpoint N] marker mid-turn and carries a self-addressed note forward; filesystem deliberately NOT `_(src/threecode/prompts.nim:2983-2987)_`
3. **amazon-q-developer-cli** (amazon-q-developer-cli-6, portable) -- Shadow bare-git checkpoint manager: per-turn tagged commits against the work tree with restore/diff/stats and cleanup, no user repo pollution `_(crates/chat-cli/src/cli/chat/checkpoint.rs:34-36,149,207,259-334)_`

**DESIGN TENSION.** what gets reverted (filesystem, conversation, or both) and by whose authority. Consensus: workspace+conversation as one unit via shadow git -- amazon-q-6, minicode-6 (refs-pinned bare trees, user index/HEAD provably untouched), ob-1-16 (own GIT_DIR so bash-made changes are revertible too), codewhale-c8 (side repo, explicit non-fatal failure model), bitfun-c7 (persisted revert phase state machine). Conversation-only: continue-c8, qwen-code-c7 (deliberately excludes shell-made changes). Authority inversion is the frontier: 3code-1's dmail and kimi-cli-1 let the MODEL revert its own context mid-turn with the filesystem kept as durable ground truth. Coverage-honesty separates the top: deepseek-reasonix-c6 refuses to restore files under partial coverage instead of pretending; kilocode-c6 declares its degradation state before attempting. gemini-cli-c8's non-transactional rewind (history rolled back, files half-reverted) is the failure pole.

**HOTDOG POSITION.** absent.

### hook-trust-scoping -- 25 subjects / 26 findings
kinds: portable 13, nuance 6, safety-hole 7

**Best three implementations** (evidence-ranked):
1. **bitfun** (bitfun-s4, portable) -- Hook approvals are scoped to waive only the interactive prompt and are structurally checked after policy Deny returns `_(src/crates/assembly/core/src/agentic/tools/pipeline/tool_pipeline.rs:666)_`
2. **claurst** (claurst-5, portable) -- project-MCP trust: fingerprint-keyed allowlist stored outside the repo, project settings barred from granting themselves trust `_(core/src/mcp_trust.rs:1-27)_`
3. **codebuff** (codebuff-4, portable) -- Dual trust gates: inventoried .agents-dir prompt before any repo code import or MCP spawn, publisher allowlist before eval'd remote handleSteps `_(cli/src/utils/agent-dir-trust.ts:13-35,)_`

**DESIGN TENSION.** repo-shipped executables are inert until trusted vs clone-and-run. Trust-gate pole: grok-build-s7 (folder trust is the single fail-closed gate for hooks/MCP/LSP AND under sandbox profiles the kernel write-denies the trust files themselves -- store poisoning is blocked at the OS), pi-4, codebuff-4 (inventoried prompt trust before any repo code import; publisher allowlist before eval'd remote code), claurst-5 ('a repo can never grant itself trust' -- while claurst-b3 violates exactly that one line below its sibling guards), agentty-6 (content-hash-bound approval, revoked on any byte change, execution inside the same OS sandbox), zap-coding-agent-5, ferrum-b2 (repo config can only NARROW: intersect allow-lists, narrow roots, disable MCP -- never loosen). Clone-and-run pole (7 safety-holes): waveloom-4, memcode-6, claurst-b3, cline-b2, mini-kode-b3, kode-cli-6. The calibration notes log clone-and-run hooks at x4 and upgraded hotdog's audit item to must-have. Evidence is unanimous: gate it.

**HOTDOG POSITION.** absent.

### god-file-loop -- 25 subjects / 25 findings
kinds: nuance 1, anti-pattern 24

**Best three implementations** (evidence-ranked):
1. **qwen-code** (qwen-code-c2, anti-pattern) -- sendMessageStream is a single ~2070-line generator fusing goal permits, telemetry spans, budgets, reminder injection, and compaction coordination `_(packages/core/src/core/client.ts:3055-5125)_`
2. **claw-code** (claw-code-4, anti-pattern) -- 19,831-LOC CLI main.rs (plus 10.9k tools/lib.rs, 7.2k commands/lib.rs) holds ~33% of all Rust code above an otherwise-clean loop `_(rust/crates/rusty-claude-cli/src/main.rs:19,831)_`
3. **prime-agent** (prime-agent-3, anti-pattern) -- agent-session.ts ballooned to 15,101 LOC and interactive-mode.ts to 12,614 LOC `_(packages/coding-agent/src/core/agent-session.ts:1518)_`

**DESIGN TENSION.** No implementation contradiction -- a pure failure census, and the study's most crowded architecture hole: the turn loop fused into a monolith (claw-code's 19,831-LOC main.rs holding a third of all Rust, bitfun's 21,911-LOC coordinator, code's 14,929-LOC streaming.rs with a 1,235-line submission_loop, opensquilla's ~6,820-line generator in a 245-method class). The counter-design lives in the layered-loop-separation cluster (gemini-cli-c1, oh-my-pi-c1, orca-agent-c3): the field demonstrably knows the fix; 25 subjects ship without it.

**HOTDOG POSITION.** absent.

### security-posture-docs -- 24 subjects / 27 findings
kinds: unique 1, portable 11, nuance 12, anti-pattern 1, safety-hole 2

**Best three implementations** (evidence-ranked):
1. **openhands** (openhands-8, portable) -- Deployment docs state the threat model bluntly, and extension spec states what its guards do NOT guard `_(docs/SELF_HOSTING.md:4-8,47-56)_`
2. **workground2** (workground2-s12, portable) -- SECURITY.md enumerates supported boundaries, trusted inputs, and @-path invariants `_(SECURITY.md:49-104)_`
3. **minicode** (minicode-10, portable) -- One-page threat model naming the deterministic execution chain (permission -> jail -> guard -> validation -> executor -> journal) with per-mode `_(docs/security-model.md:1-30)_`

**DESIGN TENSION.** SECURITY.md as threat model vs reporting boilerplate. Honest pole: pi-7 (states exactly what is NOT protected + three containerization patterns), hermes-agent-s3 (one named load-bearing boundary, every other layer graded non-boundary, 'They are useful. They are not boundaries.'), ferrum-9/b4 ('Ferrum is not a sandbox' as a design note), minicode-10 (deterministic execution-chain one-pager, 'no absolute safe'), workground2-s12, letta-code-s6. Deception-adjacent pole: gemini-cli-s13 and zeroclaw-s14/s14-b4 (layers presented without the default-off disclosure), qwen-code-s10 (9-line intake form). This cluster is where the honesty doctrine prices docs: honesty-credit is exactly one rung upward out of the misleading band and never substitutes for enforcement -- which is also why phantom protection (phantom-safety-control) scores below honest absence.

**HOTDOG POSITION.** absent.

### usage-measured-compact-trigger -- 23 subjects / 25 findings
kinds: unique 1, portable 10, nuance 12, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **keen-code** (keen-code-b1, nuance) -- Compact trigger measures the actual serialized outbound request before EVERY tool turn - but tokens stay a len/3 heuristic and provider usage never `_(internal/llm/anthropic.go:593-596,553-563)_`
2. **gemini-cli** (gemini-cli-e2, portable) -- Compression triggers and post-compression budgets use real API prompt-token counts, not estimates, with an inflation veto `_(packages/core/src/context/chatCompressionService.ts:495-499)_`
3. **opencode** (opencode-e1, portable) -- V2 compaction fires on a measurement of the exact outgoing provider request, not on post-hoc usage counts `_(packages/core/src/session/compaction.ts:232-242)_`

**DESIGN TENSION.** lagging previous-turn usage vs projected next-request measurement -- the brief's named pair, present verbatim and settled by the evidence. Projection pole (winning): opencode-e1 ('a measurement of the exact outgoing provider request, not on post-hoc usage counts'), kilocode-e1, gemini-cli-e2 (server-reported prompt-token counts + inflation veto), keen-code-3/b1 (json.Marshal the actual outbound request before EVERY tool turn -- with the honest boundary that tokens stay a len/3 heuristic), ferrum-3/b7 (pre-request projection; still-over-budget after compaction hard-BLOCKS the send), orca-agent-e2 (wire-equivalent incl. serialized tool schemas), goose-e1 (usage + unreported tool tail). Lagging pole: mistral-vibe-e2 (previous turn's API usage, overflow caught reactively once per turn -- filed anti-pattern), roo-code-e5, mini-kode-5 (hardcoded 115000 against any model). hotdog-10 is projection-family with a local-first twist -- measure at the session's wire size through the resolved WireFormat -- ranked below the measured-request exemplars, above the lagging users.

**HOTDOG POSITION.** hotdog-10: wire-size projection through the resolved WireFormat -- projection-pole (right side of the tension) but an honest chars/4 heuristic, no provider-usage feedback loop. Below all three (opencode/kilocode/gemini-cli measure real request or server counts); above the lagging-usage anti-patterns.

### regex-denylist -- 22 subjects / 25 findings
kinds: portable 2, nuance 9, anti-pattern 11, safety-hole 3

**Best three implementations** (evidence-ranked):
1. **minicode** (minicode-2, portable) -- normalize-then-match (quotes, simple-var inlining with chained rescan, cmd.exe caret escapes) with honest limits stated and paired `_(src/policy/bash-guard.ts:1-19)_`
2. **roo-code** (roo-code-s2, portable) -- bash/zsh expansion classes (${var@P}, ${!v}, =(...), *(e:...:), <<<$(...)) demote any auto_approve to ask_user, never to deny, and survive prefix `_(src/core/auto-approval/commands.ts:22-58)_`
3. **orca-agent** (orca-agent-s5, anti-pattern [conf med]) -- Workflow JS runs in node:vm behind a hand-rolled lexical identifier deny-list while the host holds full user FS rights; concatenation-split property `_(crates/orca-runtime/src/workflow/host.mjs:11-25)_`

**DESIGN TENSION.** deny-lists vs grammar/structural parsing -- effectively resolved inside the corpus's own words: hermes-agent-s8's in-code ceiling ('Shell is Turing-complete; a denylist over shell strings is structurally incomplete'). Worst instances are safety-holes (claw-code-agent-b3: 11 regexes feeding shell=True with zero sandbox; mini-kode-2 first-token-only checks riding compound-command grants; zot-b2 deny-by-substring). The parsing pole owns the evidence -- see grammar-parsed-bash-policy (16 subjects). Honest middle ground with real tests: minicode-2 (normalize-then-match, bypass classes written down, must-deny/must-allow oracle probes run in CI), roo-code-s2 (expansion classes demote auto-approve to ask, never to deny), grinta-9/tura-6 (de-obfuscate before classifying). The honest-labeled denylist (mocode-11, kolkrabbi-8 'not a perimeter') survives only as an accident-guard under real containment.

**HOTDOG POSITION.** absent.

### workflow-resume-journal -- 19 subjects / 22 findings
kinds: unique 1, portable 16, nuance 3, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **hotdog** (hotdog-4, unique) -- Workflow runs journal state transitions to append-only run.jsonl, claim the run dir with a pid+host+heartbeat owner file `_(src/extensions/workflows/engine.ts:6-14)_`
2. **kolega-code** (kolega-code-2, portable) -- resume matched by content cache_key, not position: replays survive script edits, pipeline order drift, and resume-of-resume `_(kolega_code/agent/orchestration/journal.py:18-27)_`
3. **qwen-code** (qwen-code-e5, portable) -- Workflow runs write a checkpoint journal whose survival marks a crashed run, claimed and reconciled by the next process `_(packages/core/src/agents/workflow-checkpoint.ts:10-22)_`

**DESIGN TENSION.** journal that replays vs journal that merely records -- and who owns a live run. Reconcile-and-resume pole: hotdog-4 (append-only run.jsonl + pid/host/heartbeat claim with live-owner refusal + resume that reuses only claims the filesystem verifies), zeroclaw-e5 (claims persisted BEFORE admission; failed persistence retains the run), codewhale-e5 (restart-reconciled fleet ledger, orphan recovery), qwen-code-e5 (a surviving checkpoint IS the interrupted-run signal, claimed by the next process), tura-8 (resume as typed protocol: LeaseConflict/StaleLease, orphan-worker reaping), kolega-code-2/b3 (content-addressed FIFO keys so replay survives script edits and resume-of-resume). Record-only pole: molt-9's hash-chained journal that nothing replays ('a crash or closed window ends the working session permanently') and grok-build-e10 (journal kept, recovery explicitly terminal). Double-drive safety separates the leaders: without liveness-checked ownership, resume is a race. Evidence clusters hard on claim+journal+reconcile.

**HOTDOG POSITION.** hotdog-4: append-only run.jsonl + pid/host/heartbeat claim + filesystem-reconciled resume, tested including wrong-workflow refusal. Ties the listed best three on evidence (dogfooded by this study's own T3 runs); one of exactly two clusters where hotdog is genuinely top-tier.

### cache-monotonic-compaction -- 19 subjects / 21 findings
kinds: unique 1, portable 14, nuance 6

**Best three implementations** (evidence-ranked):
1. **atomic-agent** (atomic-agent-3, portable) -- stable prefix pinned byte-identical per session (tests assert byte-equality), everything a step can change placed AFTER ### conversation in the `_(src/prompt/stable-prefix.ts:84,156,221,232,293)_`
2. **ouroboros** (ouroboros-4, portable) -- Append-only byte-prefix transcript invariant: compaction seams stamp a sanction, every other cache break is counted in usage `_(ouroboros/transcript_prefix.py:1-16)_`
3. **mocode** (mocode-3, portable) -- Compaction fork reuses the parent request prefix so the summarizer call itself is a cache hit `_(src/session/compact.ts:1044-1046)_`

**DESIGN TENSION.** compaction as cache-breaking event vs compaction monotonic against the cached prefix. Monotonic pole: waveloom-1 (decision set never regresses, so compaction never rewrites the prefix), ouroboros-4 (append-only byte-prefix invariant; every non-sanctioned break is COUNTED in usage), atomic-agent-3 (stable prefix byte-identical per session, tests assert byte-equality; everything mutable placed after a conversation seam), mocode-3 (the summarizer call itself reuses the parent prefix so compaction is a cache hit), jazz-2/jazz-b2 (pressure nudges are request-only ephemeral suffixes that never move the breakpoint), hax-1/b5 (summarization advertises the full live tool set to preserve the cached prefix; calls answered [rejected], never executed; image budget drops only the just-read image so prior requests stay byte-stable). The economic extreme: codebuff-1 compacts only when the provider cache has demonstrably gone cold; octomind-2 fires folds only when priced profitable against cache-read/write ratios. Opposing default: rewrite-on-threshold, which the no-LLM-tier movement mitigates for cost if not for cache. The clearest 'economics changed the algorithm' cluster in the corpus.

**HOTDOG POSITION.** absent.

### grammar-parsed-bash-policy -- 16 subjects / 18 findings
kinds: portable 14, nuance 4

**Best three implementations** (evidence-ranked):
1. **ferrum** (ferrum-1, portable) -- Tree-sitter-bash rejection policy with three enforced tiers; docs capability table IS the regression test `_(src/tools/shell_guard.rs:82,1267-1305)_`
2. **vtcode** (vtcode-s7, portable) -- tree-sitter strict + lenient parsers, safe-by-subcommand registry, prefix-rule policy DSL, 5 fuzz targets on exactly these parsers `_(crates/codegen/vtcode-safety/src/command_safety/safe_command_registry.rs:7-22)_`
3. **smelt** (smelt-2, portable) -- Shell-grammar walker infers tool effects instead of regex deny-listing `_(crates/core/src/permissions/bash.rs:321-660)_`

**DESIGN TENSION.** parse-and-fail-closed vs match-and-hope -- and inside the parse pole, dependency vs hand-rolled. Everyone agrees the deny-list is dead; the disagreement is what happens at the non-total edge of parsing. Best-evidence pattern: tree-sitter-bash + fail-closed on every path where the parse stops describing what runs -- ferrum-1/b1 (deny on parser-unavailable/parse-fail/syntax-error/256KB/20k-node/256-depth, ships ON at medium, 'the docs capability table IS the regression test'), grok-build-s3 (fixed-point wrapper normalization fails closed; a parse shorter than the script fails), kimi-code-b4 (unanalyzable -> ask in default mode; wrappers parsed to depth 4), mistral-vibe-s1 (line continuations, heredoc bodies, env-assignment prefixes all add approval reasons), waveloom-7 (parser-DIFFERENTIAL: 23 checks where mvdan/sh AST and naive shell-quote tokenization disagree route to ASK, never silent allow), vtcode-s7 (5 fuzz targets on exactly these parsers). hotdog-7 is the corpus's hand-rolled bail tokenizer: no parser dependency, same soundness move ('when it is unsure it asks', and under default-deny a bail denies) -- one tier below the fuzz-tested tree-sitter subjects on evidence, correct on polarity.

**HOTDOG POSITION.** hotdog-7: hand-rolled bail tokenizer, correct polarity (unsure -> ask/deny, default-deny makes bails deny), but no tree-sitter, no fuzz targets, and the gate it feeds is off by default. Rank ~last among the 16 -- the design idea converges, the evidence tier does not.

### session-rollout-journal -- 15 subjects / 15 findings
kinds: unique 1, portable 6, nuance 8

**Best three implementations** (evidence-ranked):
1. **gemini-cli** (gemini-cli-c5, portable) -- Append-only JSONL session journal with collision-safe naming, index-based resume, cleanup and export `_(packages/core/src/services/chatRecordingService.ts:512-518)_`
2. **g3** (g3-8, nuance) -- Eager per-iteration session.json snapshots marked 'running', with unanswered-tool-call trim on restore to satisfy Anthropic's tool_use/tool_result `_(crates/g3-core/tests/eager_session_write_test.rs:1-25)_`
3. **oh-my-pi** (oh-my-pi-c5, portable) -- Versioned id/parentId JSONL session tree with migrations, lenient corrupt-record loading, and an explicitly honest durability ceiling `_(packages/coding-agent/src/session/session-migrations.ts:13-51)_`

**DESIGN TENSION.** Consensus (append-only JSONL) with graded crash-semantics craft: hax-6 (flush per newline; /undo APPENDS a cut record instead of truncating; torn final line repaired on resume, with a test), san-9 (turn-gated fsync, torn-tail rejection, crash-rebuildable derived index), qwen-code-c6 (corruption-tolerant parsing backed by a 1.3k-LOC corruption suite), mistral-vibe-c3 (O_APPEND 0600 + fsync, atomic rewrite, owner-only modes, rationale 'session logs hold raw tool results'), zeroclaw-c6 (provenance sidecars so restore never infers). hotdog's session layer (strict resume-by-id, replay guard, traversal-rejected ids, persisted rewind markers) sits credibly inside this tier but its evidence lives in the subject file, not a findings record.

**HOTDOG POSITION.** no findings record here; the subject file evidences strict resume-by-id + replay guard + traversal-rejected ids -- credible mid-cluster, uncited here and therefore unclaimed.

### phantom-safety-control -- 15 subjects / 21 findings
kinds: nuance 1, anti-pattern 3, safety-hole 17

**Best three implementations** (evidence-ranked):
1. **code** (code-m1, safety-hole) -- Docs promise a Windows restricted-token sandbox with a passing smoketest harness that does not exist in the shipped tree - flagship instance of a `_(docs/platform-sandboxing.md:13)_`
2. **continue** (continue-s5, anti-pattern) -- Injection sanitizer has exemplary real-shell end-to-end tests but is called by nothing: quality gates wired to dead code `_(core/util/sanitization.vitest.ts:178-228)_`
3. **binharic-cli** (binharic-cli-2, safety-hole) -- PermissionsManager with rules, allow/block lists, session/project/global scopes and sensitive-path patterns has zero call sites in src or tests; the `_(src/agent/core/permissionsManager.ts:47,90,120)_`

**DESIGN TENSION.** Failure census, doctrine-bearing -- the rung-2 family canon'd here per wave6 (deceptive-default variants: permissive-default-vs-docs, default-posture, mode-vocabulary-drift ride as aliases). Occupants: phantom /sandbox-toggle read by nothing (claurst-3/b1), a full PermissionsManager with zero call sites (binharic-2), a TUI that renders /approve prompts no code emits while the gateway hardcodes allow-all (tura-5), --sandbox meaning git-worktree (forge-5), docs promising a Windows restricted-token sandbox whose harness does not exist (code-m1), shipped config self-declaring 'out-of-box bypass' while docs claim a Safe default (opensquilla-b2/default-posture/mode-vocabulary-drift), schema default auto-approve vs docs 'manual (Default)' (dexto-s2). Under the amended doctrine this shape is exactly what the +1 opt-in credit never rewards, and per the claurst boundary it outranks plain absence in damage: phantom protection is the codel-rung tell. hotdog sits deliberately on the honesty side -- the gate visibly ships off and the docs say so, which is why safety 5 is not 2.

**HOTDOG POSITION.** absent.

### test-suite-without-ci -- 15 subjects / 18 findings
kinds: nuance 2, anti-pattern 16

**Best three implementations** (evidence-ranked):
1. **codebuff** (codebuff-7, anti-pattern) -- 151,832 LOC of behavior-asserting tests that the public CI never runs -- CI is build + binary smoke only `_(.github/workflows/ci.yml:33-58)_`
2. **ferrum** (ferrum-6, anti-pattern) -- 477 tests including whole-binary protocol conformance with zero CI to run any of them `_(tests/acp_stdio.rs:1702)_`
3. **g3** (g3-5, anti-pattern) -- ~27.6k LOC of property-named tests and zero CI: no .github/, no workflow files, nothing runs the suite automatically `_(find crates -path '*/tests/*' -name '*.rs' | wc -l = 26,635 LOC)_`

**DESIGN TENSION.** Failure census and the starkest verification signal: 15 subjects ship genuine behavior-asserting corps that nothing ever runs -- codebuff's 151,832 LOC behind a build-only CI, ferrum's ~19k LOC + 21-task bench with ZERO workflow files, g3's 27.6k property-named tests, grok-cli's 253 property tests, mini-kode's 276 cases gated only by prepublishOnly habit. No implementer side -- the implementers live in architecture-contract-test, evals-in-ci, and jcode's anti-skip discipline. The study rule it feeds: test mass without execution earns no credit.

**HOTDOG POSITION.** absent.

### injection-screening -- 14 subjects / 14 findings
kinds: portable 1, nuance 6, anti-pattern 1, safety-hole 6

**Best three implementations** (evidence-ranked):
1. **opensquilla** (opensquilla-b1, safety-hole) -- Injection prose overclaim: tool results are never screened or wrapped, and the refusal path has no producer `_(README.md:702)_`
2. **bitfun** (bitfun-s7, safety-hole) -- no screening, no untrusted-content labeling of tool results, and the only sanitizer targets model format bleed, not adversarial content `_(src/crates/assembly/core/src/agentic/agents/prompt_builder/prompt_builder_impl.rs:398-424)_`
3. **gemini-cli** (gemini-cli-s7, nuance) -- Deterministic injection layer is thin and the LLM classifier is off by default: conseca enableConseca ?? false, env redaction opt-in outside GitHub `_(packages/core/src/config/config.ts:1325,1356-1358)_`

**DESIGN TENSION.** screen the content vs structure the wire -- the brief's named pair. Screening pole: gptme-7 (severity-tiered screening + [UNTRUSTED] marking), san-5 (write-time scan of self-learned memory entries and skill bodies -- the only place prompt poisoning can be caught before it persists), waveloom-11 (five-layer tool-output pipeline with boundary tags and escalation), openlumara-5 (six-layer regex/Unicode + random-delimiter envelopes), deepseek-reasonix-s8 (second-model screen, explicitly advisory, never blocks). The screening evidence averages med-confidence, and mistral-vibe-s5 shows the structural weakness: tagging that wraps only harness-authored advisories while raw file bodies and bash output ride unframed is spoofable by construction. The structural pole lives outside this concept: hotdog-1's marker-alias-mangling (per-session CSPRNG aliases; the model never observes real marker bytes, so quoting the tag matches nothing) and grok-build-s9's authority-boundaries-instead doctrine. Nine of 14 subjects have no layer at all, and opensquilla-b1 is the overclaim variant. Honest read: the crowded side screens; the best-evidenced single mechanism in the space is on the other side.

**HOTDOG POSITION.** absent from this concept -- our side of the tension is marker-alias-mangling (hotdog-1, unique, in the appendix): per-session CSPRNG aliases make the screening argument moot at the wire. If the cluster were scored across concepts, hotdog-1 would be the strongest single mechanism in it; inside this concept id, absent.

### crash-recovery-posture -- 13 subjects / 13 findings
kinds: unique 1, portable 9, nuance 3

**Best three implementations** (evidence-ranked):
1. **hermes-agent** (hermes-agent-c4, unique) -- Crash/liveness watchdog ladder covers every phase including pre-event-loop startup deadlock: faulthandler dumps, exit-75 for s6 respawn, lifecycle `_(hermes_startup_watchdog.py:1-27)_`
2. **roo-code** (roo-code-c8, portable) -- classified exceptions, expected-control-flow filtering, flush-then-exit shutdown, signal-only park mode, and lock-protected per-task JSON resume `_(apps/cli/src/commands/cli/run.ts:343)_`
3. **codewhale** (codewhale-m1, portable) -- process panic hook with crash dumps and terminal restore, fatal-signal guard, per-turn catch_unwind (36 sites), stall watchdog records `_(crates/tui/src/lib.rs:1810-1868)_`

**DESIGN TENSION.** contain-and-continue vs record-and-tell. Containment pole (Rust-heavy): codewhale-m1 (panic hook installed before arg parsing, terminal restore, per-turn catch_unwind at 36 sites), zeroclaw (catch_unwind around every hook phase, zero unwraps in the loop's production region, explicit stack sizing), goose-c7 (main on an 8MB-stack thread mapping panics to errors, WAL + busy_timeout), crush-6. Truth-on-resume pole: grok-build-c8 (dead-process turns get a distinct 'interrupted' stop reason surfaced to user AND model -- the model is told what it woke into), hermes-agent-c4 (watchdog ladder covering the pre-event-loop deadlock after a proven 30-hour hang), opensquilla-e8 (orphaned approvals expire, goals require explicit user resume). Evidence favors containment AT boundaries plus honest interrupted-state on resume -- hotdog's owner-release-on-all-exit-paths and lane reclaim converge here.

**HOTDOG POSITION.** absent.

### god-file-host-wiring -- 13 subjects / 13 findings
kinds: anti-pattern 13

**Best three implementations** (evidence-ranked):
1. **orca-agent** (orca-agent-c1, anti-pattern) -- tests start :23301) fusing host, supervisor, thread actor and hosted-operation runner in one file with 125 impl blocks `_(crates/orca-runtime/src/runtime_host.rs:4208)_`
2. **oh-my-pi** (oh-my-pi-c2, anti-pattern) -- AgentSession is a 12,181-LOC / ~328-method decision hub fusing mode state, advisors, cache keys, queues, extensions `_(packages/coding-agent/src/session/agent-session.ts:652)_`
3. **zeroclaw** (zeroclaw-c2, anti-pattern) -- channels orchestrator is a 50.8k-LOC production monolith fusing routing, delivery, interrupts, ingress frontier, and cost `_(crates/zeroclaw-channels/src/orchestrator/mod.rs = 52,741 lines total,)_`

**HOTDOG POSITION.** absent.

### scheduled-agent-runs -- 12 subjects / 12 findings
kinds: unique 2, portable 7, nuance 3

**Best three implementations** (evidence-ranked):
1. **hermes-agent** (hermes-agent-e7, unique) -- attempt recorded before dispatch, crashed runs recovered to unknown-with-no-retry, dead-owner claims reclaimed on a throttle, crash delivery `_(cron/scheduler.py:4087)_`
2. **vtcode** (vtcode-e6, portable) -- cron/interval/one-shot TOML records, events.jsonl, launchd/systemd service install; background subagents persist atomically and reap descendants `_(crates/codegen/vtcode-core/src/scheduler/mod.rs:108-128)_`
3. **aeon** (aeon-5, nuance) -- Scheduling as the whole architecture: debt-ledger cron catch-up designed around GitHub delivering ~10% of ticks `_(.github/workflows/scheduler.yml:5-16,63-72)_`

**DESIGN TENSION.** Durability is the whole argument here, and it converged: hermes-agent-e7 (attempt recorded BEFORE dispatch; crashed runs recovered to unknown-with-NO-retry; dead-owner claims reclaimed), zeroclaw-e7 (durable sqlite cron with an at-least-once outbox carrying persisted idempotency keys), cline-5 (sqlite-backed durable cron), vtcode-e6 (real daemon plane: launchd/systemd service install), qwen-code-e8, goose-e4 (stale running flags reset at startup), letta-code-e6 (atomic flush-to-temp rename + cross-process owner lock). aeon-5 is the opposite extreme -- the scheduler IS the product (debt-ledger catch-up designed around GitHub delivering ~10% of ticks). hotdog: no scheduler (absent).

**HOTDOG POSITION.** absent.

### overflow-degradation-ladder -- 11 subjects / 11 findings
kinds: portable 7, nuance 3, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **agentty** (agentty-5, portable) -- Reversible wire-only soft-trim under compaction: transcript never mutates, so overflow degrades to truncation instead of wedging the agent `_(src/runtime/app/cmd_factory.cpp:382-460)_`
2. **nausicaa-harness** (nausicaa-harness-6, portable) -- Context budget overflow degrades in fixed steps - drop compaction, then drop skill catalog and its paired schema atomically - never fails the turn `_(src/runtime/main-loop.ts:742-749)_`
3. **gptme** (gptme-b5, portable) -- Overflow retry is gated on a measured provider-side shrink: no-op compaction aborts the retry instead of re-billing the same overflow `_(gptme/chat.py:865-872)_`

**DESIGN TENSION.** what the rescue ladder is allowed to call, and who decides the rung. Agreement: LLM-free rungs first -- the ladder fires precisely on a backend that just rejected a conversation-sized request. Distinctive evidence: kolega-code-7 (the compaction REQUEST itself gets a descending budget ladder -- someone thought about overflow-during-overflow), agentty-5 (reversible wire-only soft-trim; the transcript never mutates, so overflow degrades to truncation instead of wedging the session), gptme-b5 (retry gated on a measured provider-side shrink -- a no-op compaction cannot re-bill the same overflow), nausicaa-harness-6 (drop compaction, then drop the skill catalog AND its paired schema atomically; never fail the turn), grok-cli-5 (halve keep-recent budget to a 4k floor). hotdog-9's misclassification guard (trim-only, and blunt drop fires only if canCompact passes, so a misread overflow cannot eat a young session) converges and adds a novel rung-selection rule; below the top three on evidence.

**HOTDOG POSITION.** hotdog-9: LLM-free rescue with the canCompact misclassification guard. Below the listed three on test weight; the rung-selection second-guess rule (the estimator that failed may not silently win) is a genuine add to the cluster. Concept-level competitive, evidence tier below.

### quality-gates-unwired -- 11 subjects / 13 findings
kinds: anti-pattern 13

**Best three implementations** (evidence-ranked):
1. **claw-code-agent** (claw-code-agent-b5, anti-pattern) -- Zero CI: no workflow runs the 1,233 claimed tests or the 19 benchmark suites `_(.github/workflows absent (ls: NO-GITHUB-WORKFLOWS))_`
2. **zeroclaw** (zeroclaw-b2, anti-pattern) -- Five fuzz targets exist but no workflow ever runs them; three of five fuzz serde itself `_(fuzz/fuzz_targets/ (fuzz_command_validation.rs, fuzz_config_parse.rs, )_`
3. **vtcode** (vtcode-c4, anti-pattern) -- Invariants declared 'mechanically enforced, caught by CI' are warn-only jobs that cannot fail the build `_(docs/harness/ARCHITECTURAL_INVARIANTS.md:3-4)_`

**DESIGN TENSION.** Failure census: gates that cannot fail the build -- invariants declared 'mechanically enforced, caught by CI' that are warn-only jobs (vtcode-c4), a test command excluding a nonexistent crate (amazon-q-12), pytest flags filtering away most of the suite (cursor-agent-6), '|| true' lint (qqcode-6), fuzz targets no workflow runs (zeroclaw-b2/s10), a dead eval skeleton whose own asset-detect step always misses (grinta-b2). Counter-examples the corpus owns: jcode-s7 (fetched-and-forced cohorts + grep-the-skip-message-or-fail) and the broken-as-configured precedent -- dock only when a gate silently skips real tests or runs red-and-ignored.

**HOTDOG POSITION.** absent.

### architecture-contract-test -- 11 subjects / 12 findings
kinds: unique 2, portable 7, nuance 3

**Best three implementations** (evidence-ranked):
1. **kode-cli** (kode-cli-8, portable) -- CI-enforced zone DAG with a reverse-edge ratchet that may shrink but never grow `_(scripts/check-architecture.mjs:12-40)_`
2. **openhands** (openhands-3, unique) -- CI-executed lint-as-architecture: a unit test bans ad-hoc HTTP to the backend outside an allowlist `_(src/api/no-direct-agent-server-calls.test.ts:32-40)_`
3. **kolkrabbi** (kolkrabbi-7, portable) -- layer import rules, engine-purity ('the engine touches no OS'), naming, plus ~14 named CI gates; knownViolations fails on both add AND un-removed-fix `_(internal/arch/layers.go:1-32)_`

**DESIGN TENSION.** Consensus with a maturity ladder: boundaries-as-CI-data is solved; the tell is what the ratchet does over time. Top rung: kolkrabbi-7/b1 (ratchet PAID TO ZERO -- knownViolations empty AND the test fails on fixed-but-still-listed entries, plus decay on unused third-party allowances), deepseek-reasonix-c2 (eight in-house AST linters for what the Go compiler cannot check inside a package), bitfun-c1 (six-layer machine-enforced layout incl. public-API allowlists and a TUI legacy ratchet), letta-code-c6 (god-files frozen at exact current LOC so they can only shrink). The rot-against-the-ratchet detail separates leaders from decoration.

**HOTDOG POSITION.** absent.

### lazy-skill-loading -- 11 subjects / 11 findings
kinds: portable 9, nuance 2

**Best three implementations** (evidence-ranked):
1. **san** (san-7, portable) -- Skills and subagent definitions parse metadata-only at startup; prompt bodies load on first use `_(internal/subagent/lazy_loading_test.go:44-47)_`
2. **qqcode** (qqcode-7, portable) -- SkillManager discovers SKILL.md frontmatter across project/user search paths and injects only an index; bodies load on demand via the skill tool `_(vibe/core/skills/manager.py:50-226)_`
3. **3code** (3code-9, portable) -- system prompt carries only a one-bullet-per-skill catalog, the model reads a skill file with the normal read tool when the task smells like it `_(src/threecode/prompts.nim:1145)_`

**DESIGN TENSION.** Converged design with a budget refinement ladder: name+description in prompt, body on demand -- 3code-9 (skills priced at zero when unused, 'deliberately no MCP for skills'), nausicaa-harness-7 (immutable metadata snapshots with changed-during-load rejection -- TOCTOU on a skill body), zap-coding-agent-3 (rank-and-truncate to a per-turn skill_token_budget with effective cost shown), kolega-code-5 (catalog budgeted as % of window with a truncation report instead of silent overflow), forge-norvialabs-9 (agentskills.io discovery/activation/execution). No tension.

**HOTDOG POSITION.** absent.

### workspace-path-containment -- 10 subjects / 11 findings
kinds: portable 5, nuance 1, safety-hole 5

**Best three implementations** (evidence-ranked):
1. **mini-kode** (mini-kode-1, safety-hole) -- FS grant containment via resolve()+startsWith admits sibling-prefix paths -- grant for /home/user authorizes /home/user-project, contradicting the `_(src/permissions/pathChecker.ts:53)_`
2. **kimi-cli** (kimi-cli-11, portable) -- Canonicalized workspace containment on read/glob with an explicit /add-dir escape hatch persisted in session state, and a distinct approval action `_(src/kimi_cli/tools/file/read.py:86-94)_`
3. **mocode** (mocode-5, portable) -- realpath-based jail incl. the new-file nearest-existing-ancestor case, centrally enforced `_(src/sandbox/jail.ts:36-62)_`

**DESIGN TENSION.** containment that survives symlinks and sibling-prefix tricks vs startsWith theater. Correct pole: mocode-5 (realpath jail incl. the new-file nearest-existing-ancestor case, centrally enforced), kimi-cli-11 (canonicalized containment + a DISTINCT approval action for out-of-workspace edits), grok-build-s8 (read-deny kernel-enforced, cross-platform glob parity validated on both backends), keen-code-5 (blanket $HOME dotfile block, Lstat symlink rejection). The bug class is mechanical and repeated: abspath().startsWith() admits /work-evil under /work (claii-1, mini-kode-1/b1, groq-code-cli-2). hotdog's resolveSafe (ancestor realpath walk, dangling-symlink chain, denylist re-checked against the REAL path) is top-pole mechanics -- evidence visible only inside hotdog-8, never submitted standalone.

**HOTDOG POSITION.** absent as a record; resolveSafe (ancestor realpath walk, dangling-symlink chains, denylist re-checked against the real path) is evidenced only inside hotdog-8. Unclaimed credit.

### actionable-errors -- 10 subjects / 10 findings
kinds: portable 9, nuance 1

**Best three implementations** (evidence-ranked):
1. **opensquilla** (actionable-errors, portable) -- User-facing error refs join bug reports to durable sanitized diagnostics: one turn_errors row per failed turn with the error_id shown in chat `_(src/opensquilla/persistence/migrator.py:532,871)_`
2. **workground2** (workground2-e14, portable) -- Actionable, i18n'd errors with named env vars and fix commands, fail-fast key preflight, and a doctor --json diagnostics command `_(internal/provider/provider.go:668-678)_`
3. **qwen-code** (qwen-code-e11, portable) -- Error paths are engineered for actionability: dedup marker, named fix flags, key-format hints `_(packages/cli/src/utils/errors.ts:22-38)_`

**HOTDOG POSITION.** absent.

### durable-queue -- 9 subjects / 10 findings
kinds: portable 2, nuance 2, anti-pattern 6

**Best three implementations** (evidence-ranked):
1. **bitfun** (bitfun-e6, portable) -- SHA-256-fingerprinted host message queue with explicit no-silent-eviction admission, persistent dispatch plane with a `_(src/crates/assembly/core/src/agentic/coordination/host_message_queue.rs:11-30)_`
2. **letta-code** (letta-code-e7, anti-pattern) -- The message/task-notification/cron/approval queue that all three turn hosts share is purely in-process: coalescing, soft/hard caps and user-interrupt `_(src/queue/queue-runtime.ts:186)_`
3. **code** (code-e6, anti-pattern) -- no queue command anywhere in the fork CLI, the Auto Drive transcript is in-memory by design, and crash recovery is an in-process transient-retry `_(cli/src/proto.rs:105,137)_`

**DESIGN TENSION.** durable queue as infrastructure vs in-memory convenience -- the brief's named pair, verbatim. Durable pole: bitfun-e6 (SHA-256 fingerprinted host message queue, no-silent-eviction admission, persistent dispatch plane with a controller-crash-cannot-skip-events invariant) and opensquilla-e7 (gateway task runtime with reservations, overflow policy, quiesce). In-memory pole: letta-code-e7 (the queue all three turn hosts share is purely in-process; a crash silently drops queued input), mistral-vibe-e6 (enqueue/steer/replace/resume with idempotency keys -- on a queue that dies with the process), roo-code-e7 (98-LOC array), gemini-cli-e9 (UI useState convenience). Only 2 of 9 subjects are durable, but both are high-confidence designed-invariant statements, while the in-memory side is uniformly filed as anti-pattern: evidence favors durability, the field mostly hasn't built it. hotdog: workflows journal; the steering queue and plain subagents do not survive process death -- partial at best.

**HOTDOG POSITION.** partial: the workflow engine journals runs (hotdog-4) but the steering queue and plain subagents die with the process. Absent as a queue.

### validated-finish-gate -- 8 subjects / 9 findings
kinds: unique 1, portable 5, nuance 3

**Best three implementations** (evidence-ranked):
1. **atomic-agent** (atomic-agent-1, portable) -- a final reply reporting a check ('tests pass', 'verified', 'ran node --check on all files') is matched against the turn's actual tool calls; a claim `_(src/agent/claim-evidence.ts:1-31,64,104,148)_`
2. **molt** (molt-1, portable) -- task criteria are Object.freeze-copied, sealed by hash, and journalled BEFORE the first request, then re-run against disk on every 'done'; refusals `_(src/engine.ts:3608-3637)_`
3. **octomind** (octomind-9, portable) -- independent LLM verifier judges END STATE from typed evidence blocks (recorded actions with mut/read shape, ground-truth diff, read-back rounds) `_(src/supervisor/gate.rs:15-19,27-75,660)_`

**DESIGN TENSION.** accept the claim with machine checks vs trust the narrative. Machine-gate pole: molt-1 (task criteria Object.freeze-copied, sealed by hash, journalled BEFORE the first request, then re-run against disk on every 'done'; refusal text is part of the design), octomind-9/b3 (independent verifier judges END STATE from typed evidence blocks -- recorded actions, ground-truth diff, read-back rounds -- narrative treated as untrusted), atomic-agent-1 (a reply claiming 'tests pass' must match the turn's actual tool calls or the reply is held back), minicode-9 (no-verify runs report 'unverified' as a first-class state; failed verify demotes to 'blocked'), ouroboros-8 (closed set of typed acceptance reasons, 'none derives from model prose'). hotdog-5 converges with file-freshness + verdict-file + judge-node routing -- freshness (stale outputs from a prior run fail the gate) is the distinctive claim; ranked below the top three on evidence, competitive on concept.

**HOTDOG POSITION.** hotdog-5: file-freshness + verdict file + judge nodes with untrusted-part splicing. Rank ~#6/9: below molt/octomind/atomic-agent (seal-and-recheck and end-state verifiers carry more evidence), above the rest; the freshness claim (stale prior-run outputs fail the gate) is our distinctive contribution. Second of the two clusters where hotdog is plausibly competitive.

### memory-files -- 8 subjects / 8 findings
kinds: unique 1, portable 3, nuance 4

**Best three implementations** (evidence-ranked):
1. **ipsupport-code** (ipsupport-code-3, portable) -- Reflection-distilled pitfalls injected into tool-error results on error-pattern match, with a distinct 'avoid' kind for dead ends and generic-pattern `_(internal/knowledge/pitfall.go:12-44)_`
2. **ra-aid** (ra-aid-6, portable) -- Dedicated GC agents garbage-collect accumulated research notes, key facts, and key snippets `_(ra_aid/tools/memory.py:230-395)_`
3. **atomic-agent** (atomic-agent-12, portable [conf med]) -- reflection fires on turn segmentation, a consolidator job merges notes, lessons/procedures are pointer-indexed sections outside the cache prefix, the `_(src/memory/reflection/; src/memory/consolidator/consolidator-job.ts)_`

**DESIGN TENSION.** No tension -- scattered maturity: reflection-distilled pitfalls injected into tool-error results at the point of failure (ipsupport-code-3), dedicated GC agents for accumulated notes (ra-aid-6), reviewable proposal stores (nanocoder-5), sqlite-FTS with a write-gate (mimo-code-3), atomic-agent-12's memory fabric with its own eval program. Convergence count overstates coherence: everyone tried something, nobody converged.

**HOTDOG POSITION.** absent.

### network-egress-approval -- 8 subjects / 8 findings
kinds: portable 3, nuance 2, anti-pattern 1, safety-hole 2

**Best three implementations** (evidence-ranked):
1. **forge-norvialabs** (forge-norvialabs-2, portable) -- nothing pre-allowed (crates.io/npmjs/pypi/github asserted 403 in tests), only PERSONAL permissions.toml host rules open it -- repo-committed allows `_(crates/forge-core/src/permission.rs:36-48)_`
2. **codex** (codex-9, portable) -- Network egress is sandboxed by default with a separate seatbelt network policy and approval flow `_(execpolicy/src/rule.rs:149)_`
3. **grok-build** (grok-build-s6, safety-hole) -- child-network blocking is a Linux-only seccomp filter (macOS no-op, no Windows), and the richer website-policy module is honestly-labeled unenforced `_(crates/codegen/xai-grok-sandbox/src/child_net.rs:178-191)_`

**DESIGN TENSION.** deny-by-default egress proxy vs open network vs refuse-to-fake. Proxy pole: forge-norvialabs-2 (nothing pre-allowed -- crates.io/npmjs/pypi/github asserted 403 in tests; repo-committed allow rules never loosen, proxy failure leaves network OFF), codex-9 (egress sandboxed by default with a separate network approval flow), orca-agent-s8 (bounded in-runtime proxy whose block reports, not output text, drive escalation). Anti-fabrication pole: vtcode-s8 (allowlists are unenforceable under its sandbox, so the design REJECTS the policy rather than faking enforcement). Open pole: 3code-b2 (FS fence on, network ships 'allow *'), workground2-s11 (write-only jail, egress on), grok-build-s6 (Linux-only seccomp child-net filter, macOS no-op footnoted). Evidence clusters on deny-by-default proxy wherever egress is enforced at all.

**HOTDOG POSITION.** absent.

### headless-contract -- 7 subjects / 7 findings
kinds: unique 1, portable 5, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **letta-code** (letta-code-e9, unique) -- The headless stream-json contract deliberately converges on Claude Code/Codex wire conventions (timestamp field/format) so downstream normalizers `_(src/stream-json-writer.ts:1-24)_`
2. **opencode** (opencode-e10, portable) -- run --format json raw event stream, --continue/--session/--fork, serve/attach, export with secret redaction, import from share URLs; generated TS `_(packages/opencode/src/cli/cmd/run.ts:14)_`
3. **roo-code** (roo-code-e11, portable) -- text|json|stream-json output, a stdin control protocol with requestIds and queue-awareness, schemas shared in @roo-code/types, emitter flush `_(apps/cli/src/types/json-events.ts:15-30)_`

**DESIGN TENSION.** converge on rival wire conventions vs speak your own protocol. Convergence pole: letta-code-e9 (stream-json deliberately matches Claude Code/Codex field names 'so downstream normalizers can treat all three CLIs uniformly'), roo-code-e11 (typed NDJSON + stdin control protocol with requestIds, shared schemas), opencode-e10 (contract with resume/fork/export-with-redaction + generated SDKs), grok-build-e14 (four output formats, script-safe resume), bitfun-e11 (codex-grade flags + rival-harness import), and the headless-rpc-protocol cluster (pi-9's documented RPC, maki-11's Claude-Code-output-compatible stream-json, codewhale-e10's CI-published SDK). hotdog: '-p/--json/--json-schema' via a synthetic terminal tool -- a usable one-shot, no contract-level stream protocol, own websocket surface undocumented (interop 5).

**HOTDOG POSITION.** absent at contract level: -p/--json/--json-schema exists (structured output via synthetic tool) but no stream protocol; own ws surface undocumented.

### context-overflow-failopen -- 7 subjects / 7 findings
kinds: anti-pattern 7

**Best three implementations** (evidence-ranked):
1. **cursor-agent** (cursor-agent-4, anti-pattern) -- No compaction at all: unbounded history, overflow caught as a user-facing error string; trim_context_history trims display lists, not model context `_(cursor_agent_tools/base.py:82)_`
2. **devon** (devon-5, anti-pattern) -- full chat history resent every turn (openai path) or flattened into a per-request transcript rebuild (anthropic path), no compaction, counting, or `_(devon_agent/agents/conversational_agent.py:151,169)_`
3. **groq-code-cli** (groq-code-cli-4, anti-pattern) -- Unbounded message history; overflow surfaces as generic retry-forever error overlay `_(src/core/agent.ts:266-541)_`

**DESIGN TENSION.** Failure census: seven subjects where overflow meets an error string or blind retry and nothing compacts (trae-agent-4 retry-forever on append-only history, cursor-agent-4, devon-5, groq-code-cli-4, open-codex-6, codel-4's 'ask the user' with summarization as TODO, auto-code-rover-b2's task death). No implementer side inside the concept -- the counterweight is the whole compaction body of this document; the cluster is a D-tier architecture marker, not a design debate.

**HOTDOG POSITION.** absent.

### eval-harness-outside-ci -- 7 subjects / 7 findings
kinds: nuance 1, anti-pattern 6

**Best three implementations** (evidence-ranked):
1. **vtcode** (vtcode-s9, nuance) -- Fuzzing lives outside CI; the in-CI 'tool-eval' is a deterministic smoke script (compile + rg presence), not a model eval `_(.github/workflows/tool-eval.yml:63-71)_`
2. **jazz** (jazz-b5, anti-pattern) -- Full A/B eval harness never invoked by any CI workflow `_(.github/workflows/ci.yml:176-179)_`
3. **agentless** (agentless-4, anti-pattern) -- Entire evidence base is SWE-bench runs reproducible only by manual README steps; zero CI and zero self-tests, evidence culture lives in `_(README.md:74-78)_`

**HOTDOG POSITION.** absent.

### interop-surface-breadth -- 7 subjects / 7 findings
kinds: unique 1, portable 1, nuance 5

**Best three implementations** (evidence-ranked):
1. **oh-my-pi** (oh-my-pi-e9, portable) -- MCP client with OAuth+Smithery, bidirectional ACP, version-negotiated RPC with a separate Python package, published npm SDK, headless json, native `_(mcp/oauth-flow.ts:1-20)_`
2. **aider** (aider-e9, nuance) -- documented+tested Python API, streamlit GUI, IDE-comment watch mode, clipboard mode, voice, and a 27-provider metadata matrix `_(aider/website/docs/scripting.md:63-64,92)_`
3. **continue** (continue-e7, nuance) -- MCP client with OAuth, 67 provider adapters, three IDE hosts plus standalone binary, first-class headless -p/--format json; but no ACP, no MCP server `_(MCPConnection.ts:88)_`

**HOTDOG POSITION.** absent.

### prompt-cache-discipline -- 7 subjects / 7 findings
kinds: portable 5, nuance 2

**Best three implementations** (evidence-ranked):
1. **grok-build** (grok-build-e5, portable) -- Prompt-cache keys are threaded end-to-end with a backend-honesty invariant that is tested at the wire `_(xai-grok-sampling-types/src/conversation/responses.rs:136-139)_`
2. **oh-my-pi** (oh-my-pi-e3, portable) -- warm-prefix rewrite guards, TTL-aware idle flush, fork-inherited cache keys, and zero-tool_choice steering design `_(packages/coding-agent/src/session/session-maintenance.ts:301-307)_`
3. **qwen-code** (qwen-code-e3, portable) -- Provider-specific cache-breakpoint placement, global-scope/1h-TTL probes, and cache-prefix-ordered context assembly `_(packages/core/src/core/anthropicContentGenerator/converter.ts:81-84)_`

**DESIGN TENSION.** end-to-end ownership vs pass-through reporting. Owned: grok-build-e5 (cache keys threaded end-to-end with a backend-honesty invariant TESTED at the wire), qwen-code-e3 (provider-specific placement + global-scope/1h TTL probes + cache-prefix-ordered context assembly), oh-my-pi-e3 (warm-prefix rewrite guards, TTL-aware idle flush, fork-inherited cache keys, zero-tool_choice steering design -- discipline as a first-class economic constraint), bitfun-e3 (generation-fenced writes, prompt_cache_key lineage). Pass-through: letta-code-e3 (zero client-side handling; nothing between harness and server). hotdog: deliberately none cloud-side; KV-warm placement is the local substitute.

**HOTDOG POSITION.** absent.

### steer-during-run -- 6 subjects / 6 findings
kinds: portable 4, nuance 2

**Best three implementations** (evidence-ranked):
1. **dexto** (dexto-m1, portable) -- Two-tier mid-task input queues (late-steer coalesced into the next model step, follow-ups only after natural stop) decided inside the loop against `_(turn-executor.ts:1122-1160)_`
2. **codebuff** (codebuff-9, portable) -- Steering queue drained exactly at step boundaries, appending user prompts and forcing the turn to continue `_(packages/agent-runtime/src/run-agent-step.ts:784-790)_`
3. **gptme** (gptme-12, portable) -- Durable file-based steering queue with a closed-sentinel protocol whose write-ordering races against concurrent subagent_steer() are explicitly `_(gptme/chat.py:230-250,310-316,407-431)_`

**DESIGN TENSION.** drain-point discipline and queue durability. Near-consensus on placement: drain between LLM calls, never between assistant(tool_calls) and tool results (hotdog, codebuff-9's step-boundary contract). Durability splits: opensquilla's at-least-once claim/apply/reclaim contract and gptme-12's file-based queue with write-ordering races argued in comments vs in-memory queues that die with the process (roo-code-e7; hotdog's own steering queue -- partial, absent). dexto-m1's two-tier decision inside the loop (late-steer coalesced into the next model step, follow-ups only after natural stop, DB-backed stores) is the sharpest mechanism at the durability end.

**HOTDOG POSITION.** our drain discipline (drain between LLM calls, never mid tool-pair) matches the consensus; the queue is in-memory -- the durability half is absent.

### prompt-cache-warming -- 6 subjects / 8 findings
kinds: unique 2, portable 1, nuance 4, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **octomind** (octomind-1, portable) -- max_tokens=1 ping on a frozen conversation snapshot resets provider cache TTL, with ping costs harvested and folded exactly-once into session cost `_(src/session/cache_keepalive.rs:17,51,198)_`
2. **opensquilla** (opensquilla-e3, unique) -- Active cache discipline: proven-prefix keepalive replay + cache-break detection wired into the loop `_(src/opensquilla/gateway/prompt_cache_keepalive.py:53)_`
3. **kilocode** (kilocode-e3, anti-pattern) -- No active cache warming or cache-hit observability `_(packages/opencode/src/kilocode/watcher.ts:55)_`

**DESIGN TENSION.** actively keep the cache warm vs let TTL die. Active pole: octomind-1 (max_tokens=1 keepalive on a frozen snapshot resets provider TTL; ping costs harvested and folded exactly-once into session cost) refined by octomind-b2 (keepalive REFUSED for providers with no observable refresh primitive), pi-3/b1 (warming gated on expected dollar savings, not TTL alone), opensquilla-e3 (proven-prefix keepalive replay + cache-break detection wired into the loop). Against a passive majority (gemini-cli-e5: one comment-level prefix rule) and kilocode-e3's explicit none (anti-pattern). hotdog's cache-warm-model-placement (loaded-model placement beats all candidate orderings; warm follow-up retries; lane slots held across mid-turn question waits so a swap cannot thrash the cache the user returns to) is the local-backend analogue -- unique in the corpus, counted under its own concept, absent from this one.

**HOTDOG POSITION.** absent.

### classifier-approval-ladder -- 6 subjects / 7 findings
kinds: portable 6, nuance 1

**Best three implementations** (evidence-ranked):
1. **deepagents** (deepagents-1, portable) -- LLM classifier batch-reviews tool calls behind deterministic allow/deny, failing to a human on unavailability with a latched bad spec `_(libs/code/deepagents_code/auto_mode.py:467-491)_`
2. **jazz** (jazz-5, portable) -- Single-token LLM command-risk classifier for execute_command with 8s/16-token budget, fail-closed to high-risk, overridable by a plugin hook that is `_(packages/core/src/agent/tools/command-risk.ts:29-45,49-68,73-82)_`
3. **kode-cli** (kode-cli-3, portable) -- Regex data-loss classifier promotes suspicious Bash commands to a quick-model LLM reviewer; gate failure fails closed only when the command would run `_(packages/tools/src/tools/system/BashTool/call.tsx:131-203)_`

**DESIGN TENSION.** Consensus shape: deterministic allow/deny first, LLM classifier in the middle, human last, fail-closed on classifier loss -- deepagents-1 (fails to a human on unavailability with a latched bad spec), jazz-5 (8s/16-token budget, fail-closed to high-risk, plugin hook denied escalation authority), kode-cli-3 (gate fails closed ONLY when the command would run unsandboxed -- an explicit economics-based fail-open), vtcode-s4 (precedence hard-coded so saved approvals, cache, classifier, and full-auto all LOSE to a risk flag -- the precedence detail everyone should copy). hotdog: no classifier (absent; bail-based tokenizer covers the bottom rung).

**HOTDOG POSITION.** absent.

### auto-continuation-goals -- 6 subjects / 6 findings
kinds: unique 2, nuance 4

**Best three implementations** (evidence-ranked):
1. **claurst** (claurst-9, unique) -- /goal multi-turn objectives with budget guards plus 'ultracode' keyword that swaps top effort AND injects a plan-delegate-integrate-verify fanout `_(core/src/effort.rs:273-405)_`
2. **codex-infinity** (codex-infinity-2, unique) -- run-forever layer: persisted goals + end-of-turn arc-monitor that steers or asks the user `_(codex-infinity/codex-rs/core/src/arc_monitor.rs:15-25)_`
3. **ipsupport-code** (ipsupport-code-7, nuance) -- judge verdicts of 'unclear' re-feed rather than end, and every give-up path (turn failure, stuck-stop, step exhaustion) gets a judge pass with a `_(internal/agent/agent.go:1013-1044,1221-1237,1287-1300,1767-1810)_`

**DESIGN TENSION.** DESIGN TENSION: what ends a goal. Explicit-terminal pole: ipsupport-code-7 (goal ends ONLY on explicit DONE, TTL expiry, or the user; judge verdicts of 'unclear' re-feed rather than end -- every give-up path gets a judge pass). Judge-terminates pole: deepagents-7 (grader sub-agent re-enters until satisfied/failed/max_iterations). The disagreement is whether model judgment may end autonomy; the explicit-terminal side is the fail-closed reading and has the more defensive machinery. hotdog: /loop with maxLoops -- naive rung, absent from this concept's records.

**HOTDOG POSITION.** absent.

### fail-closed-default-decision -- 6 subjects / 6 findings
kinds: portable 6

**Best three implementations** (evidence-ranked):
1. **bitfun** (bitfun-s5, portable) -- unmatched rules ask, empty resources ask, closed reply channel cancels execution, audit-write failure aborts request registration, unknown persisted `_(src/crates/contracts/product-domains/src/tool_permissions.rs:728-748)_`
2. **mistral-vibe** (mistral-vibe-s3, portable) -- Uncertainty lands on deny or a prompt everywhere on the enforcement path: no bound approval UI returns an immediate NO, programmatic mode never `_(vibe/core/agent_loop/_request_broker.py:49)_`
3. **gemini-cli** (gemini-cli-s4, portable) -- Default decision is ASK_USER interactive / DENY non-interactive, and shell-parse failure falls back to that default instead of allow `_(packages/core/src/policy/policy-engine.ts:299-301)_`

**DESIGN TENSION.** Consensus pole with zero defectors among implementers: uncertainty lands on deny-or-human on every degraded path -- gemini-cli-s4 (parse failure falls to default DENY under YOLO), bitfun-s5 (unmatched asks, closed reply channel cancels, audit-write failure aborts registration), grok-build-s2 (prompt transport errors -> Reject; persisted 'always allow' fail-safe gated), deepseek-reasonix-s6, mistral-vibe-s3, zeroclaw-s6 (provenance-aware). The autonomous-confirm-failopen cluster is what the absence of this shape produces. hotdog's gate matrix is corpus-class on this exact point -- off by default notwithstanding.

**HOTDOG POSITION.** absent.

### interop-matrix -- 6 subjects / 6 findings
kinds: portable 5, nuance 1

**Best three implementations** (evidence-ranked):
1. **vtcode** (vtcode-e8, portable) -- ACP agent AND client, MCP client with OAuth, A2A client + feature-gated server, headless ThreadEvent JSONL, two IDE extensions, WebMCP bridge `_(capabilities.rs:29-41)_`
2. **gemini-cli** (gemini-cli-e10, portable) -- MCP client+server, ACP as a first-class CLI mode with session resume, A2A server package, headless JSONL contract, embedding SDK in-tree `_(packages/vscode-ide-companion/src/ide-server.ts:14-15)_`
3. **mistral-vibe** (mistral-vibe-e8, portable) -- Full ACP server surface plus five-platform prebuilt vibe-acp binaries, headless -p/--output text|json|streaming, a GitHub Action, and an MCP client `_(vibe/acp/agent.py:452-1127)_`

**DESIGN TENSION.** The breadth scoreboard converges on a checklist (MCP client+server, ACP, headless JSONL, SDKs, IDE surfaces) -- gemini-cli-e10, qwen-code-e9, codewhale-e9, vtcode-e8 (ACP agent AND client, A2A, WebMCP), mistral-vibe-e8 (full ACP server + prebuilt binaries + GitHub Action). Tension is only breadth-vs-depth (see interop-ceilings). hotdog: below every entry here -- MCP client only, no server, no ACP, no SDK (interop 5).

**HOTDOG POSITION.** absent.

### mcp-client-only -- 5 subjects / 5 findings
kinds: nuance 3, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **jcode** (jcode-e8, anti-pattern) -- MCP is client-side and stdio-only: HTTP/SSE servers are recognized and skipped, and there is no mode exposing jcode as an MCP server `_(README.md:604)_`
2. **letta-code** (letta-code-e11, nuance) -- MCP support is client-side and real (stdio/SSE/streamable-HTTP + OAuth) with server-side tool management gated on backend capability, but the harness `_(src/mcp-client.ts:6-11)_`
3. **opencode** (opencode-e11, anti-pattern [conf med]) -- MCP is client-only: no server mode anywhere in the tree, and added servers are auto-stored enabled:true without a per-server handshake `_(acp/service.ts:17,)_`

**DESIGN TENSION.** The negative pole of interop: client-only MCP as a ceiling (opencode-e11, grok-build-e15, jcode-e8 stdio-only, letta-code-e11, zeroclaw-e10). The server-side counterweight lives in interop-matrix and three-transport-mcp-client (dexto-e9: all three official transports + a dual-transport server). hotdog: client-only, conformance-tested -- in this cluster, on the ceiling side with the rare mitigating test evidence.

**HOTDOG POSITION.** hotdog is in this posture: MCP client-only -- but conformance-tested (tests/conformance/mcp-conformance.test.ts), the one depth claim our interop-5 can make.

### turn-budget-accounting -- 5 subjects / 5 findings
kinds: portable 2, nuance 1, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **code** (code-e4, portable) -- escalating time-budget nudges, coordinator turn cap that hard-stops runaway runs, HARD_MESSAGE_LIMIT=320 drain with a global trimmed-items counter `_(code-rs/code-auto-drive-core/src/auto_coordinator.rs:340-366)_`
2. **codewhale** (codewhale-e6, portable) -- Budgets at every layer: workflow lifetime/concurrency caps, fast-fail BudgetSnapshot, per-turn wall-clock budget that is never reported as success `_(crates/workflow-js/src/lib.rs:82,89)_`
3. **neovate-code** (neovate-code-3, anti-pattern) -- Turn cap deflated by tool calls: turnsCount -= approvedToolUses.length makes tool-heavy loops effectively unbounded against max_turns `_(src/loop.ts:736-737)_`

**DESIGN TENSION.** budgets that count vs budgets that leak. Leaking pole: neovate-code-3 (turnsCount DECREMENTED by approved tool uses -- tool-heavy loops effectively unbounded against max_turns), openlumara-9 (no iteration cap at all), goose-e12 (turn-count budgets, never tokens or cost). Real-budget pole: code-e4 (escalating time-budget nudges, hard coordinator cap with user message, global trimmed-items counter), codewhale-e6 (fast-fail BudgetSnapshot; per-turn wall-clock budget that is NEVER reported as success). hotdog's engine banks runtime only while a turn actually runs (per-node accumulated cap) -- honest-pole mechanics, no findings record here; hotdog has no token budgets at all.

**HOTDOG POSITION.** absent: no token budgets on tasks; per-node accumulated-runtime caps (engine.ts:838-880) bank only running time -- honest-pole shape, no record here.

### autonomous-confirm-failopen -- 5 subjects / 6 findings
kinds: safety-hole 6

**Best three implementations** (evidence-ranked):
1. **zot** (zot-4, safety-hole) -- Yolo is the default: approval gate exists only behind --no-yolo and jail only behind opt-in `_(packages/agent/args.go:95-108)_`
2. **gptme** (gptme-11, safety-hole) -- Autonomous mode fail-open: no TOOL_CONFIRM hook registered means every tool auto-confirms unless guardrails are explicitly promoted to enforce `_(gptme/hooks/confirm.py:228-236)_`
3. **kode-cli** (kode-cli-5, safety-hole) -- YOLO is the factory default: fresh installs bypass all approval prompts; permission checks exist but must be opted into via --safe `_(packages/core/src/utils/permissionModeState.ts:7)_`

**DESIGN TENSION.** what headless means. Fail-to-deny pole: deepseek-reasonix-s6 (every non-interactive mode wires a denyPermissionApprover with an honest refusal reason 'so the model stops asking for a user who was never there'), gemini-cli-s4 (ASK_USER interactive / DENY non-interactive), mistral-vibe-s3 (no bound approval UI returns an immediate NO). Auto-approve pole: kode-cli-5 (yolo is the FACTORY default), zot-4, ob-1-3 (autopilot + sandbox off; the trust gate downgrades only an implicit autopilot), gptme-b2 (non-TTY stdin silently flips no_confirm -- filed as a hole even for gptme, the doctrine's 'silent flips docked' rule), and the latent-shape variant both workground2-s3 and deepseek-reasonix-s9 recorded: a Gate whose nil Approver resolves Ask to Allow. Evidence clusters on fail-to-deny; hotdog's user-gate (non-interactive -> deny, matrix-tested) has the right polarity and the wrong default.

**HOTDOG POSITION.** absent.

### acp-internal-contract -- 5 subjects / 5 findings
kinds: unique 1, portable 2, nuance 2

**Best three implementations** (evidence-ranked):
1. **prime-agent** (prime-agent-9, portable) -- pi fork adds both ACP mode and a full MCP client with safety-verified catalog `_(mcp-service-safety.test.ts:1-21,)_`
2. **grok-build** (grok-build-c2, portable) -- Every product surface is an ACP client to the same session actor; the TUI contains zero loop logic `_(xai-grok-shell/src/leader/mod.rs:1-40)_`
3. **nanocoder** (nanocoder-6, unique) -- VS Code plugin drives nanocoder over its own ACP server process `_(plugins/vscode/src/acp-client.ts, acp-process-manager.ts)_`

**HOTDOG POSITION.** absent.

### confine-or-refuse-startup -- 5 subjects / 5 findings
kinds: unique 2, portable 3

**Best three implementations** (evidence-ranked):
1. **orca-agent** (orca-agent-s2, portable) -- Single process-launch choke point (ExecutionBroker) rejects non-dangerous launches when enforcement is Unavailable/Advisory; RemoteSandbox refused `_(crates/orca-core/src/execution_broker.rs:212-262)_`
2. **agentty** (agentty-3, portable) -- Sandbox availability is a capability probe that runs the exact namespace the real command will use, not a `which` `_(src/tool/util/sandbox.cpp:66-98)_`
3. **grok-build** (grok-build-s5, unique) -- profiles that require protection exit(1) when enforcement cannot be established, and inside-bwrap identity is re-verified by mount inspection because `_(crates/codegen/xai-grok-shell/src/config/mod.rs:1442-1454)_`

**DESIGN TENSION.** degrade vs refuse. Refuse pole: forge-norvialabs-1 (on a host that cannot confine, the CLI REFUSES to start rather than run unprotected; shell deliberately NOT HITL-gated because kernel confinement, not approvals, is the boundary), grok-build-s5 (required protection profiles exit(1) when enforcement cannot be established; inside-bwrap identity re-verified by MOUNT INSPECTION because the env marker is spoofable), orca-agent-s2 (ExecutionBroker rejects launches when enforcement is Unavailable/Advisory; RemoteSandbox refused, not run locally), grinta-b5 (hard-fail when the jail binary is missing, all three platforms). Degrade pole: waveloom-13 (headless degrades to fully-unsandboxed auto-allow when bwrap is absent unless failIfUnavailable=true), sandbox-request-failopen (zeroclaw's NoopSandbox, codewhale's silent degrade). The amended doctrine's gate (i) -- refuse-to-run, never fall back to unprotected -- mechanically encodes the refuse pole, and the evidence agrees.

**HOTDOG POSITION.** absent.

### entry-path-loop-duplication -- 5 subjects / 5 findings
kinds: anti-pattern 5

**Best three implementations** (evidence-ranked):
1. **vtcode** (vtcode-c1, anti-pattern) -- Three live turn engines (interactive, headless, ACP) with agreement-by-comment instead of shared loop `_(src/agent/runloop/unified/turn/turn_loop.rs:790)_`
2. **gemini-cli** (gemini-cli-c2, anti-pattern) -- Six independent outer-loop drivers construct their own Scheduler and re-implement turn accounting `_(packages/cli/src/ui/hooks/useToolScheduler.ts:107,)_`
3. **continue** (continue-c2, anti-pattern) -- Full second loop, compaction, and permission stack re-implemented in the CLI against the GUI originals `_(extensions/cli/src/stream/streamChatResponse.ts:443)_`

**DESIGN TENSION.** Failure census driven by multi-surface pressure: six independent outer-loop drivers in gemini-cli, letta-code's six while(true)s across three hosts with a PROVEN semantic divergence, vtcode's three live engines with 'agreement-by-comment', continue's full second loop+compaction+permission stack. Paired with parallel-agent-stack-migration and dual-loop-migration the corpus writes the same lesson three times: a second entry path without one shared loop is a second product that rots.

**HOTDOG POSITION.** absent.

### foreign-session-import -- 5 subjects / 5 findings
kinds: unique 2, portable 3

**Best three implementations** (evidence-ranked):
1. **kilocode** (kilocode-m1, unique) -- Claude Code / Codex transcript import plus Claude config/skills/MCP migration with versioned receipt and ~35 typed refusal reasons `_(packages/opencode/src/kilocode/session-resume/index.ts:8-10)_`
2. **cline** (cline-b1, unique) -- Import-and-resume sessions from Claude Code, Codex, and opencode transcripts `_(sdk/packages/core/src/services/session-import/types.ts:5-14,)_`
3. **open-interpreter** (open-interpreter-b5, portable) -- External-agent session import with sha256-content idempotency ledger - convert rival transcripts to rollout items, never double-import `_(codex-rs/external-agent-sessions/src/detect.rs:16-105)_`

**DESIGN TENSION.** No contradiction, a growth edge with a safety ladder: kilocode-m1 (transcript import + config/skills/MCP migration with versioned receipt and ~35 typed refusal reasons), open-interpreter-b5 (sha256-content idempotency ledger, never double-import), grok-build-e16 (bounded, read-only, metadata-only listing), goose-c6 (format sniffing into native journal), cline-b1 (import-AND-resume). hotdog: absent.

**HOTDOG POSITION.** absent.

### lazy-tool-catalog -- 5 subjects / 5 findings
kinds: portable 3, nuance 2

**Best three implementations** (evidence-ranked):
1. **maki** (maki-8, portable) -- Deferred MCP tools hidden behind one tool_search catalog entry; a permitted call counts as loading, a denied one loads nothing `_(maki-agent/src/mcp/mod.rs:53,353-391)_`
2. **openlumara** (openlumara-1, portable) -- tools_load meta-tool keeps the tool array near-empty; AI-loaded tool sets persist per chat and self-heal via rejection messages `_(core/tool_loader.py:186-214,306-321,356-375)_`
3. **openharness** (openharness-8, portable) -- DeferredTool wraps tools with name+description-only prompts until ToolSearch or first call `_(src/DeferredTool.ts:1-33)_`

**HOTDOG POSITION.** absent.

### readme-capability-drift -- 5 subjects / 5 findings
kinds: anti-pattern 5

**Best three implementations** (evidence-ranked):
1. **goose** (goose-e10, anti-pattern) -- Docs describe background tool-pair summarization as default behavior; the code gates it behind GOOSE_TOOL_PAIR_SUMMARIZATION (default false) which is `_(documentation/docs/guides/sessions/smart-context-management.md:48-49)_`
2. **cursor-agent** (cursor-agent-5, anti-pattern) -- constraints.md documents summarization/pruning/backoff 'workarounds' that are not implemented `_(constraints.md:11)_`
3. **devon** (devon-9, anti-pattern) -- README license badge advertises Apache-2.0 while LICENSE is AGPL-3.0 `_(README.md:12)_`

**HOTDOG POSITION.** absent.

### rival-config-import -- 5 subjects / 5 findings
kinds: unique 1, portable 3, nuance 1

**Best three implementations** (evidence-ranked):
1. **atomic-agent** (atomic-agent-10, portable) -- first run offers to bring skills, memory, MCP servers, sessions, cron jobs and (opt-in) provider keys over from Hermes, OpenClaw, Claude Code, Codex `_(src/import/claude-code/; src/import/codex/)_`
2. **claw-code-agent** (claw-code-agent-b9, portable) -- Reads Claude Code's own config surfaces (.claude/CLAUDE.md, rules/, agents/, plugins cache, account.json) as first-class inputs `_(src/agent_context.py:351-365)_`
3. **codewhale** (codewhale-e11, portable) -- /import-claude: consent-gated, bounded, secret-safe migration from Claude Code, plus plugin-compat doc `_(crates/tui/src/import_claude.rs:1-26)_`

**HOTDOG POSITION.** absent.

### sandbox-absent -- 5 subjects / 5 findings
kinds: anti-pattern 1, safety-hole 4

**Best three implementations** (evidence-ranked):
1. **oh-my-pi** (oh-my-pi-s3, safety-hole) -- docs repeatedly and honestly disclaim containment - the codex 10-rung 'underneath' does not exist, not even opt-in `_(docs/approval-mode.md:72)_`
2. **bitfun** (bitfun-s1, safety-hole) -- all enforcement is in-process Rust policy, while the tool surface includes computer-use, terminal, remote SSH and relay control `_(src/crates/services/services-core/src/process_manager.rs:52)_`
3. **goose** (goose-s6, anti-pattern) -- tool exec runs at user privilege on all platforms, and the one experimental OS sandbox (macOS seatbelt) was removed and announced as removed `_(documentation/blog/2026-02-23-goose-v1-25-0/index.md:22)_`

**DESIGN TENSION.** Merged presence-of-absence census per wave6 (stance-vs-omission nuance preserved in one_why): honest absence (opencode-s2's 'does NOT sandbox the agent, escapes out of scope'; oh-my-pi-s3; goose-s6 having REMOVED its one seatbelt and announced it) vs compiled-out-and-undisclosed (zeroclaw-s2 paired with its docs finding) vs in-process-only against a computer-use/SSH surface (bitfun-s1). Per doctrine honest absence outscores phantom protection; nothing here is a design pole, but paired against confine-or-refuse-startup it maps the enforcement floor of the corpus -- a floor hotdog shares (hotdog-8) with its eyes open.

**HOTDOG POSITION.** absent.

### side-effect-verified-guardrail-tests -- 5 subjects / 5 findings
kinds: portable 5

**Best three implementations** (evidence-ranked):
1. **forge-norvialabs** (forge-norvialabs-4, portable) -- permission_contract.rs spawns real processes and asserts what the kernel did, written around four historical defects a unit test missed `_(crates/forge-tools/tests/permission_contract.rs:1-40)_`
2. **grok-build** (grok-build-s12, portable) -- poisoned git checkout, forged bwrap marker, non-leading deny segments, deny-beats-yolo, and kernel controls that must stay readable `_(permission/grants_tests.rs:15-39)_`
3. **tura** (tura-7, portable) -- the interceptor e2e actually executes commands under a Docker POSIX harness and asserts the destructive side effect never happened, while safe `_(crates/tools/tests/business/command_interceptor_e2e.rs:1-12)_`

**DESIGN TENSION.** The property-test standard for enforcement code: prove the side effect did not happen. forge-norvialabs-4 (permission_contract.rs spawns real processes and asserts what the KERNEL did, written around four historical defects a unit test missed), grok-build-s12 (tests written as attack properties: poisoned git checkout, forged bwrap marker, non-leading deny segments, deny-beats-yolo), tura-7 (destructive commands actually executed under a Docker POSIX harness; asserts the destruction never occurred), bitfun-s6 (deny-path tests assert the fake tool's execution counter never moved), zeroclaw-s11 and e2e-runs-against-real-sandbox x4 same claim. hotdog's fail-closed user-gate matrix is in-family at the prompt layer -- the kernel-denial rung is absent.

**HOTDOG POSITION.** absent.

### subagent-process-delegation -- 5 subjects / 5 findings
kinds: unique 1, portable 1, nuance 3

**Best three implementations** (evidence-ranked):
1. **goose** (goose-e5, portable) -- Subagents are first-class sessions in the same journal store (SessionType::SubAgent) with hard guards against a subagent writing a peer user session `_(crates/goose/src/agents/platform_extensions/orchestrator.rs:788-793)_`
2. **code** (code-e5, unique) -- Subagents are rival harnesses spawned as external processes from a built-in catalog (claude, gemini, copilot, qwen, antigravity, own code CLI) with `_(code-rs/core/src/agent_defaults.rs:17-34)_`
3. **kimi-cli** (kimi-cli-7, nuance) -- Background tasks run as detached worker processes with heartbeats, control-file polling, stale reconciliation, and Windows process-tree kill `_(src/kimi_cli/background/worker.py:21-40)_`

**HOTDOG POSITION.** absent.

### unrenamed-manifest-identity -- 5 subjects / 5 findings
kinds: nuance 1, anti-pattern 4

**Best three implementations** (evidence-ranked):
1. **mimo-code** (mimo-code-1, anti-pattern) -- fork keeps name:opencode in root manifest and packages/opencode/ - census rename-clone misfire `_(mimo-code/package.json:3)_`
2. **open-interpreter** (open-interpreter-2, anti-pattern) -- root package.json never renamed - defeats provenance tooling and census detection `_(FORK_BRANDING.md:1-6)_`
3. **code** (code-e7, anti-pattern) -- Both shipped SDK directories are verbatim upstream artifacts that point users at the upstream product: TypeScript SDK's name is still `_(sdk/typescript/package.json:2)_`

**HOTDOG POSITION.** absent.

### ported-proprietary-source -- 4 subjects / 5 findings
kinds: license-risk 5

**Best three implementations** (evidence-ranked):
1. **claw-code-agent** (claw-code-agent-1, license-risk) -- systematic Python port of leaked Claude Code source, declared in-code `_(claw-code-agent/src/bash_security.py:3)_`
2. **claurst** (claurst-1, license-risk) -- clean-room claim unsupported: spec generated from the leaked Claude Code sourcemap, implementation comments port TS modules 1:1 `_(README.md:208-218)_`
3. **claw-code** (claw-code-1, license-risk) -- census 'original' false: self-declared Python-then-Rust rewrite of Claude Code, evidenced by committed surface snapshots of the proprietary tree `_(src/__init__.py:1)_`

**HOTDOG POSITION.** absent.

### rival-subscription-transport -- 4 subjects / 5 findings
kinds: unique 1, nuance 3, license-risk 1

**Best three implementations** (evidence-ranked):
1. **grinta-coding-agent** (grinta-coding-agent-10, unique) -- ChatGPT-subscription Codex OAuth as the harness's own model transport `_(backend/inference/clients/codex_app_server.py:1-8,29)_`
2. **molt** (molt-7, nuance) -- Subscription models ride the vendor's own CLI over documented protocols (ACP to grok/gemini, Claude Agent SDK) instead of lifting OAuth tokens; the `_(src/acp.ts:5-15)_`
3. **zot** (zot-b3, nuance) -- Anthropic OAuth mode impersonates Claude Code's first-party identity line to ride subscription auth `_(packages/provider/anthropic.go:25,268-272)_`

**DESIGN TENSION.** Same fault line as subscription-continuity: spawning the operator's own logged-in official CLI (atomic-agent-11) or refusing to read rival auth stores (molt-7, 'refuses to read ~/.grok/auth.json') is the honest pole; lifting OAuth or shipping rival identity headers is the punished pole (grinta-b4 ruled license-risk, zot-b3, qqcode-3). The corpus has not settled whether ToS violation books as license-risk or safety; wave6 ruled license-risk for grinta and the boundary review kept it there.

**HOTDOG POSITION.** absent.

### test-fixture-loc-inflation -- 4 subjects / 5 findings
kinds: nuance 2, anti-pattern 3

**Best three implementations** (evidence-ranked):
1. **kolega-code** (kolega-code-b2, anti-pattern) -- 172k test-LOC claim inflated ~40% by shipped skill package-data and benchmark vendored trees; real property corps ~121k/416 files `_(kolega_code/_bundled_skills (30,575 py LOC, 23,527 shipped tools))_`
2. **auto-code-rover** (auto-code-rover-3, anti-pattern) -- 927MB of checked-in SWE-bench run outputs (3,161 .diff blobs under results/) made the census report 2.6M LOC, primary_language 'diff', and suggested `_(results/acr-run-1/applicable_patch/*/*.diff (3,161 files, du results =)_`
3. **deepagents** (deepagents-8, anti-pattern) -- test_loc inflated ~45% by vendored JSON datasets sitting under directories named tests `_(libs/evals/tests/evals/tau2_airline/data/db.json (205,192 LOC), tasks.)_`

**DESIGN TENSION.** Meta-failure of measurement, not design: auto-code-rover's 927MB committed run outputs pushing a 13.9k-LOC tool toward a T3 review, deepagents' 205k-LOC JSON 'test data', kolega-code's bundled skills, zeroclaw's inverse error (inline #[cfg(test)] making census test_loc understate ~15x). Rule it feeds the whole study: verification is never scored from census numbers; generated-blob exclusion lists are mandatory.

**HOTDOG POSITION.** absent.

### code-mode-runtime -- 4 subjects / 4 findings
kinds: unique 3, nuance 1

**Best three implementations** (evidence-ranked):
1. **maki** (maki-3, unique) -- Embedded monty Python interpreter exposes every tool as an async function, gated by the same dispatcher, with hook-bypass-proof file access `_(plugins/code_execution/init.lua:1-40)_`
2. **codex** (codex-7, unique [conf med]) -- Code-mode: agent drives tools by executing JS in a persistent v8 session over gRPC `_(codex-rs/code-mode/src/grpc_session/, .github/workflows/rusty-v8-relea)_`
3. **kolega-code** (kolega-code-4, unique) -- persistent per-session Python/JS eval kernels with an in-kernel tool bridge back through the agent's permission-gated tool path `_(kolega_code/agent/eval/kernel.py:1-11)_`

**HOTDOG POSITION.** absent.

### e2e-runs-against-real-sandbox -- 4 subjects / 4 findings
kinds: unique 1, portable 3

**Best three implementations** (evidence-ranked):
1. **gemini-cli** (gemini-cli-s10, unique) -- Whole integration suite runs twice - GEMINI_SANDBOX=false and GEMINI_SANDBOX=docker/podman - so safety wrapping is exercised end-to-end, and CI `_(package.json:56,58,63)_`
2. **deepseek-reasonix** (deepseek-reasonix-s11, portable) -- Sandbox denial is tested by execution, on the real backend, in CI: a macOS test writes under and outside the write-root through sandbox-exec and `_(internal/safety/sandbox/seatbelt_darwin_test.go:206-263)_`
3. **codewhale** (codewhale-s5, portable) -- Live kernel denial tests and adversarial bypass tests, not existence tests `_(crates/tui/src/sandbox/seatbelt/tests.rs:191-240)_`

**DESIGN TENSION.** Safety-under-test doctrine in four subjects: gemini-cli-s10 (whole integration suite runs twice, sandbox none and docker, bwrap provisioned in the 3-OS matrix), codewhale-s5 (real sandbox-exec spawns asserting baseline-read-works/deny-blocks), deepseek-reasonix-s11 (writes under and outside the write-root through sandbox-exec in CI), zeroclaw-s8. This is enforcement-tests-in-CI at the kernel boundary -- the exact gate the amended doctrine's +1 credit requires. hotdog: absent.

**HOTDOG POSITION.** absent.

### event-sourced-agent-state -- 4 subjects / 4 findings
kinds: unique 2, nuance 2

**Best three implementations** (evidence-ranked):
1. **nausicaa-harness** (nausicaa-harness-1, unique) -- Agent core is an append-only JSONL ledger: scoped idempotency keys, CAS replacement, content-addressed payloads, enforced event chains `_(src/ledger/ledger.ts:323-328)_`
2. **kimi-code** (kimi-code-6, unique) -- Agent state is a replayable event fold with undoable keys and journal repair migrations `_(packages/agent-core-v2/src/state/eventDispatcherService.ts:169-267,564-594)_`
3. **devon** (devon-2, nuance) -- Loop consumes an append-only producer/consumer event list persisted to sqlite every model step `_(devon_agent/session.py:788-843)_`

**HOTDOG POSITION.** absent.

### guardian-reviewer-pool -- 4 subjects / 4 findings
kinds: unique 1, portable 1, nuance 2

**Best three implementations** (evidence-ranked):
1. **san** (san-3, nuance) -- Tool-less fail-closed LLM permission judge with injection-hardened prompt composition `_(internal/reviewer/reviewer.go:1-13,82-96,106-110)_`
2. **workground2** (workground2-s7, portable) -- Opt-in second-model approval reviewer, fail-closed, circuit-broken `_(internal/boot/boot.go:1748-1761)_`
3. **codex** (codex-6, unique [conf med]) -- Guardian extension owns a reviewer pool that pre-warms review context for risky agent startups `_(codex-rs/core/src/guardian_review.rs:1-5,)_`

**DESIGN TENSION.** Second-model review surfaces: workground2-s7 (opt-in reviewer, fail-closed, circuit-broken), san-3 (tool-less fail-closed LLM permission judge with injection-hardened prompt composition), keen-code-7 (/adversary on a SEPARATE provider client with write/exec tools structurally stripped), codex-6 (pre-warmed reviewer pool for risky startups). Shared invariant: the reviewer must have LESS authority than the actor. hotdog: workflow judge nodes are the orchestration-scale cousin (validated-finish-gate).

**HOTDOG POSITION.** absent.

### headless-rpc-protocol -- 4 subjects / 4 findings
kinds: portable 4

**Best three implementations** (evidence-ranked):
1. **codewhale** (codewhale-e10, portable) -- Published @codewhale/runtime-sdk in CI + headless exec --json/--output-format stream-json + VS Code extension `_(.github/workflows/release.yml:94-102,424)_`
2. **goose** (goose-e9, portable) -- run --output-format text|json|stream-json with typed error events, session export/import, and SDKs published by CI for Python, Java/Maven and npm `_(crates/goose-cli/src/cli.rs:313-319)_`
3. **pi** (pi-9, portable) -- RPC and JSON app modes with a documented command protocol and client/server packages `_(packages/coding-agent/src/core/project-trust.ts:12)_`

**DESIGN TENSION.** Consensus: document the machine contract or lose the ecosystem -- pi-9 (documented RPC command protocol + client/server packages), maki-11 (stream-json deliberately Claude-Code-output-compatible + SDK input mode), codewhale-e10 (CI version-parity-gated SDK publish), goose-e9 (typed error events + CI-published Python/Java/npm SDKs). hotdog: structured output via synthetic tool, own ws protocol undocumented -- absent side.

**HOTDOG POSITION.** absent.

### interop-ceilings -- 4 subjects / 4 findings
kinds: nuance 2, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **orca-agent** (orca-agent-e10, anti-pattern) -- MCP client only (no server), no published SDK, no first-party IDE surface, DeepSeek-only provider behind the whole economy `_(deepseek_http.rs:339-357)_`
2. **bitfun** (bitfun-e10, nuance) -- MCP is client-only (the mcp 'server' modules manage outbound connections; no surface exposes bitfun as an MCP server), the TypeScript SDK is `_(src/crates/services/services-integrations/src/mcp/server/connection.rs:166-167)_`
3. **opensquilla** (opensquilla-e15, nuance) -- No ACP and no published out-of-tree SDK `_(scripts/generate_router_tier_contract.py:12)_`

**DESIGN TENSION.** DESIGN TENSION: breadth vs provable depth. zeroclaw-e14 is the named trap: 30+ channels, gateway, A2A, relay, WASM plugins -- each surface has a contract, none is provably deep; orca-agent-e10 and opensquilla-e15 are ceilings-by-absence. Counterweight: interop-matrix implementations with conformance tests. hotdog sits on the depth-light side too: MCP client is conformance-tested, which is the one ceiling claim we can make.

**HOTDOG POSITION.** absent.

### layered-loop-separation -- 4 subjects / 4 findings
kinds: portable 3, nuance 1

**Best three implementations** (evidence-ranked):
1. **orca-agent** (orca-agent-c3, portable) -- run_agent_loop (724) -> run_agent_turn_loop (1127) -> iteration/kernel (142/134), with children reusing the identical loop `_(crates/orca-runtime/src/agent_loop.rs:35,)_`
2. **gemini-cli** (gemini-cli-c1, portable) -- Three-stage agent loop: bounded turn driver / chat-history engine / stream-reducer, with tool execution in a separate queued scheduler `_(packages/core/src/core/client.ts:79,924,974)_`
3. **oh-my-pi** (oh-my-pi-c1, portable) -- Three-layer spine is real and single-seamed: mock-testable loop package -> AgentSession decision hub -> thin modes, one construction path `_(packages/agent/src/agent-loop.ts:604,1048,1163)_`

**HOTDOG POSITION.** absent.

### manifest-only-license -- 4 subjects / 4 findings
kinds: license-risk 4

**Best three implementations** (evidence-ranked):
1. **coro-code** (coro-code-7, license-risk) -- MIT OR Apache-2.0 declared in Cargo.toml only; no LICENSE file anywhere in the tree `_(Cargo.toml:10)_`
2. **darce-cli** (darce-cli-9, license-risk) -- MIT declared in package.json but no LICENSE file in tree and GitHub reports license:null `_(package.json:38)_`
3. **g3** (g3-9, license-risk) -- MIT declared in Cargo.toml and README but no LICENSE file anywhere in the tree `_(Cargo.toml:37)_`

**HOTDOG POSITION.** absent.

### opt-in-hard-enforcement -- 4 subjects / 4 findings
kinds: nuance 1, anti-pattern 2, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **letta-code** (letta-code-s4, anti-pattern) -- The cross-agent shell sandbox is opt-in (agent shells run unconfined by default), even though the machinery itself is first-rate: bwrap tmpfs-masks `_(src/permissions/sandbox-gate.ts:37-38)_`
2. **octomind** (octomind-b4, safety-hole) -- Shipped default has zero binding enforcement; no SECURITY.md `_(config-templates/default.toml:33,750-751)_`
3. **codewhale** (codewhale-s3, anti-pattern) -- OS-sandbox enforcement is macOS-only automatic; Linux bwrap opt-in, seccomp dormant, Windows absent `_(crates/tui/src/sandbox/mod.rs:9-21)_`

**HOTDOG POSITION.** absent.

### parallel-agent-stack-migration -- 4 subjects / 4 findings
kinds: nuance 2, anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **goose** (goose-c2, anti-pattern) -- Two complete loops ship side-by-side and the clean one is OFF by default: GOOSE_STATE_MACHINE defaults false, CI never sets it, so production runs `_(crates/goose/src/agents/state_machine/mod.rs:72-75)_`
2. **mistral-vibe** (mistral-vibe-c1, anti-pattern) -- Dual-backend migration: full agent stack exists twice behind a 9,519-LOC adapter, and CI exercises only the legacy leg `_(tests/conftest.py:96)_`
3. **amazon-q-developer-cli** (amazon-q-developer-cli-10, nuance) -- 53k-LOC legacy chat loop and a 13k-LOC clean agent-framework crate, each with its own tools, hooks, and MCP code `_(Cargo.toml:8-9)_`

**HOTDOG POSITION.** absent.

### ui-coupled-loop -- 4 subjects / 4 findings
kinds: anti-pattern 4

**Best three implementations** (evidence-ranked):
1. **continue** (continue-c1, anti-pattern) -- The agent loop lives in the GUI's Redux thunk layer, not in core/: core is demoted to a model/tool RPC service on the primary IDE surface `_(gui/src/redux/thunks/streamNormalInput.ts:85-398)_`
2. **nanocoder** (nanocoder-7, anti-pattern) -- Agent loop implemented inside React/Ink hooks forces parallel non-UI paths `_(source/hooks/chat-handler/conversation/conversation-loop.tsx (1414 LOC)_`
3. **codewhale** (codewhale-c2, anti-pattern) -- The loop calls the product surface directly inside run_turn, gated by a config flag instead of a port `_(crates/tui/src/core/engine/turn_loop.rs:728-729)_`

**HOTDOG POSITION.** absent.

### interactive-pty-e2e -- 3 subjects / 3 findings
kinds: portable 3

**Best three implementations** (evidence-ranked):
1. **3code** (3code-6, portable) -- one TtySession API over openpty (POSIX) and ConPTY (Windows kernel32 declarations hand-rolled), driving the real binary through a ttty grid with `_(tests/tty_expect.nim:1-9)_`
2. **ob-1** (ob-1-13, portable) -- TUI tested under a REAL pseudo-terminal in CI with a pyte terminal emulator asserting render persistence and streaming hygiene `_(.github/workflows/ci.yml:27-49)_`
3. **octomind** (octomind-10, portable) -- real binary driven inside a pseudo-terminal via a Python pty driver -- type a prompt, read the streamed answer, exit via /slash-command `_(tests/interactive_pty_e2e_test.rs:15-18,46-47)_`

**DESIGN TENSION.** The verification rung nobody under 8 seems to have: drive the REAL binary under a real pseudo-terminal. 3code-6 (self-built openpty/ConPTY expect harness, 3,885-LOC TTY grid), ob-1-13 (pyte terminal emulator in CI asserting render persistence + streaming hygiene), octomind-10 (pty driver types a prompt and exits via slash). hotdog: everything in-process -- explicitly the verification-7 ceiling marker (absent).

**HOTDOG POSITION.** absent -- everything in-process; this is the named verification-7 ceiling marker, stated plainly.

### session-tree-branching -- 3 subjects / 3 findings
kinds: unique 1, portable 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **pi** (pi-1, unique) -- Sessions are entry trees: /fork /clone /tree navigation with branch-switch summarization `_(packages/coding-agent/docs/sessions.md:20-32,)_`
2. **codewhale** (codewhale-c5, portable) -- Canonical session history is an append-only entry journal with a leaf pointer; the message list is a compat projection `_(crates/tui/src/session_manager.rs:906-917)_`
3. **zeroclaw** (zeroclaw-c8, anti-pattern) -- Sessions resume but cannot fork, branch, rewind, or checkpoint; session/git_branch is display-only `_(dispatch.rs:16818,)_`

**DESIGN TENSION.** DESIGN TENSION: sessions as trees vs linear logs. Tree pole: pi-1 (entry trees, /fork /clone /tree navigation with branch-switch summarization -- the anchor), codewhale-c5 (canonical history is an append-only entry journal with a LEAF pointer; the message list is a compat projection -- trees without forking ceremony). Linear pole: zeroclaw-c8 (resume works; fork/branch/rewind explicitly absent). hotdog: /fork is a linear branch-off, no tree navigation -- between the poles, closer to linear.

**HOTDOG POSITION.** /fork exists as linear branch-off, no tree -- between the poles, closer to the zeroclaw side without the anti-pattern finding.

### approval-only-enforcement -- 3 subjects / 3 findings
kinds: anti-pattern 2, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **aider** (aider-s1, anti-pattern) -- Model-suggested shell commands execute as raw shell strings with zero enforcement underneath: no sandbox, no allowlist, no path confinement anywhere `_(aider/run_cmd.py:10-19)_`
2. **plandex** (plandex-4, safety-hole) -- Automode presets gate exec by config only -- shell runs via sh -c with nothing underneath `_(app/shared/plan_config.go:130-143)_`
3. **san** (san-10, anti-pattern) -- No OS sandbox at all: approved tool calls run with full user privileges, and SECURITY.md never says so plainly `_(selflearn/scan.go:12)_`

**HOTDOG POSITION.** absent.

### autopilot-default-failopen -- 3 subjects / 3 findings
kinds: anti-pattern 1, safety-hole 2

**Best three implementations** (evidence-ranked):
1. **ob-1** (ob-1-3, safety-hole) -- the on-by-default folder-trust gate downgrades only an IMPLICIT autopilot, so one /trust or any saved setting yields unapproved, unsandboxed writes `_(src/config.ts:706)_`
2. **bitfun** (bitfun-s2, safety-hole) -- a fresh install with no config file resolves the baseline rule ('*','*')=>Allow, i.e. no approval at all `_(src/crates/contracts/product-domains/src/tool_permissions.rs:187-190)_`
3. **letta-code** (letta-code-s1, anti-pattern) -- The shipped default permission mode auto-allows every tool: approval machinery is real and well-tested but switched off out of the box `_(src/permissions/mode.ts:10)_`

**HOTDOG POSITION.** absent.

### cost-accounting -- 3 subjects / 3 findings
kinds: portable 2, nuance 1

**Best three implementations** (evidence-ranked):
1. **continue** (continue-e4, nuance) -- Token/cost accounting exists end-to-end, including cache-cost line items and actionable context errors `_(core/llm/utils/calculateRequestCost.ts:136)_`
2. **grok-build** (grok-build-e7, portable) -- Usage accounting distinguishes fully-reported, partial, and missing provider costs `_(xai-chat-state/src/usage.rs:26)_`
3. **zeroclaw** (zeroclaw-e3, portable [conf None]) -- single budget-resolver authority, image/schema/system-floor estimates, remediation text, documented estimate blindness `_(crates/zeroclaw-runtime/src/agent/turn/mod.rs:97-108)_`

**HOTDOG POSITION.** absent.

### default-on-telemetry -- 3 subjects / 3 findings
kinds: nuance 2, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **keen-code** (keen-code-11, anti-pattern) -- GA4 session start/end telemetry is on by default at build time, with opt-out only via KEEN_TELEMETRY / DO_NOT_TRACK / CI env vars `_(internal/telemetry/telemetry.go:81-92)_`
2. **devon** (devon-7, nuance) -- PostHog telemetry on by default with hardcoded project key; opt-out only on exact env string "true" `_(devon_agent/utils/telemetry.py:106-113)_`
3. **jcode** (jcode-s8, nuance) -- transcript sharing is genuinely off-by-default, versioned-consent, double-redacted before upload; ordinary usage telemetry however is ON by default `_(TELEMETRY.md:5-7)_`

**HOTDOG POSITION.** absent.

### deterministic-fuzz-replay -- 3 subjects / 3 findings
kinds: unique 1, portable 2

**Best three implementations** (evidence-ranked):
1. **agentty** (agentty-b1, portable) -- Oracled property fuzzers with hardcoded seed families run in the primary blocking CI gate; found failures promote to committed regression tests `_(tests/frozen_invariant_fuzz.cpp:22-38,425-433)_`
2. **smelt** (smelt-1, unique) -- 17 oracled fuzz targets with committed regression seeds replayed in CI `_(fuzz/README.md:3-27)_`
3. **minicode** (minicode-4, portable) -- Seeded-PRNG bash-guard fuzz plus shadow-git stress and adversarial MCP server on nightly CI `_(experiments/extreme-bash-fuzz.ts:17-34)_`

**DESIGN TENSION.** Consensus rule with exemplars: fuzz must have an oracle, persisted seeds, and a blocking CI home. smelt-1 (17 oracled targets, committed regression seeds, in CI), agentty-b1 (seed families run in the primary blocking gate; found failures promote to committed regression tests), minicode-4 (seeded-PRNG bash-guard fuzz + shadow-git stress + adversarial MCP server nightly). Against zeroclaw-b2's five unwired targets (three fuzzing serde itself) and kolkrabbi-b3's seed-only 'deliberately never gating' fuzz.

**HOTDOG POSITION.** absent.

### docs-drift-gates -- 3 subjects / 3 findings
kinds: unique 1, portable 2

**Best three implementations** (evidence-ranked):
1. **qwen-code** (qwen-code-e10, portable) -- 1121-doc tree + generated OpenAPI kept honest by a contract test; feature docs match real surfaces `_(README.md:11-17)_`
2. **orca-agent** (orca-agent-e12, portable) -- a clap-tree-vs-manifest equality test pins the public CLI manifest, and a forbidden-claims test sweeps every public docs page for obsolete surface `_(src/cli.rs:908-960)_`
3. **dexto** (dexto-e12, unique) -- OpenAPI spec sync check, CLI README sync, and custom ESLint rules that force every route to declare JSON error responses `_(docs/docusaurus.config.ts:65)_`

**DESIGN TENSION.** Bidirectional drift gating: orca-agent-e12 (clap-tree-vs-manifest equality test + forbidden-claims sweep over every public docs page), qwen-code-e10 (contract test guards OpenAPI vs reference), dexto-e12 (OpenAPI sync + CLI README sync + custom ESLint rules forcing typed error declarations per route). Consensus mechanism, no tension.

**HOTDOG POSITION.** absent.

### dual-session-abstraction -- 3 subjects / 3 findings
kinds: nuance 2, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **kilocode** (kilocode-c2, anti-pattern) -- Two session engines ship simultaneously on one server and one database (v1 prompt loop + v2 Effect runner) `_(prompt.ts:1543)_`
2. **bitfun** (bitfun-c4, nuance [conf med]) -- Runtime concepts exist in two mirrored trees (assembly/core agentic vs execution/agent-runtime) but the direction is enforced: agent-runtime owns `_(core/state.rs:5)_`
3. **gemini-cli** (gemini-cli-c10, nuance [conf med]) -- New AgentSession protocol wrapper and legacy-agent-session coexist inside core during migration `_(packages/core/src/agent/agent-session.ts:225)_`

**HOTDOG POSITION.** absent.

### fork-cache-alignment -- 3 subjects / 3 findings
kinds: unique 3

**Best three implementations** (evidence-ranked):
1. **waveloom** (waveloom-3, unique) -- Forked subagents fire byte-aligned with the parent's cached prefix, tools array included; parent-only tools become explicit error stubs `_(pkg/agentloop/execute.go:69-76)_`
2. **grok-build** (grok-build-e6, unique) -- Auxiliary calls (recap, turn summary) replay the parent conversation with byte-identical prefixes to reuse the parent's prompt cache `_(acp_session_impl/side_call.rs:102-131)_`
3. **qwen-code** (qwen-code-e4, unique) -- Subagent forks have an explicit cache-preserving mode that shares the parent's prompt prefix, tested separately `_(packages/core/src/agents/forkedAgent.ts:10-26)_`

**HOTDOG POSITION.** absent.

### lease-fenced-run-execution -- 3 subjects / 3 findings
kinds: unique 1, portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **nausicaa-harness** (nausicaa-harness-9, unique) -- Runs are owned through TTL leases with monotonic fencing tokens; daemon workers persist hash-chained recovery journals and durable writes re-check `_(src/runtime/execution-lease.ts:9-15,53-58)_`
2. **mistral-vibe** (mistral-vibe-c4, portable) -- Session lease: OS byte-range lock with symlink refusal, pid/timestamp diagnostics beside (not inside) the lock, explicit busy error `_(vibe/core/session/session_lease.py:42-96)_`
3. **deepseek-reasonix** (deepseek-reasonix-c5, nuance) -- acquire with holder identity, same-runtime reclaim, cross-runtime detection, plus a parent-guard on recovery branches and idle covered recovery `_(internal/state/sessionstore/session_lease.go:84,126,180,225)_`

**HOTDOG POSITION.** absent.

### module-size-ratchet -- 3 subjects / 3 findings
kinds: portable 2, nuance 1

**Best three implementations** (evidence-ranked):
1. **ouroboros** (ouroboros-1, portable) -- CI-blocking size ratchet: 1600-line module / 300-line function ceilings with tracked debt, enforced in every job `_(ouroboros/review.py:19-23)_`
2. **qwen-code** (qwen-code-c9, portable) -- ESLint ratchet files freeze legacy imports/filenames so boundary debt only shrinks `_(eslint.legacy-core-barrel-imports.mjs:1-11)_`
3. **kolkrabbi** (kolkrabbi-b2, nuance) -- Zero-god-file discipline is measured-true (max non-test file 1,692 LOC) but has NO direct per-file LOC gate - smallness is maintained indirectly by `_(internal/engine/agent.go:1-1692)_`

**HOTDOG POSITION.** absent.

### no-fuzzing-advisory-evals -- 3 subjects / 3 findings
kinds: nuance 2, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **hax** (hax-b4, anti-pattern) -- Zero fuzz targets and no model evals in CI - a wire-parsing C agent that hand-tests SSE/JSON bodies but never fuzzes them `_(grep -ri 'LLVMFuzzer|fuzz' across src tests scripts .github = 0 hits)_`
2. **gemini-cli** (gemini-cli-s11, nuance) -- No fuzzing anywhere, and the eval harness deliberately weakens: USUALLY_PASSES/USUALLY_FAILS cases are skipped unless RUN_EVALS, so evals are `_(evals/test-helper.ts:380-387)_`
3. **codewhale** (codewhale-s11, nuance) -- No fuzzing anywhere; the in-CI 'eval' harness is offline prompt/composition checking, not a model eval `_(.github/workflows/ci.yml:901-908,)_`

**HOTDOG POSITION.** absent.

### oauth-client-impersonation -- 3 subjects / 3 findings
kinds: nuance 2, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **claurst** (claurst-2, safety-hole) -- documented stealth stack spoofs Claude Code client so subscription OAuth tokens pass Anthropic's gates `_(core/src/oauth_config.rs:51)_`
2. **qqcode** (qqcode-3, nuance) -- Anthropic OAuth path injects the verbatim Claude Code system-prompt prefix and oauth beta header to pass the subscription gate `_(vibe/core/llm/backend/anthropic_sdk.py:29,58-61,339-344)_`
3. **open-interpreter** (open-interpreter-b3, nuance [conf med]) -- Harness modes ship rival client identity headers ('claude-cli/2.1.158 (external, sdk-cli)', 'Bun/1.3.14', ZCode referer/user-agent) with no `_(core/src/harness/claude_code.rs:52-53,131-136)_`

**DESIGN TENSION.** The narrow safety-hole slice of the subscription fight: claurst-2's documented stealth stack with an extracted proprietary billing salt, qqcode-3's verbatim Claude Code prompt prefix + oauth beta header, open-interpreter-b3's rival client-identity headers with no fork-owned header on the wire. There is no implementer-defense side inside the concept; the counterweight is honest-transport evidence in subscription-continuity / rival-subscription-transport.

**HOTDOG POSITION.** absent.

### onboarding-error-dx -- 3 subjects / 3 findings
kinds: portable 3

**Best three implementations** (evidence-ranked):
1. **bitfun** (bitfun-e13, portable) -- check:build-prereqs --fix naming every failure mode with its fix command, openbitfun doctor / acp doctor, structured provider-agnostic errors built `_(CONTRIBUTING.md:26-47)_`
2. **orca-agent** (orca-agent-e13, portable) -- `orca doctor` fails checks with actionable fix hints, first-run disclosure is digest-pinned and lock-guarded, and docs disclaim their own limits `_(crates/orca-runtime/src/diagnostics.rs:248-271)_`
3. **opensquilla** (opensquilla-e17, portable) -- Onboarding and error surfaces are operator-grade `_(docs/quickstart.md:36-56)_`

**HOTDOG POSITION.** absent.

### receipt-based-token-accounting -- 3 subjects / 3 findings
kinds: portable 3

**Best three implementations** (evidence-ranked):
1. **tau** (tau-2, portable) -- Context accounting treats the latest provider usage receipt as authoritative for the prefix and estimates only the trailing delta `_(src/tau_coding/context_window.py:186-260)_`
2. **codebuff** (codebuff-10, portable) -- Context size = last provider receipt plus string-length deltas, never re-tokenizing; recount even on failed and cancelled turns `_(packages/agent-runtime/src/util/context-token-count.ts:20-40)_`
3. **3code** (3code-8, portable [conf med]) -- Live token bar repainted with accurate post-call usage, then committed as a same-tone 'receipt' row in scrollback when the user submits the next turn `_(src/threecode/fatprompt/runtime.nim:63-79)_`

**HOTDOG POSITION.** absent.

### recursive-subagent-spawn -- 3 subjects / 3 findings
kinds: portable 2, nuance 1

**Best three implementations** (evidence-ranked):
1. **opencode** (opencode-e7, portable) -- typed subagent spawn with resume-by-task_id, configurable depth budget, deny-inheriting permission derivation, and flag-gated background mode that `_(packages/opencode/src/tool/task.ts:44-50)_`
2. **prime-agent** (prime-agent-5, portable) -- RLM recursive subagent registry with user-settable max-depth budget `_(packages/coding-agent/src/core/rlm-runtime.ts:71,137-377)_`
3. **openhands** (openhands-5, nuance) -- Agent-triggered child conversations with isolation enum and a corruption-tolerant local launch ledger `_(src/services/child-conversation-launch.ts:22-40,206-215)_`

**HOTDOG POSITION.** absent.

### subagent-permission-inheritance -- 3 subjects / 3 findings
kinds: unique 1, portable 1, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **opencode** (opencode-b6, unique) -- Subagent sessions structurally cannot shed parent denies; task/todowrite denied unless the subagent's own ruleset grants them `_(packages/opencode/src/agent/subagent-permissions.ts:15-30)_`
2. **keen-code** (keen-code-b4, safety-hole) -- Subagent registries ship a hard-wired unconditional AutoApprover - delegated bash/write/edit/MCP never surface a user prompt `_(internal/subagents/tool_factory.go:13-15,31-45)_`
3. **zeroclaw** (zeroclaw-subagent-policy-containment, portable [conf None]) -- Subagents are policy-validated with shared action budgets and a depth-1 cap; the code admits exactly which ceilings are enforced `_(crates/zeroclaw-runtime/src/subagent/mod.rs:1-4)_`

**DESIGN TENSION.** what a subagent may shed. Clamp pole: opencode-b6 (subagents structurally cannot shed parent denies -- negative-tested), bitfun-s3 (independently evaluated runtime ceilings that typed-reject Allow rules), kolkrabbi-4 (Ask converts to Deny in subagents), openhands-1 (narrowing-only rank table), zeroclaw's shared-budget containment. Hole shape: keen-code-b4 (hard-wired unconditional AutoApprover in subagent registries), kolega-code-b1 (delegation hardcodes AUTO + auto-allow and the delegation tools carry no permission kind). Evidence is unanimous: inheritance must be a narrowing clamp. hotdog manager-profile gating is in-family, unrecorded.

**HOTDOG POSITION.** absent.

### subscription-continuity -- 3 subjects / 3 findings
kinds: unique 2, portable 1

**Best three implementations** (evidence-ranked):
1. **kolkrabbi** (kolkrabbi-1, unique) -- durable pause, token-free lift probe, and mid-turn auto chain-walk across the user's own equivalents (vendor CLIs, gateway, local endpoints) `_(internal/engine/resume.go:15-49)_`
2. **goose** (goose-e8, unique) -- the Claude Code, Codex, Gemini CLI and Copilot ACP runtimes are spawned as child processes and driven through goose's Provider trait `_(crates/goose/src/providers/claude_code.rs:255)_`
3. **3code** (3code-7, portable) -- exponential backoff to a 2048s ceiling for ~36 hours on 429/5xx/network, retries after the initial window printed into the transcript as numbered `_(docs/manual.md:588-613)_`

**DESIGN TENSION.** ride the subscription honestly vs impersonate it. Honest pole: kolkrabbi-1 (durable pause + token-free lift probe + mid-turn auto chain-walk across the user's OWN equivalents -- vendor CLIs, gateway, local endpoints), 3code-7 (patient retry sized to outlive rolling caps, printed into the transcript), goose-e8/letta-code-e5 (rival harnesses spawned as their own official CLIs over documented protocols). Impersonation pole: zot-b3 (spoofs Claude Code's first-party identity line to ride subscription auth). hotdog: local-first, no stake either way -- absent, and the honest pole fits the thesis.

**HOTDOG POSITION.** absent.

### tool-output-spillover-files -- 3 subjects / 3 findings
kinds: portable 3

**Best three implementations** (evidence-ranked):
1. **gemini-cli** (gemini-cli-e3, portable) -- Oversized tool output distilled at execution time: raw saved to disk, placeholder in context, LLM structural summary only for huge outputs `_(packages/core/src/scheduler/tool-executor.ts:207-212)_`
2. **crab-code** (crab-code-9, portable) -- Oversized tool outputs spill to temp files with a 2k-char in-band preview pointing Read at the full text `_(crates/tools/src/executor.rs:15-45)_`
3. **forge** (forge-10, portable) -- Truncation of oversized shell stdout/stderr and fetch bodies writes the FULL output to a temp file (disable_cleanup) and hands the model `_(crates/forge_app/src/tool_executor.rs:67-110)_`

**HOTDOG POSITION.** absent.

### trim-only-context-window -- 3 subjects / 3 findings
kinds: anti-pattern 3

**Best three implementations** (evidence-ranked):
1. **binharic-cli** (binharic-cli-8, anti-pattern) -- Real-path context management is drop-oldest-message trimming past 80% of context, with a token count that switches to a 0.4-tokens-per-char heuristic `_(src/agent/context/contextWindow.ts:7)_`
2. **darce-cli** (darce-cli-3, anti-pattern) -- Compaction is middle-drop with a count-only placeholder summary, triggered on chars/4 estimate against a hardcoded 100k that ignores the shipped `_(src/core/conversation.ts:4-31)_`
3. **zeroclaw** (zeroclaw-e2, anti-pattern [conf None]) -- Zero LLM-summarization compaction anywhere: every tier is lossy whole-turn drop; unique among study subjects in having no summarize step `_(crates/zeroclaw-runtime/src/agent/history_trim.rs:1-2)_`

**HOTDOG POSITION.** absent.

### unauthenticated-control-plane -- 3 subjects / 3 findings
kinds: safety-hole 3

**Best three implementations** (evidence-ranked):
1. **codel** (codel-2, safety-hole) -- GraphQL/websocket API that spawns containers and runs terminal commands has no auth `_(backend/router/router.go:31)_`
2. **devon** (devon-4, safety-hole) -- FastAPI control plane binds 0.0.0.0 with zero auth, exposing session-start and event-injection endpoints that drive arbitrary local bash `_(devon_agent/__main__.py:41)_`
3. **openlumara** (openlumara-6, safety-hole) -- api_bridge defaults api_key_required=False and executes every request with commands_authorized=True `_(channels/api_bridge.py:40-44,186)_`

**HOTDOG POSITION.** absent.

### windows-sandbox-gap -- 3 subjects / 3 findings
kinds: safety-hole 3

**Best three implementations** (evidence-ranked):
1. **workground2** (workground2-s2, safety-hole) -- Fail-open to unwrapped execution on Windows and on bwrap-less Linux `_(internal/sandbox/seatbelt_other.go:16-27,44-47)_`
2. **deepseek-reasonix** (deepseek-reasonix-s7, safety-hole) -- the effective mode is force-off even when the user's config says enforce, and the entire safety-under-bash story (jail, egress proxy, forbid-read) `_(internal/contract/config/config.go:1004-1007)_`
3. **zeroclaw** (zeroclaw-s4, safety-hole [conf None]) -- No kernel-enforced boundary on Windows `_(crates/zeroclaw-runtime/Cargo.toml:104-106)_`

**HOTDOG POSITION.** absent.

### zero-verification-snapshot -- 3 subjects / 3 findings
kinds: anti-pattern 3

**Best three implementations** (evidence-ranked):
1. **codemachine-cli** (codemachine-cli-2, anti-pattern) -- 55,434 LOC, zero test files, CI runs nothing but tag-gated binary builds `_(.github/workflows/build.yml:3-7)_`
2. **openlumara** (openlumara-10, anti-pattern) -- 1597 commits, 29k LOC, zero automated tests and no CI; the one test-named file is an interactive TTY viewer `_(channels/turn_grouping_test.py:1-31)_`
3. **free-code** (free-code-4, anti-pattern) -- 1,914 source files, 199 LOC of tests, 1 commit `_(FEATURES.md:280)_`

**HOTDOG POSITION.** absent.

### reviewer-retry-loop -- 2 subjects / 3 findings
kinds: unique 1, portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **auto-code-rover** (auto-code-rover-1, portable) -- reproducer executed with AND without the candidate patch, reviewer judges patch and test separately, feedback routed to whichever agent was wrong `_(app/api/review_manage.py:104-167)_`
2. **SWE-agent** (SWE-agent-6, unique) -- LLM reviewer/chooser retry loop with cost-aware budget allocation across attempts `_(sweagent/agent/agents.py:257,307-310)_`

**HOTDOG POSITION.** absent.

### aci-tool-design -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **SWE-agent** (SWE-agent-1, unique) -- ACI as a first-class artifact: documented interface concept with in-repo implemented tool bundles `_(docs/background/aci.md:1-14)_`
2. **auto-code-rover** (auto-code-rover-5, nuance) -- 8 tree-sitter-backed semantic APIs (search_class/method/in-file, get_code_around_line) instead of shell or grep tools, with arity asserted at dispatch `_(app/agents/agent_search.py:24-38)_`

**HOTDOG POSITION.** absent.

### acp-bidirectional -- 2 subjects / 2 findings
kinds: unique 2

**Best three implementations** (evidence-ranked):
1. **goose** (goose-e7, unique) -- goose is a full ACP agent server (stdio/HTTP/WS, session load/fork/list/delete, permission, usage updates) AND an ACP client provider that mounts `_(crates/goose/src/acp/server.rs:44-60)_`
2. **bitfun** (bitfun-e9, unique) -- bitfun serves ACP over stdio (openbitfun acp) AND hosts five rival harnesses as ACP clients with one-click npm installers `_(src/crates/interfaces/acp/src/server.rs:85-93)_`

**HOTDOG POSITION.** absent.

### acp-server-surface -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **workground2** (workground2-e10, portable) -- initialize/authenticate plus session new/load/resume/prompt/set_model/set_mode/list/delete/cancel, with framed-size cap and serialized writes `_(internal/acp/protocol.go:1-5)_`
2. **zeroclaw** (zeroclaw-e9, portable [conf None]) -- ACP server implements the real Zed Agent Client Protocol v1 with a durable session store and structured cancellation sentinels `_(crates/zeroclaw-channels/src/orchestrator/acp_server.rs:1)_`

**HOTDOG POSITION.** absent.

### agent-directive-file -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **groq-code-cli** (groq-code-cli-5, portable) -- Deterministic no-LLM /init project-context generator with env-var overrides `_(src/utils/context.ts:35-80)_`
2. **codemachine-cli** (codemachine-cli-5, portable [conf med]) -- Agents steer the workflow control loop by writing .codemachine/memory/directive.json {action: loop|checkpoint|stop|continue}, read by per-behavior `_(src/workflows/directives/reader.ts:13)_`

**HOTDOG POSITION.** absent.

### cache-miss-attribution -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **deepseek-reasonix** (deepseek-reasonix-e2, unique) -- PrefixShape hashes system/tools and a cumulative BodyChain hashes each message, CompareShape merges provider usage with session-drained rewrite `_(internal/runtime/agent/cache_shape.go:14-24)_`
2. **workground2** (workground2-e3, portable) -- Prefix-shape hashing across turns turns cache misses into explained events, with per-provider cache-usage normalization on both wire formats `_(internal/agent/cache_shape.go:16-31)_`

**HOTDOG POSITION.** absent.

### cancellation-race-hygiene -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **open-codex** (open-codex-2, portable) -- Esc-esc cancellation defended by generation counter, dual stream/exec abort controllers, and synthetic tool outputs replayed for unresolved call_ids `_(codex-cli/src/utils/agent/agent-loop.ts:88-92,100-107,371-385,113-127)_`
2. **opensquilla** (cancellation-race-hygiene, nuance) -- 0.25s grace for user Stop vs 5s for timeouts, bounded vs must_settle policy, and run_turn's finally shields cleanup tasks through their own `_(src/opensquilla/engine/cancellation.py:13,17-18)_`

**HOTDOG POSITION.** absent.

### classifier-model-routing -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **waveloom** (waveloom-12, portable) -- 'proplan' routing: plan-mode runs the pro model, everything else flash, with a 200k-context downgrade guard and a never-leak-sentinel invariant `_(pkg/agentloop/loop.go:152)_`
2. **zap-coding-agent** (zap-coding-agent-11, unique) -- Per-turn task classifier swaps the provider/model for one turn from user-declared model_routes, then restores on exit -- including on every `_(src/session/routing.rs:12-27)_`

**HOTDOG POSITION.** absent.

### compaction-continuity-gate -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **grinta-coding-agent** (grinta-coding-agent-1, unique) -- Post-compaction semantic fact-survival gate with blocking loss categories `_(backend/context/continuity_eval.py:20-36,57-58)_`
2. **forge-norvialabs** (forge-norvialabs-3, portable) -- user-authored constraints/corrections/decisions/irreversible-actions are extracted from canonical history, fed to the checkpoint prompt as must-keep `_(crates/forge-context/src/compaction/facts.rs:1-40)_`

**HOTDOG POSITION.** absent.

### compaction-hooks -- 2 subjects / 2 findings
kinds: portable 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **maki** (maki-5, portable) -- agent.compact.prepare lets extension layers re-pick which tool results the summarizer sees collapsed; host applies the edit `_(maki-agent/src/agent/compaction.rs:503-563,44)_`
2. **codewhale** (codewhale-e3, anti-pattern) -- No pre/post-compaction hook events; image budget is a flat ~1k estimate `_(crates/hooks/src/lib.rs:28-60)_`

**HOTDOG POSITION.** absent.

### compaction-pricing -- 2 subjects / 2 findings
kinds: unique 2

**Best three implementations** (evidence-ranked):
1. **deepseek-reasonix** (deepseek-reasonix-e3, unique) -- a context-carried spend accumulator sums every summarizer call's usage ('a discarded answer is still a charge'), foldEconomics refuses folds whose `_(internal/runtime/agent/compact_accounting.go:13-31)_`
2. **opensquilla** (opensquilla-e5, unique) -- Compaction is itself budgeted: LLM-call cap, deadline, quality report `_(src/opensquilla/session/compaction.py:1376-1392)_`

**DESIGN TENSION.** Compaction bills itself: deepseek-reasonix-e3 (context-carried spend accumulator sums every summarizer call -- 'a discarded answer is still a charge'; foldEconomics refuses folds whose savings do not cover the routine) and opensquilla-e5 (LLM-call cap, deadline, quality report on the compaction itself). Two subjects, same thesis: the compactor is a cost center and must be budgeted like a subagent. Related to cache-monotonic economics.

**HOTDOG POSITION.** absent.

### context-economy-guardrails -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **orca-agent** (orca-agent-e5, portable) -- PreCompact/PostCompact/OnBudgetWarning hooks, 8192-token history image budget, 2KB stale-tool-output micro-compact, 1MB bounded head+rolling-tail `_(crates/orca-core/src/hook_types.rs:10-14)_`
2. **opensquilla** (opensquilla-e6, portable) -- Single-window budget governor with per-class tool argument/result caps `_(src/opensquilla/context_budget.py:66)_`

**HOTDOG POSITION.** absent.

### coverage-regression-gate -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **nanocoder** (nanocoder-8, portable) -- PR checks fail on coverage drops and CI runs the bwrap jail spec on real bubblewrap `_(.github/workflows/pr-checks.yml:17-21)_`
2. **prime-agent** (prime-agent-10, nuance) -- Diff-based test-presence policy gate instead of coverage threshold `_(scripts/check-test-policy.mjs:1-25)_`

**HOTDOG POSITION.** absent.

### crash-exit-diagnostics -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **oh-my-pi** (oh-my-pi-c6, unique) -- Per-tool start markers plus typed session-exit records with pending-tool-call diagnostics written on normal, signal, fatal, and process-exit teardown `_(packages/coding-agent/src/session/exit-diagnostics.ts:22-45)_`
2. **mistral-vibe** (mistral-vibe-c7, nuance [conf med]) -- No faulthandler or native-crash hook anywhere despite a native-Rust second backend; Python-level observability is otherwise solid `_(vibe/observability/logging.py:57-108)_`

**HOTDOG POSITION.** absent.

### declared-parity-lineage -- 2 subjects / 2 findings
kinds: nuance 2

**Best three implementations** (evidence-ranked):
1. **kimi-code** (kimi-code-b5, nuance) -- Vendored pi-tui ruled a library dependency, not lineage: documented pin, MIT, intent cards, own CI job; no pi god-file pattern inherited `_(packages/pi-tui/UPSTREAM.md:10-16)_`
2. **ob-1** (ob-1-11, nuance) -- 11 src modules carry a first-line 'parity with claw-code's X' header - original Apache-2.0 TypeScript, but the feature list is deliberately modeled `_(src/safety/policy.ts:1,)_`

**HOTDOG POSITION.** absent.

### default-on-os-jail -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **workground2** (workground2-s1, portable) -- Default-on OS jail with fail-safe mode normalization `_(internal/config/config.go:1180-1187,)_`
2. **deepseek-reasonix** (deepseek-reasonix-s1, portable) -- empty/unknown config normalises to enforce, per-command Seatbelt/bwrap wrap, and enforce-without-backend refuses the command instead of running it `_(internal/contract/config/config.go:1004-1017)_`

**DESIGN TENSION.** The affirmative half of the sandbox doctrine at small scale: deepseek-reasonix-s1 (empty/unknown config normalises to ENFORCE; enforce-without-backend refuses the command rather than running unwrapped) and workground2-s1 (default-on jail with fail-safe mode normalization). Cross-reference confine-or-refuse-startup and permission-policy: this is where the shipped-default doctrine gets implemented instead of argued.

**HOTDOG POSITION.** absent.

### dns-pinned-ssrf-guard -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **openlumara** (openlumara-4, portable) -- HTTP module resolves DNS, validates every resolved address (incl. IPv4-mapped IPv6 unwrapping) against a full blocklist, and re-validates on redirects `_(modules/http.py:673-723,987-1012)_`
2. **maki** (maki-15, portable) -- Every extension-facing HTTP vets DNS, pins the vetted IP for connect, and re-validates each redirect hop by hand `_(maki-lua/src/api/net.rs:183-187,277,431-433,692)_`

**HOTDOG POSITION.** absent.

### docs-as-shipped-artifact -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **grok-build** (grok-build-e17, portable) -- The user guide is a shipped product artifact with a drift-gated config reference and documented error copy `_(config_docs/mod.rs:3-7)_`
2. **zeroclaw** (zeroclaw-e12, portable [conf None]) -- 217-page mdbook with a generated-documentation pipeline; spot-checked pages match the code exactly `_(src/main.rs:1629)_`

**DESIGN TENSION.** Docs as build output: grok-build-e17 (user guide shipped with the binary, config reference drift-gated with the config as its declared source), zeroclaw-e12 (217-page mdbook from a generated pipeline, spot-checked against code). Counterweight: the readme-capability-drift and phantom-docs clusters. hotdog: CONTEXT.md planned/shipped discipline verified true -- in spirit, unrecorded here.

**HOTDOG POSITION.** absent.

### docs-dx-mass -- 2 subjects / 2 findings
kinds: nuance 2

**Best three implementations** (evidence-ranked):
1. **opencode** (opencode-e13a, nuance) -- Docs/DX is broad and CI-gated - 36 top-level product pages x 25 locales (614 mdx), locale-sync workflow, AGENTS.md style guide, CONTEXT.md domain `_(AGENTS.md:1-30)_`
2. **aider** (aider-e10, nuance) -- 84 in-repo doc pages incl. 17 provider guides and 7 troubleshooting pages, in-app errors that carry actionable next steps, deprecation shims, and an `_(base_coder.py:1400-1414)_`

**HOTDOG POSITION.** absent.

### docs-match-code -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **codewhale** (codewhale-e12, portable) -- 100-file docs tree that makes falsifiable claims the code demonstrably keeps; 20 README locales; did-you-mean errors; doctor `_(docs/CACHE.md:9-14,63-71)_`
2. **opensquilla** (opensquilla-e16, nuance) -- 64 in-repo docs including design specs, and spot-checks match code `_(docs/features/compaction-and-cache.md:22-24)_`

**DESIGN TENSION.** Small but pointed: codewhale-e12 (docs make FALSIFIABLE claims the code demonstrably keeps -- docs/CACHE.md records even rejected old behaviors) and opensquilla-e16. The docs-drift-gates cluster is the enforcement mechanism; phantom-safety-control is what unenforced drift becomes at the extreme.

**HOTDOG POSITION.** absent.

### documented-omissions -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **codewhale** (codewhale-s7, portable) -- Exceptional documentation of what the safety model does NOT enforce, embedded at the code surface and under test `_(crates/tui/src/sandbox/mod.rs:12-21)_`
2. **hax** (hax-7, unique) -- A philosophy doc answering 'why not X' for each shipped-elsewhere feature (MCP, hooks, permission prompts, slash commands, deps), including an `_(docs/philosophy.md:39-113)_`

**HOTDOG POSITION.** absent.

### dual-copy-parity-test -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **codebuff** (codebuff-b5, portable) -- Forced algorithm duplication guarded by a parity test that runs both copies over the same histories `_(packages/agent-runtime/src/compact-history.ts:14-23)_`
2. **3code** (3code-b6, portable) -- In-process tool gate reimplements the OS backend's path canonicalization and is drift-guarded by named traversal regressions `_(src/threecode/sandbox.nim:450-497)_`

**HOTDOG POSITION.** absent.

### dual-loop-migration -- 2 subjects / 2 findings
kinds: nuance 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **opencode** (opencode-c1, anti-pattern) -- Both agent loops are shipped live: legacy V1 while(true) and the V2 durable runner serve the same server `_(packages/opencode/src/session/prompt.ts:1088)_`
2. **qwen-code** (qwen-code-c8, nuance [conf med]) -- Two full TUIs coexist: Ink ui/ and opentui/ (34k non-test LOC) with 'ink-parity' maintained by hand `_(live-session.ts:182-199)_`

**HOTDOG POSITION.** absent.

### embedded-internal-markers -- 2 subjects / 2 findings
kinds: license-risk 2

**Best three implementations** (evidence-ranked):
1. **free-code** (free-code-2, license-risk) -- Anthropic-internal Slack channel ID in system-prompt source proves non-public origin `_(free-code/src/constants/prompts.ts:245)_`
2. **kode-cli** (kode-cli-1, license-risk [conf med]) -- Anthropic-internal USER_TYPE branches ('ant', 'SWE_BENCH') in readable TypeScript prove Claude-Code-internals origin, contradicting the cline-lineage `_(packages/core/src/engine/query-executor.ts:22)_`

**HOTDOG POSITION.** absent.

### embedded-wal-datastore -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **kimi-code** (kimi-code-8, unique) -- minidb: pure-Node embedded KV store (WAL group-commit, skip-list indexes, torn-tail truncation) backing agent persistence `_(packages/minidb/README.md:8-30)_`
2. **smelt** (smelt-7, portable) -- Sessions live in a content-addressed SQLite lineage store with operator tools `_(src/main.rs:837-936)_`

**HOTDOG POSITION.** absent.

### expired-compat-shim-layer -- 2 subjects / 2 findings
kinds: nuance 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **hermes-agent** (hermes-agent-c6, nuance) -- Decomposition compat layer is 12 days past its announced removal date at head: ~2,000-entry shim still ships and old-path plugins are already `_(COMPAT_MANIFEST.md:8)_`
2. **opensquilla** (opensquilla-e18, anti-pattern) -- Retired-feature churn inside one release cycle risks doc lag `_(CHANGELOG.md:8-19)_`

**HOTDOG POSITION.** absent.

### extension-owned-policy -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **pi** (pi-5, unique) -- Permission gates, destructive-confirm, repo guards shipped as example extensions over a blocking beforeToolCall `_(packages/agent/src/agent-loop.ts:739-740)_`
2. **smelt** (smelt-3, nuance) -- Compaction is entirely a bundled Lua plugin against two engine contracts `_(runtime/lua/smelt/plugins/compact.lua:638,687)_`

**HOTDOG POSITION.** absent.

### external-coding-agents-as-tools -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **letta-code** (letta-code-e5, unique) -- Claude Code driven over its stream-json stdout contract and Codex driven over `codex app-server --stdio`, with session resume and MCP-server `_(src/tools/impl/external-coding-agent.ts:13)_`
2. **zeroclaw** (zeroclaw-e11, portable [conf None]) -- Rival-harness adapters spawn external coding CLIs with env-cleared allowlists, including driving Grok's CLI headlessly over its ACP surface `_(crates/zeroclaw-tools/src/coding_cli.rs:9-21)_`

**HOTDOG POSITION.** absent.

### fail-open-scanner-default -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **goose** (goose-s2, anti-pattern) -- Prompt-injection/command-classifier defense is off by default, and when on, classifier and inspector failures degrade silently open `_(crates/goose/src/security/mod.rs:72)_`
2. **hermes-agent** (hermes-agent-s5, anti-pattern) -- Optional tirith external scanner defaults fail-open, and an unreadable config also reads as fail-open `_(tools/approval_context.py:308-315)_`

**HOTDOG POSITION.** absent.

### generated-code-loc-census -- 2 subjects / 2 findings
kinds: nuance 2

**Best three implementations** (evidence-ranked):
1. **amazon-q-developer-cli** (amazon-q-developer-cli-b6, nuance) -- 203,219 of ~280k repo LOC are smithy-generated clients - zero architecture credit, zero god-file blame `_(crates/amzn-codewhisperer-client/src/lib.rs:48)_`
2. **opensquilla** (opensquilla-b3, nuance) -- Census undercounts test LOC by ~77k and counts generated contracts toward non-test `_(recount: src/*.py=567,497, tests/*.py=723,277 across 1,516 test_*.py, )_`

**HOTDOG POSITION.** absent.

### god-object-config -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **qwen-code** (qwen-code-c3, anti-pattern) -- Config is a 12.2k-LOC god object threaded through every module `_(packages/core/src/config/config.ts:12173)_`
2. **gemini-cli** (gemini-cli-c4, anti-pattern) -- Config is a 4249-LOC hub implementing both McpContext and AgentLoopContext for every subsystem `_(packages/core/src/config/config.ts:757)_`

**HOTDOG POSITION.** absent.

### harness-adapter-contract -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **aeon** (aeon-1, unique) -- One Claude-Code-shaped headless contract over nine rival coding-agent CLIs `_(harness-adapter/harnesses.json:1-10)_`
2. **codemachine-cli** (codemachine-cli-3, nuance) -- Seven-rival-CLI adapter matrix as the whole product, but without aeon's capability manifest or contract tests `_(src/infra/engines/providers/claude/execution/commands.ts:15-40)_`

**HOTDOG POSITION.** absent.

### heuristic-task-completion -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **cursor-agent** (cursor-agent-8, anti-pattern) -- Task termination decided by prose phrase-matching ('in conclusion' + 'all requirements') with an exact terminate sentinel as the only reliable signal `_(cursor_agent_tools/interact.py:652-691)_`
2. **trae-agent** (trae-agent-b5, anti-pattern) -- Task completion decided by substring match on 'done'/'task finished' with base validator always true `_(trae_agent/agent/base_agent.py:259-270)_`

**HOTDOG POSITION.** absent.

### landlock-self-sandbox -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **3code** (3code-3, portable) -- Landlock/Seatbelt/restricted-token FS policy on every tool call, dual enforcement `_(src/threecode/sandbox.nim:26-31)_`
2. **octomind** (octomind-7, unique) -- In-process kernel jail applied to self before any child spawn and inherited by shells/MCP/subagents: Landlock (Linux) / seatbelt (macOS), write-only `_(src/sandbox/mod.rs:15-40)_`

**HOTDOG POSITION.** absent.

### leaked-proprietary-snapshot -- 2 subjects / 2 findings
kinds: license-risk 2

**Best three implementations** (evidence-ranked):
1. **free-code** (free-code-1, license-risk) -- full proprietary Claude Code source redistributed with no license `_(free-code/package.json:1-4)_`
2. **workground2** (workground2-b1, license-risk) -- Committed Codex Desktop session JSONL embeds full Codex base_instructions and codex_app tool schemas `_(upstream-hardening-complete-execution-log-2026-07-10.txt:13)_`

**HOTDOG POSITION.** absent.

### master-context-recovery -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **gptme** (gptme-2, unique) -- Lossy views over an append-only master log with byte-range recovery references embedded in every truncated message `_(gptme/util/master_context.py:26,62)_`
2. **ferrum** (ferrum-10, nuance) -- history_search/history_read tools query the session JSONL including pre-compaction archived messages, returning line numbers for follow-up reads `_(src/agent/mod.rs:2348-2365,4094)_`

**DESIGN TENSION.** The 'recall instead of compact' corner the brief asked about: gptme-2 (lossy views over an APPEND-ONLY master log with byte-range recovery references embedded in every truncated message -- nothing is ever destroyed, only projected) and ferrum-10 (history_search/history_read tools query the session JSONL including pre-compaction archives, returning line numbers). This is the alternative answer to history loss the compact-harder majority never takes; two subjects is not a camp, but it is the seed of one. hotdog: compaction is destructive here -- absent.

**HOTDOG POSITION.** absent.

### mcp-prompt-skills -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **crab-code** (crab-code-11, portable) -- MCP server prompts auto-converted into namespaced skills at connect `_(crates/agents/src/mcp_skills.rs:1-40)_`
2. **keen-code** (keen-code-4, nuance) -- Each connected MCP server auto-generates a managed skill (mcp:<server>) containing a tool table; full JSON schemas load on demand - no `_(internal/mcpskills/mcpskills.go:15-49,51-63)_`

**HOTDOG POSITION.** absent.

### message-bus-host-split -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **neovate-code** (neovate-code-6, portable) -- Every product surface - Ink TUI, programmatic SDK, ACP server, web+websocket server - is a thin client of one typed MessageBus/HandlerMap RPC `_(src/nodeBridge.ts:3-19)_`
2. **continue** (continue-c5, portable) -- Typed tuple-protocol messenger cleanly binds webview/core/IDE across in-process, IPC and TCP transports `_(core/protocol/webview.ts:11)_`

**HOTDOG POSITION.** absent.

### mid-turn-user-inbox -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **g3** (g3-7, unique) -- Filesystem mailbox lets external processes inject user messages into a running turn: one file per message in .g3/sessions/<id>/inbox/, drained at the `_(crates/g3-core/src/pending_input.rs:1-31,82-102)_`
2. **grok-build** (grok-build-c12, portable) -- Running turns accept versioned steering: queued-prompt interjection, priority notification injection, cancel-and-send `_(run_loop.rs:1124)_`

**HOTDOG POSITION.** absent.

### model-provenance-attestation -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **gptme** (gptme-10, unique) -- Durable attestation of which model was requested, how aliases resolved, and what evidence exists about the serving backend `_(gptme/model_attestation.py:1-15,30-40)_`
2. **aeon** (aeon-7, nuance) -- Signed run manifests via GitHub artifact attestation, with proof re-runs against a PR's attested head `_(.github/workflows/aeon.yml:1364-1470)_`

**HOTDOG POSITION.** absent.

### parallel-engine-migration-tax -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **kimi-code** (kimi-code-b2, anti-pattern) -- Phantom CI job survives the v1-engine removal: test-vscode-legacy sets KIMI_CODE_LEGACY_FLAG=1 that no code reads `_(.github/workflows/ci.yml:72-88)_`
2. **opencode** (opencode-b7, anti-pattern) -- Two session engines ship simultaneously with bidirectional bridge imports `_(server/routes/instance/httpapi/handlers/session.ts:11)_`

**HOTDOG POSITION.** absent.

### phantom-artifact-save -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **coro-code** (coro-code-4, anti-pattern) -- --trajectory-file logs 'Trajectory saved to' but nothing is ever written; --must-patch writes a literal placeholder file `_(cli/src/commands/run.rs:57)_`
2. **mini-kode** (mini-kode-4, anti-pattern) -- Session persistence module exists but saveSession/loadSession have zero call sites -- sessions live only in the React reducer and die at exit; no `_(src/sessions/persistence.ts:28,41)_`

**HOTDOG POSITION.** absent.

### project-settings-self-grant -- 2 subjects / 2 findings
kinds: safety-hole 2

**Best three implementations** (evidence-ranked):
1. **claurst** (claurst-4, safety-hole) -- repo-shipped .claurst/settings.json can inject shell hooks and permission rules while sibling fields are explicitly protected `_(core/src/lib.rs:1962)_`
2. **mini-kode** (mini-kode-b2, safety-hole) -- Model self-grants global bash by writing .mini-kode/permissions.json - the grant store sits inside the auto-approved directory `_(src/permissions/policyResolver.ts:143-145)_`

**HOTDOG POSITION.** absent.

### provider-plane-fork -- 2 subjects / 2 findings
kinds: nuance 2

**Best three implementations** (evidence-ranked):
1. **open-codex** (open-codex-1, nuance) -- Fork delta is exactly the transport swap Responses-API -> Chat-Completions plus a five-provider key/baseURL/model table; loop and sandbox `_(codex-cli/src/utils/config.ts:39-125)_`
2. **open-interpreter** (open-interpreter-4, nuance [conf med]) -- fork work concentrates in transport/provider plane, not the loop `_(codex-rs/acp-server/; codex-api/src/anthropic.rs)_`

**HOTDOG POSITION.** absent.

### read-before-edit-hash-guard -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **darce-cli** (darce-cli-6, portable) -- EditTool refuses edits to files not previously Read, enforced via a readFiles Set on ToolContext, and it is tested `_(src/tools/EditTool.ts:27-29)_`
2. **mini-kode** (mini-kode-9, portable) -- fileEdit refuses any update unless a sha256 of the on-disk file matches the cached hash from the last read -- staleness enforced, not just `_(src/tools/fileEdit.ts:154-163)_`

**HOTDOG POSITION.** absent.

### reasoning-chain-carryover -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **forge** (forge-2, unique) -- Compaction extracts the LAST reasoning_details from the compacted range and injects it into the first surviving assistant message, preserving `_(crates/forge_app/src/compact.rs:109-172)_`
2. **grok-cli** (grok-cli-8, nuance) -- Encrypted-reasoning marker detection hides opaque reasoning from display and summary inputs `_(src/agent/reasoning.ts:4-9)_`

**HOTDOG POSITION.** absent.

### retry-blind-backoff -- 2 subjects / 2 findings
kinds: anti-pattern 2

**Best three implementations** (evidence-ranked):
1. **trae-agent** (trae-agent-b4, anti-pattern) -- Every provider exception retried 10x with 3-30s random sleeps, including deterministic context-overflow errors, on append-only history `_(trae_agent/utils/llm_clients/retry_utils.py:34-50)_`
2. **auto-code-rover** (auto-code-rover-4, anti-pattern) -- tenacity retry wraps the whole model call and re-fires deterministic failures (context_length_exceeded) 3 times at 30-600s random backoff before the `_(app/model/common.py:127)_`

**HOTDOG POSITION.** absent.

### sandbox-request-failopen -- 2 subjects / 2 findings
kinds: anti-pattern 1, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **codewhale** (codewhale-s4, anti-pattern) -- A requested sandbox silently degrades to unsandboxed when no backend is available `_(crates/tui/src/sandbox/mod.rs:564-576)_`
2. **zeroclaw** (zeroclaw-s1, safety-hole [conf None]) -- Sandbox fails open to NoopSandbox; only Seatbelt has a fail-closed variant `_(crates/zeroclaw-runtime/src/security/detect.rs:350)_`

**HOTDOG POSITION.** absent.

### self-modification-tiering -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **ouroboros** (ouroboros-9, unique) -- Runtime-mode tiers gate which source surfaces the agent may rewrite, with consent-tested elevation and constitution-protected files `_(ouroboros/runtime_mode_policy.py:1-6)_`
2. **san** (san-6, nuance [conf med]) -- Agent self-learning writes memory and new skills under explicit SkillPermissions bounds `_(internal/selflearn/config.go:14-27)_`

**HOTDOG POSITION.** absent.

### server-mode-permission-deferral -- 2 subjects / 2 findings
kinds: safety-hole 2

**Best three implementations** (evidence-ranked):
1. **ra-aid** (ra-aid-8, safety-hole) -- Server binds 0.0.0.0 by default with an unauthenticated /v1/spawn_agent; cowboy+server merely prints a warning `_(ra_aid/__main__.py:489-490)_`
2. **openharness** (openharness-4, safety-hole) -- oh mcp-server exposes all tools with toolPermission rules unenforced `_(docs/ecosystem-compat.md:24)_`

**HOTDOG POSITION.** absent.

### session-writer-lease -- 2 subjects / 2 findings
kinds: unique 2

**Best three implementations** (evidence-ranked):
1. **hermes-agent** (hermes-agent-c2, unique) -- Durable cross-process turn lease with TTL refresher and race-safe liveness watchdog gates every turn `_(agent/turn_facade_lease.py:1-7)_`
2. **qwen-code** (qwen-code-c5, unique) -- Cross-process session writer lease with boot-id + PID-namespace liveness fencing and inode-verified lock dirs `_(packages/core/src/services/session-writer-lease.ts (3125 LOC): readLoc)_`

**HOTDOG POSITION.** absent.

### shadow-mode-guardrail -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **gptme** (gptme-5, portable) -- Deterministic confused-deputy blocker shipping in shadow mode by default with promote-to-enforce via env knob `_(gptme/hooks/guardrails.py:1-40,220)_`
2. **ipsupport-code** (ipsupport-code-1, portable) -- Embedded offline-trained linear risk classifier scores every tool call in shadow mode by default, logging disagreements with the permission policy `_(internal/risk/shadow.go:40-92)_`

**DESIGN TENSION.** Ship enforcement in shadow before enforcing: gptme-5 (deterministic confused-deputy blocker, shadow by default, promote-to-enforce via env knob) and ipsupport-code-1 (offline-trained linear risk classifier scoring every tool call in shadow, logging disagreements with the live policy, blocking nothing). The corpus's answer to 'how do you turn enforcement on without breaking users' -- small cluster, high roadmap value. hotdog: absent.

**HOTDOG POSITION.** absent.

### side-channel-readonly-queries -- 2 subjects / 2 findings
kinds: unique 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **kimi-code** (kimi-code-10, unique) -- btw: side-question agent on a lightweight instance, read-only tools, tool defs kept visible to preserve prompt cache `_(packages/agent-core-v2/src/features/btw/btw.ts:3-20)_`
2. **keen-code** (keen-code-8, nuance) -- /btw side-question runs a tool-less one-shot over a last-10-message window alongside the main stream, bypassing the 5-item input queue `_(internal/appstate/state.go:243-258)_`

**HOTDOG POSITION.** absent.

### speculative-compaction -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **bitfun** (bitfun-e2, portable) -- 10k-token prefetch spawns a side-effect-free summary candidate; commit re-validates request identity, rebases against the canonical suffix, re-checks `_(src/crates/execution/agent-runtime/src/compression_prefetch.rs:8-14)_`
2. **oh-my-pi** (oh-my-pi-e4, unique) -- Compaction is speculated in the background ahead of the threshold and armed for instant commit, with growth refresh, failure latching and UI state `_(packages/coding-agent/src/session/session-maintenance.ts:335-343)_`

**DESIGN TENSION.** DESIGN TENSION: pay compaction latency at the threshold vs before it. Speculation pole: oh-my-pi-e4 (compaction speculated in background ahead of threshold, armed for instant commit, growth refresh + failure latching), bitfun-e2 (10k-token prefetch spawns a side-effect-free summary candidate; commit LINEARIZES: re-validates request identity, rebases against the canonical suffix), grok-build-e2 (two-pass prefires pass-1 below the hard threshold with prefix-fingerprint invalidation -- vtcode-e3 is its consumer-with-no-producer twin, filed anti-pattern). Consensus risk to watch: speculation without a rebase rule is a correctness bug.

**HOTDOG POSITION.** absent.

### subagent-budgeting -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **workground2** (workground2-e8, portable) -- Subagents with inherited compaction/pricing, capped depth, child step budgets, persisted transcripts with continuation/fork refs and an interrupted `_(internal/agent/task.go:75-77)_`
2. **opensquilla** (opensquilla-e9, portable) -- Subagent governance: shared depth cap, child-owned physical budgets, task-size caps `_(src/opensquilla/agents/limits.py:10)_`

**DESIGN TENSION.** Budget shapes for delegated work: opensquilla-e9 (shared depth cap, child-OWNED physical budgets, task-size caps) vs workground2-e8 (inherited compaction/pricing ratios, capped depth, child step budgets, persisted transcripts with continuation/fork refs and an interrupted status). Consensus-leaning: depth caps everywhere, cost inheritance is the refinement. hotdog: no token budgets on tasks -- below this bar (absent).

**HOTDOG POSITION.** absent.

### tamper-evident-audit-chain -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **dvalincode** (dvalincode-3, portable) -- Hash-chained per-run audit log seeded with the policy hash, plus an offline verify command and an install-specific `trust` self-report `_(src/audit/log.ts:19-40,81-88,191)_`
2. **molt** (molt-6, nuance) -- Hash-chained session journal with offline `molt verify`, whose header states the honest bound itself: 'tamper EVIDENCE, not tamper prevention' `_(src/journal.ts:1-16)_`

**HOTDOG POSITION.** absent.

### tests-prove-properties -- 2 subjects / 2 findings
kinds: portable 2

**Best three implementations** (evidence-ranked):
1. **vtcode** (vtcode-s10, portable) -- ~9.9k inline #[test] + 325 async tests asserting behavior, not existence; census test_loc is a severe undercount `_(crates/codegen/vtcode-safety/src/command_safety/dangerous_commands.rs:614-619)_`
2. **gemini-cli** (gemini-cli-s14, portable) -- chained &&/|| gating, nested command substitution denial, forbidden-path-after-allow ordering, integrity determinism `_(packages/core/src/policy/shell-safety-regression.test.ts:35-56)_`

**HOTDOG POSITION.** absent.

### three-transport-mcp-client -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **dexto** (dexto-e9, portable) -- MCP client with all three official transports plus a dual-transport MCP server exposing any agent as a tool `_(packages/core/src/mcp/mcp-client.ts:161)_`
2. **roo-code** (roo-code-e10, nuance) -- Solid MCP client (stdio + SSE + streamable-HTTP, config hot-watch, per-tool allowlists, variable injection) but no MCP server mode and no ACP `_(src/services/mcp/McpHub.ts:5-8)_`

**HOTDOG POSITION.** absent.

### tool-bundle-install -- 2 subjects / 2 findings
kinds: portable 1, nuance 1

**Best three implementations** (evidence-ranked):
1. **SWE-agent** (SWE-agent-2, portable) -- Tools as self-contained bundles (bin/ + install.sh + config.yaml) uploaded and installed into the runtime at setup `_(sweagent/tools/tools.py:252-275)_`
2. **trae-agent** (trae-agent-7, nuance) -- Tools pyinstaller-frozen on the host, copied into the container at /agent_tools, with host<->container path translation on arguments `_(trae_agent/cli.py:101-115)_`

**HOTDOG POSITION.** absent.

### tool-disclosure-tiers -- 2 subjects / 2 findings
kinds: unique 1, portable 1

**Best three implementations** (evidence-ranked):
1. **mocode** (mocode-10, portable) -- Immutable per-step tool-policy snapshots with explicit add_tool_groups expansion and an optional small-model router `_(src/agent/run-coordinator.ts:166-192)_`
2. **jazz** (jazz-7, unique [conf med]) -- External doors (peers, webhooks) share one authorization model: a disclosure tier intersects the toolset down, and anything riskier than read-only `_(packages/core/src/types/disclosure-tier.ts:1-50)_`

**HOTDOG POSITION.** absent.

### turn-provenance-tagging -- 2 subjects / 2 findings
kinds: nuance 2

**Best three implementations** (evidence-ranked):
1. **prime-agent** (prime-agent-6, nuance) -- semantic-edges-v1: commit-gated causal edge ledger over model requests `_(packages/coding-agent/src/core/semantic-edges.ts:6-27)_`
2. **crush** (crush-7, nuance [conf med]) -- Turns carry their originating channel in context so remote-originated turns are distinguishable `_(internal/agent/channel.go:5-15)_`

**HOTDOG POSITION.** absent.

### two-pass-background-prefire -- 2 subjects / 2 findings
kinds: unique 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **grok-build** (grok-build-e2, unique) -- Two-pass compaction prefires pass-1 in the background below the hard threshold, with prefix-fingerprint invalidation `_(xai-grok-shell/src/session/compaction.rs:59-66)_`
2. **vtcode** (vtcode-e3, anti-pattern) -- Prefire two-pass compaction has a full consumer and no producer: nothing in any runtime ever runs pass-1 `_(crates/codegen/vtcode-core/src/compaction/auto.rs:146-148)_`

**HOTDOG POSITION.** absent.

### verbatim-proprietary-prompt -- 2 subjects / 2 findings
kinds: license-risk 2

**Best three implementations** (evidence-ranked):
1. **claw-code-agent** (claw-code-agent-2, license-risk) -- proprietary prompt strings and Claude branding shipped verbatim `_(src/prompt_constants.py:46-49)_`
2. **open-interpreter** (open-interpreter-b2, license-risk) -- Rival agents' system prompts and compaction prompts reproduced near-verbatim inside an Apache-2.0 repo `_(core/src/harness/claude_code_prompt.rs:5-32)_`

**HOTDOG POSITION.** absent.

### verification-ceiling -- 2 subjects / 2 findings
kinds: nuance 1, anti-pattern 1

**Best three implementations** (evidence-ranked):
1. **opencode** (opencode-s12, anti-pattern) -- tool-permission tests stub ctx.ask, so no end-to-end 'rejected -> process never spawned' spec through the real service `_(packages/opencode/test/tool/shell.test.ts:164-166)_`
2. **oh-my-pi** (oh-my-pi-s12, nuance) -- no fuzzing, no in-CI model evals, no coverage gate, and the CI itself admits branch protection and rulesets are off on main `_(.github/workflows/ci.yml:578)_`

**HOTDOG POSITION.** absent.

### yolo-no-floor -- 2 subjects / 2 findings
kinds: portable 1, safety-hole 1

**Best three implementations** (evidence-ranked):
1. **grok-build** (grok-build-s1, portable) -- Always-approve is a prompt waiver, not a policy waiver: deny rules, hooks, protected-target floors and a managed yolo-pin all survive bypass mode `_(docs/user-guide/22-permissions-and-safety.md:40)_`
2. **gemini-cli** (gemini-cli-s8, safety-hole) -- YOLO with the (default) disabled sandbox preserves dangerous-command and redirection decisions with nothing underneath `_(packages/core/src/policy/policy-engine.ts:352-359)_`

**HOTDOG POSITION.** absent.

## Appendix: concepts observed in exactly 1 subject (517)

One line each: concept - subject - mechanism clause [record ids]. Grouped by kind (concepts with mixed-kind records grouped under their dominant kind). Nothing invisible; these are roadmap raw material, not convergence claims.

### unique (142 concepts)

- **a2a-peer-door** - jazz - A2A server door on the existing trust model, tested against the official SDK's parser [jazz-b6]
- **a2a-plus-foreign-harness** - dexto - No anchor ships an A2A protocol server, and the only foreign-harness touchpoint is protocol-level: dexto drives the external Codex app-server over [dexto-e10]
- **ablation-pair-eval** - memcode - The product's central claim ships with a counterfactual harness: same task run A/B in isolated git worktrees, context-pack vs cold [memcode-8]
- **advisory-observer-lane** - nausicaa-harness - Teto: a second cognitive lane that observes only completed owner steps and may interrupt via A2A only when advice could change the next decision [nausicaa-harness-2]
- **agent-driven-ui-control** - openhands - Client tool lets the agent steer the human's UI: server executor is a no-op, frontend dispatches canvas_ui_control actions [openhands-4]
- **agent-flow-diagrams** - kimi-cli - Agent flows declared as D2/Mermaid fenced blocks in SKILL.md, parsed into task/decision graphs; the ralph loop is just a synthesized 2-node flow [kimi-cli-3]
- **agentless-pipeline** - agentless - Anti-agent three-stage funnel: hierarchical localize -> top-N one-shot sampled repair -> test-arbitrated rerank, chained by JSONL artifacts [agentless-1]
- **airgap-ssh-relay** - agentty - Air-gapped mode: agent runs on an offline host, bytes relay over `ssh -R 1080` SOCKS5 with TLS pinned end-to-end so the relay cannot MITM [agentty-7]
- **anti-vacuous-test-gate** - molt - Shipped builtin check rejects completions whose new tests prove nothing: tautologies (assert.equal(f(x),f(x))), assertion-free its, and new test [molt-2]
- **approval-free-kernel-fence** - 3code - Anti-approval thesis implemented as product: no approval code exists anywhere in the tool path; the default-on kernel fence is the sole enforcement [3code-b1]
- **async-remote-subagents** - deepagents - AsyncSubAgentMiddleware launches background subagent tasks on remote Agent Protocol servers, returning task IDs immediately so the main agent keeps [deepagents-b2]
- **background-task-adopt** - hax - Bash commands outliving their foreground window are adopted into a task registry owning the process tree, a drainer thread, a spool file, and a [hax-5]
- **budgeted-delegation** - hermes-agent - Subagents bounded at every axis -- depth-1 flat by default, concurrency cap, per-child timeout and iteration budget with execute_code refunds [hermes-agent-e6]
- **cache-discipline-as-observable-property** - jcode - Sliding READ/WRITE Anthropic breakpoint strategy with a stated 4-breakpoint budget, per-model-generation OpenAI cache-TTL handling, and a three-layer [jcode-e3]
- **cache-evidence-release-gate** - nausicaa-harness - Prompt-cache performance is an observed ledger projection and a release gate: recorded live probe artifact with provenance yields pass/hold on [nausicaa-harness-4]
- **cache-impact-ci-gate** - workground2 - CI gate that blocks PRs touching cache-sensitive prompt/tool surfaces unless they carry a cache-impact + guard-test note, plus a release-time [workground2-e2, workground2-b2]
- **cache-warm-model-placement** - hotdog - spawn candidates are ordered by a cached llama-swap /running peek (loaded models beat all), gate-failed retries run as warm follow-up turns on the [hotdog-6]
- **checkpoint-log-projection** - mistral-vibe - Rewind engine is an append-only event log (turns, manual edits, keep/revert decisions) re-projected into file state, with dependency-closed per-hunk [mistral-vibe-c5]
- **classifier-fail-closed** - qwen-code - LLM permission classifier verified fail-closed: API/timeout/schema/context failures fall back to manual approval, never to allow [qwen-code-s2]
- **cli-internal-boundary** - tura - gateway hosts the router, runtime workers are spawned subprocesses driven over a stdin/stdout JSON line envelope, and child-agent dispatch goes [tura-12]
- **code-as-actions-loop** - ra-aid - CIAYN backend: tool calls are model-written Python expressions validated by ast.parse, with LLM-based tool-call extraction as per-model rescue [ra-aid-3]
- **command-arity-attention-patterns** - opencode - LLM-generated command-arity dictionary collapses shell commands into human-scoped approval patterns [opencode-b1]
- **compaction-authority-ordering** - ferrum - Compaction re-appends immutable system/repo policy AFTER the generated summary so summarized text can never carry system authority [ferrum-2]
- **compaction-trigger-ground-truth** - hermes-agent - Compression trigger and effectiveness are judged on the provider's real post-response prompt-token count, not the rough estimate, with an explicit [hermes-agent-e1]
- **compile-time-exhaustive-policy-proof** - agentty - 48-cell trust matrix proved at compile time: exhaustive constexpr sweep of permission() vs an independently re-stated spec, enforced by static_assert [agentty-b2]
- **compression-evidence-lanes** - octomind - deterministic evidence selection around the generative fold -- packet kinds, provenance enum (RealUser/ToolObserved/ValidatedSummary) [octomind-3]
- **computed-blast-radius-deny-tier** - jcode - Two-stage blast-radius gate wired unconditionally into the bash tool: deterministic tokenizer-based risk cascade (Safe/Low/Confirm/Catastrophic) [jcode-s1]
- **consequence-class-grants** - memcode - Unattended agents run under canonical-hashed delegation policies expressed as consequence classes plus hard budgets, expiry, and revocation [memcode-9]
- **containment-derived-grants** - codebuff - Capability grant is computed from measured containment, with Windows's unsandboxed 'floor' as its own type, grant, and env [codebuff-6]
- **content-addressed-summary-cache** - orca-agent - Remote summary calls are content-addressed on scope+purpose+previous-summary+delta and persisted under ORCA_HOME, so identical re-compactions [orca-agent-e4]
- **context-dehydration-fragments** - g3 - post-compaction history is offloaded to linked fragment files and replaced by metadata stubs the model can selectively rehydrate via a capacity-gated [g3-1]
- **context-epoch-cache-baseline** - opencode - Immutable per-session system-context baseline (Context Epoch) held for provider-cache stability, with reconcile/replace semantics [opencode-c4]
- **counterfactual-artifact-promotion** - octomind - Self-generated guardrails/skills promote only via counterfactual A/B: turns where a shadow artifact's trigger matched but was not applied form the [octomind-11]
- **cross-harness-session-resume** - jcode - Session import/resume from rival harnesses -- claude-code, codex, opencode, pi, cursor -- with byte-bounded transcript folds so an imported history [jcode-e9]
- **cross-process-lane-ledger** - hotdog - Machine-wide provider-concurrency as a filesystem counting semaphore: atomic exclusive-create slots, rename-onto reclaim with read-back, mtime [hotdog-3]
- **delegation-prefix-identity** - mocode - Sub-agents reuse the parent's exact wire prefix - byte-equality asserted by test [mocode-13]
- **destructive-discard-guard** - prime-agent - Destructive-git dirty-tree guard with cd-chain resolution and explicit-intent bypass [prime-agent-2]
- **docs-as-governed-corpus** - deepseek-reasonix - Documentation is a governed corpus with a machine-checked standard: every path matches a CODEOWNERS primary+backup rule, every doc carries an [deepseek-reasonix-e10]
- **edit-ledger-injection** - zap-coding-agent - Structured edit ledger (file, ops count, last turn) is rebuilt from session state and injected into the system prompt every turn, surviving [zap-coding-agent-13]
- **effort-routing-dial** - kolkrabbi - one setting simultaneously selects model tier, per-turn tool-round budget, shell timeout, and orchestration task width [kolkrabbi-9]
- **egress-authority-separation** - deepseek-reasonix - Network egress is three separate decisions - external network (allow-listed domains via host egress proxy, with per-host ask outside the list), host [deepseek-reasonix-s3]
- **embedding-capability-routing** - octomind - user intent embedded locally (ONNX muvon/octomind-embed) and matched against hand-authored capability triggers (mean-of-top-K cosine + margin); a hit [octomind-8]
- **estimate-calibration** - agentty - Byte-based token estimate is calibrated online by EMA against the provider's real usage counts before it gates compaction [agentty-9]
- **event-driven-daemon** - nanocoder - Per-project daemon running cron + file-watcher triggered skill runs through a backpressure dispatcher [nanocoder-4]
- **event-sourced-session-core** - opencode - V2 session state is a durable per-aggregate sequenced event journal with projector, not a message table [opencode-c2]
- **execution-budget-engine** - orca-agent - Typed execution budgets with soft-landing and non-double-spending child leases: turns/tool-calls/cost/wall-time independently optional, Stopped [orca-agent-e6]
- **exposure-aware-tool-output-attenuation** - jcode - agentgrep attenuates results using a harness-built model of what the agent has already been exposed to -- read ranges, prior grep output, trace [jcode-e4]
- **fanout-scope-grant** - atomic-agent - one operator approval lets fusion workers write and run commands inside one named directory for exactly one turn - grants cleared at turn START so an [atomic-agent-5]
- **fleet-graph-restart-reissue** - deepseek-reasonix - The fleet tool dispatches 2-64 sub-agent tasks as a declared dependency graph with disjoint write_path claims checked in preflight [deepseek-reasonix-e5]
- **fork-annotation-governance** - kilocode - CI-blocking kilocode_change annotation check on every shared-upstream-path PR plus 30-minute upstream release watcher [kilocode-b4]
- **free-tier-model-router** - ob-1 - Keyless free-tier provider plane as a first-class architecture: ~20 free cloud providers routed in-process with strategy-ordered selection, 8-hop [ob-1-1]
- **frozen-fact-turn-recovery** - bitfun - permission mode, resolved model, and a model-binding fingerprint are persisted at interruption and the resume hard-refuses if any changed [bitfun-c3]
- **frozen-kernel-seam** - minicode - 1.6k-LOC agent kernel frozen, vendored into the product via #minicore subpath imports, seams additive-only, sync hash-pinned and CI-checked [minicode-8]
- **full-context-singlepass** - open-codex - whole repo flattened into one prompt using per-path cumulative size maps to budget directory-level truncation [open-codex-9]
- **generation-fenced-rehydration** - zeroclaw - Reap-and-rehydrate of daemon sessions is fenced by transcript generations and killed-markers, with a stale-refresh rejection test [zeroclaw-c7]
- **grammar-constrained-tool-calls** - atomic-agent - array-only root kills first-token bias into single-object form, per-request grammar narrowing [atomic-agent-2]
- **grounded-guard-evolution** - octomind - Learned guardrails require quote-backed user authorization, replay cases, and shadow trials [octomind-b6]
- **guardian-llm-approval** - hermes-agent - an auxiliary-LLM guardian adjudicates flagged commands, injection-hardened and fail-toward-human, with a 3-strike denial breaker [hermes-agent-s4]
- **harness-constant-benchmark** - kolkrabbi - pre-registered benchmark methodology that holds the model constant to measure the harness, with the failed local-8B pilot published as its own finding [kolkrabbi-10]
- **harness-self-optimization** - openharness - MetaHarness hill-climbs its own config: benchmark, LLM-suggest change, re-run, revert if worse [openharness-12]
- **hot-reload-session-continuity** - jcode - Daemon self-reload execs a new binary in place with per-session recovery records, GC, and restart snapshots [jcode-c3]
- **in-flight-turn-resume** - workground2 - InFlightTurnMeta persists the turn boundary and Continue() restarts the unfinished model/tool round without duplicating the user turn [workground2-c2]
- **incident-grounded-retry-budgets** - jcode - Turn-loop retry and cancel budgets are tuned to named production incidents with regression guards [jcode-c5]
- **journal-driven-state-machine** - goose - Generic 1,455-LOC engine crate (goose-agent, zero dependency on goose) runs the turn loop as ordered Operations over typed effects, reloading the [goose-c1]
- **leader-follower-ipc-daemon** - grok-build - One leader process per machine owns all agent state; clients attach/detach over a namespaced UDS protocol [grok-build-c10]
- **leaderboard-fallback-model** - ra-aid - Tool-call failure fallback chain seeded from a frozen 2025-02 snapshot of the Berkeley gorilla function-calling leaderboard [ra-aid-7]
- **leak-detection-production-corpus** - oh-my-pi - Model-token leakage defense (harmony-leak) with signal-fusion detection, truncate-and-resume recovery, and a test corpus extracted from real [oh-my-pi-s7]
- **leaked-call-repair** - hotdog - leaked Hermes/antml/chatML call markup in content or reasoning is grammar-parsed into real tool calls mid-loop, and a corrupt stored tool call [hotdog-12]
- **llm-compaction-judge** - code - once 150k tokens in a 1M window, a cheap aux model is consulted each turn with a pressure-band + projected-post-turn-risk JSON payload to decide [code-e2]
- **llm-patch-selection** - trae-agent - Research harness: a separate sandboxed selector agent adjudicates N candidate patches with grouped majority voting [trae-agent-6]
- **lua-scriptable-core** - smelt - Whole agent surface scriptable in Lua, generated and drift-gated [smelt-8]
- **macro-command-graph** - tura - One macro tool (command_run) executes a stepped command graph per LLM turn: same-step reads run concurrently, mutating steps are barriers under [tura-1]
- **manifest-gated-product-surface** - openhands - Product surfaces render only what an admitted extension manifest declares - no manifest, no nav entries, routes 404 [openhands-6]
- **marker-alias-mangling** - hotdog - tool-call/thinking/summary tags, WireFormat-owned names, and model control tokens are rewritten to random m_<16-char> aliases before the model ever [hotdog-1]
- **measured-cache-policy** - hermes-agent - Per-provider-family cache_control marker planner with measured before/after hit-rates in the comments, asserted nightly by secrets-gated live [hermes-agent-e3]
- **memory-export-server** - memcode - Offline MCP stdio server exposes the repo's memory to rival harnesses — switch clients, not agents [memcode-10]
- **memory-reflection-trees** - ob-1 - Project memory as a sqlite graph with depth-capped grounded reflection: a LLM-managed evolution pass (ADD/UPDATE/DELETE/NOOP with id-validation) plus [ob-1-14]
- **model-retrain-regression-gate** - ipsupport-code - Monthly model-retrain CI refuses to open its own PR if held-out precision/recall regresses >0.05, with cross-language feature parity pinned by shared [ipsupport-code-2]
- **model-visible-token-budget** - openlumara - At 80% of context the model itself receives a budget warning and is instructed to tell the user to /compress [openlumara-3]
- **openai-compat-app-server-plus-sdk** - letta-code - Agents are re-exposed as an OpenAI Responses-compatible HTTP endpoint (/v1/responses) with a versioned protocol package and shipped typed client [letta-code-e10]
- **operation-journal-invariants** - orca-agent - Per-operation append-only journal enforces ordering invariants on append and crash-recovers atomically: torn final line, non-contiguous ordinals [orca-agent-c4]
- **orphan-free-tool-pairing** - hotdog - Tool-call/result pairing integrity is enforced at every truncation seam: finish_reason=length refuses to execute any call [hotdog-11]
- **peer-egress-containment** - jazz - ask_peer forces the outbound question through a single string parameter and types the reply as a quotation, not a fact [jazz-9]
- **phase-pipeline-loop** - hermes-agent - one _LoopState dataclass declares every loop local, ~30 turn_*.py phases take named kwargs and return verdict dataclasses copied back by field name [hermes-agent-c1]
- **policy-tier-hardening** - gemini-cli - extension ALLOW rules stripped, admin override ignored when system policy exists, system dirs skipped unless secure, integrity hashes per policy dir [gemini-cli-s5]
- **portable-agent-bundle** - zot - Zotfiles: downloadable portable agents with enforced permission manifests and consent receipts [zot-1]
- **preregistered-placebo-eval** - nausicaa-harness - frozen fixture+scorer hashes, four arms including a shadow placebo, per-pair budget envelopes, 95% bootstrap [nausicaa-harness-3]
- **preview-suffix-continuation** - waveloom - Detects 'preview' assistant text ending in a colon/arrow/ellipsis with no tool call and grants exactly one grace turn to re-issue it [waveloom-10]
- **priced-fold-economics** - octomind - Mid-turn compaction gated by a provider-priced economic inequality [octomind-b1]
- **protocol-level-cache-budgeting** - opencode - default-on auto breakpoint policy with cost math, Anthropic 4-breakpoint cap enforced with drop counters, 5m/1h TTL buckets, and promptCacheKey gating [opencode-e3]
- **provenance-anchored-compaction** - octomind - PACT: runtime owns pins, provenance and attribution checks around the LLM fold [octomind-b5]
- **re-derivable-fix-proof** - dvalincode - verdict from observed exit codes + independent re-scan, carrying its own gate rule so a third party can re-reach it; v1 keeps verifying under v1 [dvalincode-4]
- **recoverability-tiered-permissions** - san - Permission model splits confirmation into unrecoverable vs git-recoverable tiers, with a breaker that survives bypass [san-1]
- **remote-control-surface** - grok-cli - Telegram remote control of a live CLI session: pairing codes, voice STT, streamed previews [grok-cli-7]
- **render-anti-spoofing** - hotdog - the mangler protects the model's eyes, spoof.ts protects the human's - bidi overrides/isolates, zero-widths, C0/C1, BOM become visible counted [hotdog-2]
- **reversible-summarization-cutoff** - openlumara - Compaction writes a SUMMARIZATION_CUTOFF marker instead of deleting history - model view is cut, user history survives, deletion of the marker reverts [openlumara-2]
- **rival-agent-scan** - gptme - Process-scan detection of concurrent rival agents (claude, codex, aider, goose, opencode, amp) sharing the workspace, lock-free by design [gptme-13]
- **rival-harness-adoption** - workground2 - Codex CLI wrapped as a first-class provider with --json stream normalization, and .claude/.agents convention dirs plus CLAUDE.md read natively [workground2-e12]
- **rival-harness-wire-emulation** - open-interpreter - Whole transport plane reshapes the codex engine into 14 rival coding-agent contracts [open-interpreter-b1]
- **rotation-stable-cache-scope** - hermes-agent - Content-addressed prompt_cache_key derived from a compression-lineage root, so the provider cache bucket survives compaction session-rotation and [hermes-agent-e2]
- **sbfl-context-seeding** - auto-code-rover - Optional spectrum-based fault localization (685-LOC SBFL engine running coverage over the test suite) injected as an advisory prompt the search agent [auto-code-rover-6]
- **security-table-contract-test** - ferrum - Published security.md tier-by-command matrix is pinned by test (tier_capability_contract_is_table_driven) [ferrum-b5]
- **self-authored-tools** - claude-engineer - Model writes new tool classes into tools/ at runtime, hot-reloaded via package rescan [claude-engineer-1]
- **self-healing-session-store** - workground2 - Durability core: digest/revision-fenced snapshot writes, torn-tail log self-repair, cross-process session leases, depth-capped recovery branches [workground2-c3]
- **serializable-turn-state** - dexto - Turn loop is an explicit phase state machine with strict-zod serializable checkpoint states that refuse to snapshot mid-flight [dexto-c1]
- **server-hosted-agent-loop** - letta-code - The reasoning loop and conversation state are not in the harness at all: the default backend is a stream-consumer over the Letta API, with the [letta-code-c1]
- **skill-integrity-lockfile** - aeon - Content-addressed skill lockfile (eyebrow) gating post-admission capability drift, incl. egress host sets [aeon-4]
- **snapcompact-bitmap-archive** - oh-my-pi - Snapcompact archives discarded history as eval-tuned pixel-font PNG frames billed per provider image model - a no-LLM, no-network compaction tier [oh-my-pi-e5]
- **sni-pinned-egress-proxy** - kilocode - Network allowlist enforced by token-auth localhost proxy that inspects the TLS ClientHello and dials pre-resolved public-only addresses [kilocode-s5, kilocode-b2]
- **spec-first-generation** - developer - Two-phase plan-then-manifest-then-per-file generation: shared_deps doc + explicit file-path array drive the codegen [developer-1]
- **speculative-streamed-tool-execution** - tura - command_run steps execute speculatively against the still-streaming provider tool-call arguments, emitting streaming_partial updates and terminating [tura-2]
- **sqlite-code-knowledge-graph** - coro-code - CKG tool: tree-sitter-parsed code knowledge graph persisted in bundled-sqlite, shipped as one 796-LOC tool [coro-code-5]
- **state-lifetime-partitioning** - deepseek-reasonix - Host state is partitioned by lifetime into four named structs - turnRuntime replaced wholesale each Run, taskRuntime spanning one delivery scope [deepseek-reasonix-c1]
- **subagent-protocol-parity** - gemini-cli - Subagents as tools with one protocol across local, custom-loaded, and remote (A2A) executors [gemini-cli-e7]
- **survival-contract-schema** - codewhale - Survival contract: language-invariant survive-fields schema for compaction output, versioned, with live refusal enforcement [codewhale-e4]
- **tamagotchi-companion** - openharness - Cybergotchi: a virtual pet whose needs/personality are driven by live agent session events [openharness-13]
- **task-status-control-plane** - tura - task_status makes task state structured session data (task_group/status/task_type/compact_context) that mechanically drives prompt-manual selection [tura-4]
- **test-edit-guard** - ob-1 - during a correction round a write to a test-pattern path is REFUSED, after any observed failure it is allowed but flagged loudly in the transcript [ob-1-9]
- **textual-tool-call-recovery** - ob-1 - whole-message prose or serialized-JSON tool calls (anchored, arg-shaped, so explanatory text mentioning tools is not flagged) and false capability [ob-1-10]
- **thrash-aware-tool-eviction** - memcode - Eviction derived from a measured read-evict-reread thrash: superseded-read dedup (range-aware), decaying hot-path pins, budget-scaled keep counts [memcode-3]
- **three-mode-compaction-with-anti-signal-guard** - jcode - Three compaction modes (reactive 80% / proactive EWMA-growth projection / semantic embedding topic-shift) behind a shared anti-signal guard, with [jcode-e1]
- **token-estimator-feedback** - cline - Compaction budget self-corrects from the provider's measured input-token count [cline-b4]
- **tombstone-installer-redirect** - kimi-cli - final releases convert the CLI's own entry points into a redirector that downloads and runs the successor's installer, plus a tested tombstone package [kimi-cli-12]
- **tool-output-masking-fifo** - gemini-cli - Hybrid backward-scanned FIFO masking: 50k-token protection window + 30k prunable batch trigger, high-signal tools never masked [gemini-cli-e4]
- **tool-output-range-condensing** - octomind - Task-aware narrowing of oversized tool outputs by ONE cheap-model call selecting line ranges over a numbered copy, reconstructed verbatim from the [octomind-4]
- **trajectory-step-tagging** - trae-agent - LakeView: post-hoc LLM pass tags each recorded step with a fixed 8-label SWE taxonomy and dual-granularity summary [trae-agent-1]
- **transport-abstracted-exec** - kimi-cli - All fs+exec tools go through a kaos backend interface with local, SSH, and ACP-host implementations -- in ACP mode the agent's shell and file tools [kimi-cli-6]
- **tree-sitter-anchor-edits** - plandex - Ellipsis-anchored structured edits resolved against tree-sitter node maps with unique/fuzzy replacement fallbacks [plandex-1]
- **turn-class-prompt-tiering** - zap-coding-agent - casual greeting prompt (~12 tokens of persona), ~400-token SLM prompt, or full prompt -- with the compaction projection honoring the same class split [zap-coding-agent-12]
- **turn-exclusion-guard** - mistral-vibe - Explicit turn-vs-mutation exclusion inside the loop: act() refuses while any session 'holder' runs, with the TOCTOU window documented [mistral-vibe-c8]
- **turn-memory-projection** - keen-code - future-turn history carries placement-tagged tool ACTIVITY records, not results - raw outputs are json:"-" excluded and survive only in-session under [keen-code-1]
- **turn-step-decomposition** - zeroclaw - Turn engine decomposed into 25 single-purpose step modules behind a TurnCtx with per-route projection [zeroclaw-c1]
- **unexpected-stop-rejudge** - oh-my-pi - Terminal stops are classified by a small judge model and re-driven: 'says it will act, then ends without doing so' gets its own bounded retry budget [oh-my-pi-c7]
- **untrusted-input-framing** - qwen-code - single escape boundary for untrusted strings, classifier treats config hints as adversarial, user answers cryptographically bounded, WebFetch [qwen-code-s7]
- **untrusted-result-framing** - hermes-agent - MCP/browser/web tool results are threat-scanned and wrapped in forge-proof untrusted-data delimiters [hermes-agent-s7]
- **user-directive-ledger** - mocode - Deterministic intent/constraint ledger survives compaction independently of the summarizer [mocode-1]
- **vendored-fork-intent-cards** - kimi-code - Fork governance: UPSTREAM.md intent cards pin the vendor delta against a fixed upstream commit [kimi-code-7]
- **verification-corps** - opensquilla - ~646k test LOC, 1,691 safety-lane test functions asserting behavior, full offline pytest + real bwrap boundary probe on every PR [verification-corps]
- **verified-failure-escalation** - ob-1 - self-fix budget spent AND the check still red -> runTurn returns { escalate } handing the turn to Fusion best-of-N; pure shouldEscalate gate [ob-1-2]
- **wal-ofd-lockguard** - hermes-agent - OFD file locks guard live WAL generations against stray close() cancellation in sibling processes [hermes-agent-c3]
- **x402-agent-payments** - grok-cli - x402 wallet payments with BRIN security pre-scan hard-blocking low-score URLs [grok-cli-6]
- **zero-dependency-distribution** - hotdog - Zero runtime dependencies by construction with a written supply-chain posture: no install step, no build, no lifecycle hooks, bun.lock pins only [hotdog-13]

### portable (186 concepts)

- **acp-as-universal-surface** - grok-build - ACP is the product's automation contract: stdio and WebSocket-with-secret server modes, documented IDE compatibility matrix [grok-build-e13]
- **acp-first-class** - opencode - 12-file module with its own permission bridge, usage/profile reporting, and per-session MCP server registration, plus a native IDE install matrix and [opencode-e9]
- **acp-plane** - orca-agent - versioned-pin agent as `--mode=acp`, plus a multi-client local daemon with bridge and attach, single mutation lease per session, and [orca-agent-e9]
- **action-pin-consistency-test** - dvalincode - Offline meta-test asserting every third-party action's commit digest and its version comment never contradict each other across workflows [dvalincode-10]
- **actuals-based-context-projection** - dexto - Next-input token estimate is anchored on the previous call's real usage (lastInput+lastOutput+newMessages) with pure-estimate fallback and logged [dexto-e1]
- **adversarial-fixture-tests** - goose - Security tests build real attack fixtures and assert the negative behavior: repo-supplied fsmonitor hook execution, implicit bare-repo acceptance [goose-s7]
- **agents-docs-mesh** - bitfun - 65-file bilingual AGENTS.md mesh with per-directory exact verification commands whose stated numeric semantics verifiably match the code [bitfun-e12]
- **approval-bound-in-loop** - opencode - Shell tool parses the command into an AST and issues per-subcommand asks (bash patterns + external_directory globs) before spawning, tested for bash [opencode-s5]
- **approval-grammar-tested** - opencode - Wildcard last-match-wins permission grammar with default ask, unit-tested as an algebra (~60 decision-asserting specs) [opencode-s1]
- **approval-lifecycle-spec** - opencode - reject cascades per session, always-approvals persist and auto-resolve matches, requests isolated by directory, pending fails on dispose/reload/abort [opencode-s7]
- **ask-fails-closed** - kilocode - Unanswerable permission prompts fail closed everywhere: headless subagents Denied, ACP auto-rejects, shutdown rejects pending [kilocode-s11]
- **auto-debug-repair-loop** - plandex - Exec failure triggers capped rollback -> tell-with-output -> rebuild -> re-exec recursion [plandex-6]
- **autofix-edit-search** - binharic-cli - When an edit's search string fails to match, a side-model call repairs it under a verbatim-presence constraint before retrying; malformed edit JSON [binharic-cli-7]
- **backend-registry-multi-origin** - openhands - Multi-backend registry: one shell drives local/remote/cloud agent servers and different agents per backend, with version-compat gating [openhands-7]
- **background-run-stream-resume** - letta-code - Runs default to server-side background execution with client-side stall detection, timed run re-discovery, and stream resume that replays the missed [letta-code-c4]
- **bash-ast-rule-matching** - san - allow requires every subcommand, command substitutions are flattened so egress cannot hide in $() [san-2]
- **behavior-equivalence-gate** - tura - A CI job runs a Runtime Session equivalence gate against committed baselines with explicit failure-injection evidence artifacts uploaded each run [tura-10]
- **binary-hijack-var-block** - waveloom - LD_PRELOAD/DYLD_*/NODE_OPTIONS commands hard-denied ahead of every short-circuiting allow path [waveloom-6]
- **boomerang-delegation-stack** - roo-code - Subtasks are a parent-child delegation stack (Orchestrator/Boomerang mode): parent is flushed and disposed, child becomes sole active task, result [roo-code-e6]
- **bundled-diagnostics-command** - workground2 - doctor command (JSON-capable, path-redacting), per-turn cache-miss diagnostics, session trash/restore CLI, external crash-report worker [workground2-c8]
- **cache-safe-result-dedup** - smelt - Duplicate tool outputs replaced by pointers placed only on the new invocation [smelt-5]
- **cache-warm-resume-replay** - 3code - Sessions persist the verbatim wire bytes of every request (key order, extra fields, raw argument bytes) so a resumed session re-sends byte-identical [3code-10]
- **caller-allowlist-default-deny** - hermes-agent - pairing codes, no state for ignored strangers, per-profile gate isolation, timeout fail-closed approvals [hermes-agent-s9]
- **cgroup-process-containment** - ferrum - Per-spawn delegated cgroup-v2 with pre_exec attach and cgroup.kill for bash and MCP children [ferrum-8]
- **ci-culture** - opencode - CI: unit + Playwright e2e matrices on linux+windows, SHA-pinned actions, tsgo typecheck, HttpApi exerciser gates, generated-client drift check [opencode-s11]
- **ci-security-gates** - qwen-code - daily high-severity CVE hard gate, monthly OpenSSF Scorecard, nightly CodeQL, SHA-pinned actions, and a collaborator-trust precheck feeding the [qwen-code-s8]
- **circuit-breaker-scheduler** - aeon - Auto-recovering circuit breaker (closed/open/half-open probe) as the single source of the dispatch decision [aeon-6]
- **compile-enforced-protocol-parity** - codewhale - Engine Event/Op to protocol wire parity is compile-enforced by exhaustive no-wildcard match projections [codewhale-c4]
- **config-monotonic-tightening** - kilocode - Project-level config can only tighten the sandbox, never widen it [kilocode-s3]
- **config-scope-monotone-narrowing** - kilocode - Repo-sourced config can only tighten the jail: local scope keeps only enabled:true and network:deny, never allowed_hosts or writable_paths [kilocode-b3]
- **confinement-intersection** - kilocode - Subagent confinement is the set-intersection of parent and child snapshots; it can only narrow [kilocode-s4]
- **context-upgrade-ladder** - crab-code - Swap to a larger-context variant of the same model at 75% before resorting to compaction [crab-code-2]
- **cost-budget-autosubmit** - SWE-agent - Cost/context/budget ceilings never abort empty-handed: on hit, the agent autosubmits the partial patch [SWE-agent-7]
- **crash-durable-swarm-coordination** - jcode - Swarm state is durable with per-swarm serialization locks, swarm mutations are idempotent via hashed request keys, and a hot binary reload survives [jcode-e5]
- **crash-safe-json-persistence** - roo-code - Session JSON writes go through an advisory-locked temp-file + backup + rollback helper, and the history index self-reconciles [roo-code-c6]
- **crate-extraction-path-alias** - codewhale - Giant-crate extraction by git-mv plus one use-alias block and a cargo-graph boundary gate, with no dual tree on disk [codewhale-c3]
- **cross-surface-rewind-invariant** - hermes-agent - One rewind_user_turn implementation backs CLI/gateway/TUI /undo-/retry, with a test asserting all three surfaces leave the identical active message [hermes-agent-c5]
- **curl-replay-export** - groq-code-cli - Debug mode writes masked curl repro plus full request JSON per API call [groq-code-cli-6]
- **cursor-addressable-replay** - grok-build - Reconnect resumes mid-stream via eventId cursors into updates.jsonl, with a replay buffer in the actor loop [grok-build-c7]
- **daemon-separated-agent-loop** - jcode - Agent loop is process-separated from every product surface, and the dependency direction is CI-enforced [jcode-c1]
- **dangling-turn-repair** - kilocode - Interrupted/failed turns are repaired before the next prompt is accepted, with explicit resume semantics [kilocode-c4]
- **debug-dump-eval-asserts** - forge - Evals verify tool-use policy by asserting jq predicates on a request-context dump (FORGE_DEBUG_REQUESTS={{dir}}/context.json) written by the agent [forge-7]
- **debug-socket-surface** - jcode - Opt-in daemon debug socket exposes structured state and prompt-level introspection commands [jcode-c7]
- **declarative-ci-recipes** - picocode - YAML recipes pin provider/model/persona for non-interactive runs, with error_if output-gating for CI exit codes [picocode-1]
- **denial-receipt-escalation** - orca-agent - Escalation is user-approval-driven with structured grants (fs write roots, network domains; turn/session scope) via an opaque digest-bound router [orca-agent-s6]
- **diagnostic-doctor-command** - orca-agent - orca doctor: read-only versioned-schema diagnostics (config, trust, sandbox readiness) with JSON/text output and credential redaction, contract-tested [orca-agent-c6]
- **differential-parser-oracle** - kimi-code - Hand-written pure-TS bash parser gated by byte-for-byte differential tests against the official tree-sitter-bash 0.25.0 wasm reference, with a [kimi-code-b3]
- **dispatcher-side-sandbox** - aeon - Dispatcher applies its own bwrap sandbox around whatever harness runs, because no harness enforces read-only reliably [aeon-2]
- **distribution-fork-branding** - open-interpreter - centralized product-identity crate makes rebrand testable instead of search-and-replace [open-interpreter-3]
- **doc-tree-depth** - roo-code - 510-file docusaurus tree with per-tool references and feature pages that match live mechanisms [roo-code-e13]
- **docs-and-error-quality** - kilocode - In-repo feature doc tree plus contributor-facing discipline docs, and error messages written for recovery, not for logs [kilocode-e10]
- **docs-dx-tooling** - vtcode - generated config reference shared by-reference between docs and the live /config palette, schema-export CLI, did-you-mean errors, three-channel [vtcode-e11]
- **durability-loss-contract** - grok-build - Session persistence publishes an explicit hard-power-loss contract: awaited acks, barriers, fsync-before-rename, latched write errors [grok-build-c6]
- **durable-bounded-board** - kilocode - Subagent message board is sqlite-persistent with hard size caps and prompt-injection-aware protocol instructions [kilocode-e5]
- **durable-inbox-and-restart-honest-jobs** - deepseek-reasonix - A durable session-level instruction queue returns a receipt only after disk write, with its own recovery and idempotency store and an explicit [deepseek-reasonix-e6]
- **durable-run-orchestration** - workground2 - RunHub receipts with interrupted-write reconciliation, event-sourced Work DAG scheduler with per-task-ID flight admission, and a session-scoped [workground2-e7]
- **e2e-boundary-harness** - hermes-agent - Security tests prove properties against real process trees, not existence: 25 obfuscated rm variants with twin-run falsifiability [hermes-agent-s10]
- **edit-format-strategy-factory** - aider - Coder.create dispatches over a class registry, each format pairs a subclass with its own prompts module, and ChatChunks is a typed context model with [aider-c2]
- **enforcement-chokepoint** - opencode - All tool execution funnels one ctx.ask wired to Permission.ask with orDie; plan-mode deny, subagent deny-inheritance, doom-loop ask and ACP bridge [opencode-s8]
- **escalation-human-only** - kilocode - Sandbox-escalation and skill-shell asks cannot be auto-approved by any machine client [kilocode-s2]
- **fail-closed-escalation** - opensquilla - Backend denials never silently replay on the host; approvals are generation-bound and stale generations fail closed [fail-closed-escalation]
- **fail-closed-folder-trust** - orca-agent - Folder trust fails closed (unknown = untrusted) and untrusted folders downgrade the default sandbox to strict read-only no-network, property-tested [orca-agent-s11]
- **fail-closed-permission-skeleton** - goose - When enforcement is on, the permission skeleton fails closed and precedence is order-independent and tested: no decision -> needs_approval, judge [goose-s3]
- **fail-safe-crash-semantics** - oh-my-pi - interrupted tool calls are surfaced as resume warnings not silent replays, isolated subagents are never resurrected outside their worktree, and [oh-my-pi-s10]
- **failure-copy-engineering** - hermes-agent - predicate tables map exceptions and OAuth error codes to actionable sentences with the raw exception demoted to a Details line, and hermes doctor has [hermes-agent-e11]
- **fatal-crash-mirror** - deepseek-reasonix - Runtime-fatal output (panic on any goroutine, fatal runtime error) is mirrored into a per-PID 0600 file via debug.SetCrashOutput and queued for [deepseek-reasonix-c7]
- **fatal-exit-semantics** - letta-code - Fatal crash path sets the process exit code synchronously before any async work, drains telemetry under a 3s bound, and is verified by [letta-code-c7]
- **fault-localization-agent-tool** - auto-code-rover - Coverage-based SBFL fault localization exposed to the agent as a first-class tool [auto-code-rover-b3]
- **folded-file-context-in-summary** - roo-code - Post-condense summary re-injects tree-sitter signature-only folds of every file the agent read (default 50k-char cap), so code structure survives [roo-code-e2]
- **four-protocol-surfaces** - deepseek-reasonix - full-spec MCP client (stdio/streamable-http/SSE, OAuth with PKCE + protected-resource metadata + token rotation, registry browse/install), an ACP v1 [deepseek-reasonix-e8]
- **goroutine-leak-test-gate** - workground2 - goleak.VerifyTestMain guards the concurrency-critical packages; turn goroutines recover at the boundary that matters [workground2-c9]
- **harness-addressable-loop** - dexto - TurnExecutor.execute drives the loop through the same public step-callback interface its 4,456-LOC integration suite drives, so the real loop is [dexto-c3]
- **hermetic-multi-os-ci** - codewhale - Heavy multi-OS nextest CI with security rails; policy-filtered leg runs authority/sandbox tests directly [codewhale-s10]
- **history-change-retry-guard** - dexto - Model-request retry is refused unless the persisted conversation history is byte-for-length unchanged, preventing duplicate tool side effects on [dexto-c4]
- **honest-durability-checklist** - opencode - Runner carries a checked/unchecked implementation contract in its own docblock, and parity lives in one canonical spec [opencode-c6]
- **idempotent-turn-ingress** - opensquilla - unique index on (source_scope, request_session_key, client_request_id) with a read-first replay path that returns TurnAcceptanceResult(replayed=True) [idempotent-turn-ingress]
- **internals-docs-matched-code** - oh-my-pi - 85-file docs tree describes internals at mechanism depth and spot-checks match code; onboarding is multi-channel with live-generated completions and [oh-my-pi-e11]
- **interop-regression-ci** - dvalincode - CI drives the built binary with the reference MCP client over a real pipe, with a require-flag that turns a missing build into a failure; a separate [dvalincode-9]
- **journal-derived-approval-state** - goose - Tool-approval state is derived from the message journal, and confirmation persistence is idempotent: duplicate decisions no-op, conflicting decisions [goose-c4]
- **kanban-first-class-queue** - hermes-agent - SQLite kanban with WAL + BEGIN IMMEDIATE + compare-and-swap claims, worker dispatch, task graph, and per-board isolation [hermes-agent-e8]
- **kernel-enforcement-depth** - opensquilla - Real kernel backends on all three platforms with fail-closed selection and denial-depth tests [kernel-enforcement-depth]
- **layered-authorization-contract** - codewhale - Documented 9-layer authorization order with monotonicity enforced by tests at the owning layers [codewhale-s2]
- **layered-budgets-with-actionable-denials** - jcode - Runaway caps and budgets at every orchestration level -- absolute swarm membership cap, configurable live-worker concurrency default 32, adaptive [jcode-e6]
- **layered-loop-breakers** - oh-my-pi - Tool-call loop breaker ships in the provider-agnostic package and is retrofitted into the advisor's private loop, closing a bounded-run gap the [oh-my-pi-e8]
- **learned-context-window** - atomic-agent - a 400/413 naming the real window repacks the next prompt to it once [atomic-agent-6]
- **least-privilege-secret-injection** - aeon - Per-skill requires: secrets resolved from an allowlisted blob that is unset before the agent starts [aeon-3]
- **license-gated-skill-install** - openharness - /skill-install refuses non-permissive SPDX licenses without an explicit --accept-license flag [openharness-14]
- **llm-drift-gated-docs** - gemini-cli - Docs kept honest by a weekly Gemini-run audit workflow plus generated settings/keybindings references, all dogfooded via in-repo skills [gemini-cli-e11]
- **llm-permission-judge** - goose - LLM judges per-request read-only-ness with explicit injection discipline: request data labeled UNTRUSTED, embedded instructions declared [goose-s4]
- **local-stack-supervisor** - openhands - npm package ships a real process supervisor for its polyglot stack: shutdown-hook registry, port leases, stable 0600 keys, file logs [openhands-11]
- **loss-bounded-compaction** - workground2 - user turns kept verbatim, originals archived, digest never re-summarized, and a deterministic mechanical-fold fallback when the summarizer fails [workground2-e4]
- **mcp-client-depth** - workground2 - MCP client over stdio, Streamable HTTP and SSE with capability-gated prompts/resources, hot-add, a diagnostics package, and config import from rival [workground2-e11]
- **mcp-oauth-client** - amazon-q-developer-cli - Full OAuth dance implemented client-side for MCP servers (740 LOC) on top of rmcp with child-process/SSE/streamable-http transports [amazon-q-developer-cli-11]
- **mcp-secret-redaction** - openhands - MCP tool results and error text are scrubbed of configured secrets before reaching the UI [openhands-10]
- **mcp-step-transition-gate** - codemachine-cli - propose_step_completion (artifact path + sha256 + success-criteria checklist) -> schema validation -> approve_step_transition by a second agent [codemachine-cli-4]
- **middleware-exclusion-coverage-check** - deepagents - an exclusion entry matching nothing across main and subagent stacks raises, and required classes are unexcludable [deepagents-b3]
- **model-exchange-tracing** - bitfun - Versioned, per-field-capturable model request/response trace sink wired into every round -- the corpus's strongest analog to codex's debug [bitfun-c6]
- **no-llm-context-eviction** - qwen-code - Microcompaction blanks old tool results/images in place without any LLM round-trip [qwen-code-e2]
- **no-llm-economy-layers** - grok-build - Two no-LLM economy layers run before compaction: deterministic tool-result pruning and byte-budget image eviction [grok-build-e4]
- **no-llm-emergency-and-413-payload-recovery** - jcode - No-LLM hard-compact above 95% plus a distinct recovery path for provider 413 'request too large' driven by base64 images, with bounded image token [jcode-e2]
- **non-authoritative-denial-text** - orca-agent - Sandbox-denial strings parsed from child stdout/stderr are strictly diagnostic: labeled non-authoritative in the tool result and explicitly cannot [orca-agent-s7]
- **opt-in-retention-preview** - orca-agent - orca storage prunes only explicitly archived, unlocked, unchanged sessions and previews by default; retention refuses symlinks and non-regular files [orca-agent-c8]
- **orchestration-budgets** - grok-build - Layered hard budgets on orchestration: agent-call budget, host-call cap, fanout cap - with no-double-charge-on-cancel tests [grok-build-e9]
- **ordered-fit-ladder** - grok-build - Summarizer input is fitted by a strict five-rung ordered ladder with telemetry on which rung fired [grok-build-e3]
- **orphan-tool-settlement-on-resume** - opencode - On every drain, tools still projected as running from a previous process are durably failed before any model call [opencode-c3]
- **overflow-summary-graft** - plandex - On token overflow, search stored conversation summaries for the newest timestamp that fits, graft it, keep the tail [plandex-5]
- **panic-contained-turn-execution** - orca-agent - Every hosted generation runs inside catch_unwind and surfaces as a typed GenerationTaskOutcome::Panicked with usage accounting preserved around it [orca-agent-c9]
- **panic-to-crashlog** - code - Unified panic hook, session end logging, and bespoke forensics commands (doctor, order-replay, sandbox debug runners) [code-c8]
- **park-revive-subagent-crashrecovery** - oh-my-pi - parked agents survive process exit and are cold-revived from their session files by hub scan, collab mirror, or IRC wake [oh-my-pi-e7]
- **per-session-run-coordinator** - opencode - 105-LOC coordinator: one drain per session key, concurrent resumes join the active run, wakes coalesce, interrupt waits for cleanup [opencode-c5]
- **permission-coalescing** - crab-code - Concurrent teammate approvals coalesce into one prompt with broadcast decisions [crab-code-5]
- **persist-partial-on-cancel** - dexto - Cancellation persists the partial assistant response and synthesizes failed/cancelled tool results into history, then recovers the partial text from [dexto-c10]
- **policy-file-self-lock** - 3code - Hidden guard rules lock the two activatable policy files read-only inside every sandboxed launch, appended last so no policy text can weaken them [3code-b5]
- **policy-hot-reload** - 3code - Policy reloaded by mtime before every restricted operation AND re-loaded by each sandbox child, so a mid-session policy edit can only tighten what [3code-b8]
- **port-segregated-controller** - workground2 - One transport-agnostic Controller behind every frontend, consumed through 16 segregated sub-ports instead of the concrete struct [workground2-c1]
- **pr-comment-injection-sanitization** - codebuff - pull_request_target workflow sanitizes the attacker-controlled PR title before echoing it into a comment, and documents why it must never check out [codebuff-b4]
- **pre-mode-unbypassable-guards** - letta-code - Two guards evaluate before any mode/rule logic and survive even unrestricted mode: the workspace sandbox and the cross-agent memory guard, the latter [letta-code-s2]
- **principal-boundary** - opensquilla - guests and unauthenticated LAN traffic are always sandboxed and can never soft-land on the host, even when the owner default is Full [principal-boundary]
- **process-lifecycle-hygiene** - kilocode - pid-liveness + health + version-match stale detection, flock-serialized start, parent watchdog, quarantine with split command budgets [kilocode-e7]
- **prompt-cache-strategy-layer** - roo-code - MultiPointStrategy keeps cache-point placements stable across turns, enforces min-tokens-per-point, and only reshuffles placements when a new span [roo-code-e3]
- **prompt-preview-command** - codewhale - /preview-request: engine-owned, privacy-bounded reconstruction of the exact next outbound request [codewhale-c9]
- **property-asserting-test-corps-in-ci** - oh-my-pi - ~2,676 TS test files plus 282 Rust files with #[test], all wired into bucketed CI jobs alongside cargo-deny, clippy, rustfmt, native-addon smoke and [oh-my-pi-s11]
- **provider-anchored-token-accounting** - oh-my-pi - Token accounting anchors on the provider's own usage report and re-counts only the tail; a stored-conversation floor keeps the trigger honest under [oh-my-pi-e2]
- **provider-retry-discipline** - opencode - Retry policy honors retry-after-ms / retry-after seconds / HTTP-date, adds jittered exponential backoff with caps, and exempts ContextOverflowError [opencode-e6]
- **rate-limit-model-feedback** - waveloom - Session-level per-host ThrottleStore turns 429/403 into model-readable errors carrying earliest-retry time; subagents inherit the store [waveloom-9]
- **readonly-command-table** - workground2 - Shared fail-closed read-only command table feeds both gates [workground2-s5]
- **real-violation-tests-in-ci** - kilocode - Confinement violation tests execute real sandboxed processes in CI on macOS and Linux, including egress denial through the real spawner [kilocode-s9]
- **redact-at-ingress** - smelt - Secrets scrubbed from user input and tool results before entering history [smelt-6]
- **reflection-gate-justification** - jcode - a Confirm verdict is refused once and only unlocked by a model-supplied `justification` naming the user request; blind retry fails identically, bare [jcode-s2]
- **reproducible-release-builds** - 3code - container pinned by sha256 digest, apt frozen to a snapshot.ubuntu.com timestamp, sha256-pinned Nim toolchain, clean-rebuild byte-equality check [3code-5, 3code-b4]
- **rewind-safe-compaction** - roo-code - Condense and sliding-window truncation are non-destructive tag-hides (condenseParent/truncationParent + marker messages), so rewinding past a [roo-code-c4]
- **rival-artifact-compat** - orca-agent - Interop strategy includes consuming rival harnesses' artifacts rather than their code: plugin discovery reads .codex/plugins alongside .orca/plugins [orca-agent-e11]
- **rival-harness-compat-imports** - opencode - reads project CLAUDE.md and global ~/.claude/CLAUDE.md behind a disable flag, and ships per-tool migration docs for Claude Code, Cursor, and Windsurf [opencode-e12]
- **route-coverage-exerciser** - opencode - CI gate proves every public HTTP route has a decode/auth/mutation scenario [opencode-b2]
- **run-span-inspection-cli** - dexto - Every loop phase emits an OTel span with result attributes, and a CLI (dexto span list <runId>, dexto trace) fetches and formats run traces on demand [dexto-c9]
- **sandbox-by-default** - orca-agent - AutoEdit maps bash to WorkspaceWrite no-network; Plan is a read-only hard ceiling rules cannot lift; host shell only via explicit FullAuto [orca-agent-s1]
- **sandbox-escape-approval** - deepseek-reasonix - If the jail fails to start, running unconfined is a separate, per-escape human decision: a nil approver is fail-closed, typed-nil context values are [deepseek-reasonix-s2]
- **sandbox-escape-proof** - kolkrabbi - refusedByTheOS distinguishes kolk declining (exit -1), the pre-exec child guard (exit 125), and a real sandbox denial (platform refusal phrase) [kolkrabbi-5]
- **sandbox-fail-closed-optin** - kilocode - Opt-in jail that refuses to enable without a backend and fails at exec instead of falling back to unprotected sh [kilocode-b1, kilocode-s1]
- **sanitizer-gated-portable-ci** - hax - Push-gating CI carries ASan+UBSan, TSan, AND boot-VM FreeBSD/OpenBSD jobs that run the identical suite - sanitizer and port regressions block the [hax-b2]
- **semantic-contract-corps** - orca-agent - ~240k-LOC test corps asserting loop and policy semantics, not existence: root contracts spawn the real binary over the JSONL harness asserting [orca-agent-s13]
- **session-actor-command-loop** - grok-build - Session core is one actor on a dedicated OS thread, driven by an explicit typed command enum [grok-build-c1]
- **session-drain-graph** - kilocode - Refcounted session-drain graph with parent linking and cycle detection gates instance shutdown [kilocode-c5]
- **session-epoch-fencing** - opensquilla - Writes fence on (expected_session_id, expected_session_epoch): 52 threading sites, StaleEpochError raised at 7 points, ownership check shared by [session-epoch-fencing]
- **shared-pure-recovery-policy** - letta-code - All turn-recovery verdicts (pre-stream conflict class, retry-vs-rethrow, approval-recovery eligibility, denial rebuild) extracted into one pure [letta-code-c3]
- **shim-hosted-engine-reuse** - roo-code - it loads the VS Code extension headlessly via a vscode API shim and drives it over the existing webview-message protocol [roo-code-c2]
- **single-mutation-entry-point** - roo-code - MessageManager declared the SINGLE entry point for all conversation-message deletion, consolidating splice sites that the god-class had scattered [roo-code-c5]
- **single-owner-operation-host** - orca-agent - RuntimeHost/ThreadActor control plane enforces six ownership properties [orca-agent-c2]
- **single-persistence-layout-authority** - workground2 - A leaf store package owns every on-disk session-artifact path; package boundaries are guarded by import tests [workground2-c10]
- **sqlite-cron-compensation** - dexto - Scheduler is a genuine persistent-cron with write-compensation: schedules reload from the toolState store at boot and a failed delete rolls the list [dexto-e8]
- **ssrf-guarded-fetch** - workground2 - SSRF-guarded dialers on web_fetch and installer fetches [workground2-s15]
- **stale-approval-denial** - letta-code - On restart into a conversation with server-pending approvals, every host denies the stale tool calls with an explicit recovery reason instead of [letta-code-c8]
- **strict-sandbox-backends** - orca-agent - Linux sandbox = bwrap primary with landlock+seccomp in-process fallback; both production builders pin strict:true; no backend yields [orca-agent-s3]
- **structural-injection-defense** - codewhale - Structural injection defense: typed capped fragments, injection-hostile reviewer parser, autonomy-is-guidance constitution [codewhale-s6]
- **subagent-as-persisted-child-session** - mistral-vibe - Subagents are real child sessions with their own lease, saved under the parent's session dir, linked into parent history, with typed cancel-cleanup [mistral-vibe-e5]
- **subagent-coordinator** - grok-build - Central subagent coordinator: message wakes, backpressure permits, scoped cancellation, and foreground hand-off [grok-build-e12]
- **subagent-governance-bundle** - dexto - self-spawn block, per-parent concurrency cap, iteration cap, allow-list, and automatic reasoning-variant downgrade for cost containment [dexto-e6]
- **subagent-machinery** - bitfun - 6,449-LOC task tool family with 2,501 LOC of tests, hidden/background subagents with cancel handles cascading to descendants, swarm planner [bitfun-e5]
- **subagent-queue-recovery** - orca-agent - Async subagents run as detached persisted workers with a config-bounded admission queue and an in-session steering queue, all with recovery contracts [orca-agent-e8]
- **subagent-run-budgets** - oh-my-pi - Subagent runs carry soft request budgets with escalating enforcement and global concurrency/runtime caps; user-side +Nk turn budgets are parsed as [oh-my-pi-e6]
- **subagent-startup-context-budget** - letta-code - Reflection subagent startup context hard-capped at ~16k estimated tokens with graceful degradation: parent-memory section shrinks to the file-tree [letta-code-e4]
- **supply-chain-ci** - opensquilla - Unconditional fresh dependency security audit job on every PR: pinned pip-audit + npm audit with report-coverage assertions [supply-chain-ci]
- **support-diagnostics-bundle** - goose - versioned report with system info, capped config, CLI log tails, capped LLM request logs, prompt templates, and schedule state; plus an in-loop [goose-c5]
- **symlink-aware-path-containment** - oh-my-pi - Where containment is needed it is real: realpath-aware workspace confinement in TS and a Rust write-policy with symlink-escape refusal tests [oh-my-pi-s6]
- **test-coverage-completeness-checker** - letta-code - A meta-checker proves every test file is claimed by a CI shard, and the checkers themselves have tests -- closing the silent-untested-file hole that [letta-code-s9]
- **tested-approval-nothing-underneath** - oh-my-pi - Approval gate is in-loop, unconditionally installed, and its resolution semantics are property-tested: tier matrix, deny precedence, fail-closed [oh-my-pi-s1]
- **thread-goal-budgets** - bitfun - Thread-goal token budgets with BudgetLimited terminal status and per-round billable accounting that discounts cached tokens [bitfun-e7]
- **three-axis-run-budgets** - mistral-vibe - Turn, dollar, and token session budgets as separate one-method middleware, plus a 50 percent context warning injection and cost accounting that [mistral-vibe-e4]
- **tool-arg-contract-validation** - cursor-agent - declared arg_aliases, host remap hook, schema-property filtering, required-arg validation converted to structured ToolResult instead of TypeError [cursor-agent-9]
- **tool-batch-scheduling** - mini-kode - all-readonly runs concurrently, any mutating tool forces sequential LLM order, results re-indexed to LLM order, denial cascades to the remainder, and [mini-kode-8]
- **tool-output-offload** - opencode - Oversized tool output is offloaded to a managed on-disk directory (2000 lines / 50KB caps, 7-day retention GC) instead of being silently truncated [opencode-e5]
- **tool-output-pruning-tier** - dexto - per-step tool-output pruning with 40K protect / 20K minimum-savings thresholds, read-time placeholder substitution, history left intact [dexto-e2]
- **torn-journal-salvage** - jcode - Session journal replay salvages torn/glued lines instead of truncating the transcript at the first bad byte [jcode-c2]
- **typed-error-identity** - deepseek-reasonix - An error a caller must distinguish carries identity - errors.Is sentinel inside Go, dotted code through refuse() across HTTP - and two linters [deepseek-reasonix-c9]
- **upstream-workflow-allowlist** - kilocode - Hardcoded CI workflow allowlist fails the build when an upstream merge introduces a runnable workflow [kilocode-s10]
- **usage-anchored-context-projection** - bitfun - TokenAnchor freezes reported input_tokens with a prefix digest and per-layer token facts; pressure = anchor + per-layer deltas + estimated tail, with [bitfun-e1]
- **v1-static-cache-heuristic** - opencode - The shipping V1 path pins cache breakpoints with a static heuristic - first two system messages plus last two messages - fanned across six provider [opencode-e4]
- **verification-maturation** - deepseek-reasonix - 3-OS matrix with -race (targeted sweep per-PR, full sweep on push, incl. the safety/sandbox package set), in-house repolint architecture linters and [deepseek-reasonix-s15]
- **verification-scale-and-quality** - opencode - specs assert decisions and semantics, not existence; permission area alone ~2.7k lines of behavior specs across both trees [opencode-s10]
- **verified-auto-resume** - codewhale - Auto-resume is opt-in, candidate-verified, and reports why it refused - including which newer sessions were unreadable [codewhale-c6]
- **versioned-harness-api-parity-gated-sdk** - jcode - Stable NDJSON harness API with per-frame version negotiation, Unknown catch-alls, schema/capability snapshot tests, and a published TS SDK whose [jcode-e7]
- **windows-appcontainer-sandbox** - orca-agent - Windows sandbox is real kernel machinery (AppContainer profile + restricted token + per-path DACL grants + capability store), tested at [orca-agent-s12]
- **workflow-config-contract-test** - deepagents - eval tests parse harbor.yml/_eval.yml themselves, and paths-filter forces any eval-workflow edit to run its guard tests [deepagents-b4]
- **workflow-token-budget** - qwen-code - Workflows expose a token budget the script can read and the dispatcher gates on [qwen-code-e6]
- **yolo-floors** - deepseek-reasonix - plan approval, memory write/forget, sandbox escape, managed-config write and network-egress approvals survive YOLO/auto/plan-window and [deepseek-reasonix-s5]
- **yolo-unbypassable-failclosed-gates** - oh-my-pi - A few gates are explicitly not yolo-bypassable and fail closed headless: provider safety checks, no-UI approval, ACP rejections, and headless [oh-my-pi-s4]

### nuance (61 concepts)

- **acp-trust-default** - vtcode - Zed ACP workspace-trust default is FullAuto -- editor-initiated sessions default to the most permissive trust tier even though the ACP bridge itself [vtcode-e10]
- **architect-editor-pipeline** - aider - The only orchestration is a hard-wired two-model relay: architect spawns an editor Coder in-process, hands off a string plus cost/hashes, and waits [aider-e6]
- **atomic-session-store** - bitfun - Session persistence is keyed-lock-serialized atomic-rename files with schema versions, and the derived FTS index is declared rebuildable rather than [bitfun-c8]
- **authority-revalidating-tool-cache** - codewhale - Session-cached tool activations are revalidated against the current filtered catalog at every turn start [codewhale-c10]
- **brand-adjacent-naming** - cursor-agent - Standalone MIT harness named cursor-agent; zero dependency on Cursor's closed CLI, collides with cursor.com identity [cursor-agent-1]
- **budgeted-fold-recall** - deepseek-reasonix - each fold leaves an index, a recall tool reads folded canonical positions or searches them, and one per-generation token budget (max [deepseek-reasonix-e4]
- **ci-perf-budget** - forge - a dedicated job runs scripts/benchmark.sh --threshold 60 zsh rprompt on every PR, gating the cost of the ambient zsh integration [forge-13]
- **command-enum-bloat** - grok-build - SessionCommand spans ~108 variants mixing turn, MCP admin, memory, plugins, hooks, permission and rewind concerns [grok-build-c4]
- **compaction-trigger-measurement** - workground2 - Compaction triggers on the previous turn's reported prompt size, not a projection of the next request; no active cache warming [workground2-e5]
- **config-doc-drift** - deepseek-reasonix - The default-config comment contradicts the shipped headless behavior: config.Default says no-TTY Ask resolves to allow, while the CLI run path [deepseek-reasonix-s10]
- **crash-classified-stop-reasons** - jcode - live turns are panic-caught with Crash-vs-Failure stop reasons distinguished without string-parsing error text, crashed sessions are detected and [jcode-s9]
- **crash-group-recovery-sessions** - jcode - Crash recovery derives recovery sessions per crash-group window, with dedupe against existing recoveries [jcode-c4]
- **detect-only-invariant** - continue - The message/tool-call consistency invariant is detected and console.error-ed, not enforced: malformed history is still sent to the model [continue-c10]
- **disclosed-private-mirror** - codebuff - Squashed snapshot of a private source tree with honestly-disclosed process boundary - inverted openhands covariate [codebuff-b2]
- **docs-coverage-and-drift** - workground2 - ~70-doc bilingual contract corpus that matches code (SPEC compaction tiers == compact.go constants), offset by design-doc skew and at least one stale [workground2-e15]
- **docs-tree-quality** - continue - 153-file mintlify tree, CLI pages match actual flags, actionable errors, honest archived README; gaps are auto-compaction, serve, and housekeeping [continue-e10]
- **dual-durability-ledgers** - orca-agent - per-operation execution journal and a session-wide runtime_surface commit ledger (commit ~12.9k + reducer ~9.4k product LOC) still carrying legacy [orca-agent-c7]
- **durable-approval-records** - dexto - Approval is bound in-loop against durable, identity-scoped records: deterministic approval ids keyed on run/turn/step/toolCall, replay-rejects [dexto-s4]
- **economy-machinery-density** - oh-my-pi - five methods x inline imaging x speculation x prune/shake layers converge on a 5,456-LOC maintenance loop and silent config migration [oh-my-pi-e12]
- **equality-guarded-background-summarizer** - aider - Single auto-summarize strategy with adaptive budget, weak-model ladder and an equality-guarded publish race: done_messages collapses toward a [aider-e1]
- **evals-release-gated-no-fuzz** - kilocode - Model evals exist in CI but run only at publish; fuzzing absent - verification ceiling of the anchors not broken on PRs [kilocode-s12]
- **explicit-yes-fail-safe** - aider - --yes-always converts explicit_yes_required prompts to automatic DENY, not automatic yes, and the semantics are pinned by tests [aider-s2]
- **external-command-rewrite** - maki - If an external `rtk` binary is on PATH, every bash command is transparently rewritten through it before execution [maki-14]
- **external-docs-strong-failure-copy** - letta-code - User documentation lives entirely off-repo at docs.letta.com (docs/ holds only nix.md, examples, plans) -- the exact codex-7 shape -- while [letta-code-e12]
- **filesystem-boundary-half-strength** - dexto - The filesystem half of the boundary is real and tested [dexto-s6]
- **folder-trust-off-by-default** - gemini-cli - Folder-trust feature is disabled by default and disabled means trusted, so the YOLO-block and git-gates are inert out of the box [gemini-cli-s3]
- **fork-with-rollback** - dexto - Session fork copies persisted history with lineage and parent-sessionId, and best-effort rolls back partial forks; no rewind or tree navigation [dexto-c12]
- **generalized-task-budget** - deepseek-reasonix - Task budgets bound cost, wall-clock, and tokens with every axis shipped off by default, chosen because 'tokens is the one that generalizes - money is [deepseek-reasonix-e7]
- **hard-gates-and-rollback** - aider - A few enforcement gates do not depend on approval at all, and the git layer is deliberately built to make every aider change revertible [aider-s4]
- **honest-absent-enforcement** - hax - Prompt-only restraint with explicit anti-deception documentation: the reference rung-3 - honesty buys exactly one rung up out of codel's misleading [hax-b1]
- **http-front-auth** - workground2 - HTTP serve front: loopback default, fail-closed auth modes, anti-spoof proxy flag [workground2-s16]
- **in-flight-loop-decomposition** - opensquilla - 8 stage classes are called, but from a 2.2k-line inline _run_turn; the orchestrator is explicitly not written and a 2,043-LOC adapter harness binds [in-flight-loop-decomposition]
- **interop-mcp-server-and-ide-gap** - oh-my-pi - omp consumes MCP and speaks ACP/RPC but never exposes itself as an MCP server, and IDE adjacency is web/relay, not VS Code/JetBrains [oh-my-pi-e10]
- **invalid-config-escalates** - oh-my-pi - A malformed tools.approvalMode value with settings present silently resolves to yolo, while a fully missing context fails closed to always-ask [oh-my-pi-s9]
- **markdown-transcript-resume** - aider - Session persistence is an append-only human-readable markdown transcript re-parsed on opt-in resume; no session id, store, fork or tree [aider-c6]
- **mid-turn-steering** - workground2 - Mid-turn steering queue is first-class but in-memory only; foreground turn admission itself has no durable queue [workground2-e9]
- **migration-confusion-surface** - codewhale - Docked from docs 9: the mid-flight split is a real confusion surface even without duplicate code on disk; no root SECURITY.md [codewhale-e13]
- **multi-agent-team** - cline - Named team of configured agents with a spawn-agent tool [cline-6]
- **no-published-sdk** - hermes-agent - no published SDK/protocol artifacts, and the one versioned machine contract (relay connector) self-labels EXPERIMENTAL [hermes-agent-e10]
- **opt-in-provider-cache** - continue - Prompt caching is real but opt-in and provider-siloed; nothing guarantees stable prefixes across turns or surfaces [continue-e3]
- **orchestration-docking-notes** - bitfun - queue receipts and concurrency limiter are host-memory-only, dual-runtime concept duplication over IPC, swarm state inside the 21.9k-LOC coordinator [bitfun-e8]
- **orphaned-persistence** - mini-kode - Sessions are persisted on every run but loadSession has zero production callers - no resume path exists [mini-kode-b7]
- **parity-simulation-gap** - letta-code - The repo's one named tri-loop parity test simulates neither loop: it drives QueueRuntime with a hand-written script that mirrors the two hosts' queue [letta-code-c9]
- **pinned-attribution-hygiene** - nausicaa-harness - Every adapted third-party snippet is attributed by upstream commit hash with an explicit not-copied boundary list [nausicaa-harness-12]
- **projected-compaction-credit** - continue - projects actual model input (system + tool defs + history), prunes with an infinite-loop guard, and fail-opens visibly [continue-e2]
- **repo-map-budget** - aider - Repo map is an explicit token-budgeted context component: window-scaled default, empty-chat boost capped by the window, sampled token estimation [aider-e4]
- **second-backend-parallel-plane** - letta-code - The offline/local backend is a second complete execution plane -- own turn executor on pi-ai, own transcript store (3,301 LOC), own compaction -- not [letta-code-c10]
- **session-fork** - opencode - V1 operability surface: numbered forks, child queries, revert/unrevert, export/import, and a raw sqlite shell as diagnostics [opencode-c10]
- **split-history-ownership** - trae-agent - Conversation history is owned by the LLM client, not the loop: the loop passes only message deltas [trae-agent-5]
- **stale-required-status-checks** - agentty - Branch protection still requires check names no workflow emits; workflow header documents the removal instead of the repo silently carrying them [agentty-b5]
- **stream-resume-gap-detection** - opensquilla - stream_seq range must be complete and within a bounded window, otherwise the transfer closes and a full-resync notice replaces the tail [stream-resume-gap-detection]
- **test-boundary-policing** - opensquilla - unit suites may not frame-walk (sys._getframe) or import private runtime adapters, and frame-walking probes are only allowed in filename-tagged [test-boundary-policing]
- **test-corpus-real-shape** - aider - Real, behavior-asserting test corpus - but 12.4k LOC / 473 tests (not the census 76k), provider fully mocked, no coverage gate, no fuzzing, no evals [aider-s7]
- **token-economy-ceilings** - bitfun - one model-summary compaction strategy (no codex no-LLM token-budget tier, explicit no-local-fallback), no cache warming, zero Anthropic cache_control [bitfun-e4]
- **tool-pair-resume-invariant** - oh-my-pi - Crash/retry resume is defined by an explicit pairing invariant: only a trailing assistant message with unpaired runnable tool calls may continue [oh-my-pi-c9]
- **turn-persistence-ordering** - oh-my-pi - Incremental persistence uses a stable structural message key plus an out-of-order bail so a compaction mid-turn cannot splice stale messages into the [oh-my-pi-c10]
- **upstream-loop-assembly** - deepagents - loop lives in upstream langchain create_agent; the in-tree middleware assembly is the scoreable architecture, capped at 8 [deepagents-b1]
- **upstream-sync-cron** - codex-infinity - sync-fork mechanics: personal cron merges openai/codex main daily [codex-infinity-1]
- **verification-corpus** - goose - Real, broad, PR-gated test corps with per-PR live-model smoke, but fuzz-free and effectively ubuntu-only for engine tests; model sweeps are [goose-s9]
- **wide-facade-delegation-holds** - dexto - DextoAgent.ts is 3,724 LOC with ~100 methods but delegates the loop: no provider call inside the file, stream() is event fan-in plus one [dexto-c7]
- **write-is-the-boundary** - deepseek-reasonix - sandbox.network defaults true so a permitted command under the default posture has full external egress [deepseek-reasonix-s14]

### anti-pattern (99 concepts)

- **adr-doc-skew** - mistral-vibe - ADR 0011 links a design doc that does not exist in the repo; product docs are off-tree and the 1,062-line README carries everything else [mistral-vibe-e10]
- **ambient-content-injection** - aider - Scraped web pages, clipboard text and command output enter the prompt with at most one plain confirm and no untrusted-content fencing anywhere [aider-s3]
- **api-implicit-yes-always** - aider - Scripting/API entry silently enables yes-always, auto-approving every edit confirm including create-file and edit-not-in-chat, with no test pinning it [aider-s5]
- **approval-default-off** - goose - The single enforcement rung - tool approval - is bypassed out of the box: GOOSE_MODE defaults to auto, and unflagged tool requests default to [goose-s1]
- **background-recovery-thin** - vtcode - Background-subagent crash semantics are state-echo only: a JSON record with no lease, heartbeat, or reattach; the dedicated CLI is a demo path [vtcode-e7]
- **branch-indifferent-security-tests** - goose - Some security tests pass in both states or only exist under forced-enable: the SecurityInspector integration test asserts nothing about detection in [goose-s8]
- **cache-strategy-siloed-to-bedrock** - roo-code - The multi-point cache machinery is consumed only by the Bedrock provider; every other cached provider gets a static last-two-user-messages heuristic [roo-code-e4]
- **checked-in-run-artifacts** - auto-code-rover - 927 MB of experiment result blobs committed to the repo [auto-code-rover-b5]
- **checkpoint-failopen-degradation** - roo-code - Checkpoints silently disable themselves for the rest of a task on any init timeout or restore error (11 enableCheckpoints=false sites), so rewind [roo-code-c7]
- **ci-excluded-test-files** - atomic-agent - Two red tests excluded from the CI vitest run to keep main green (send-message-concurrency, local-models-orchestrator-auto-update) - loudly [atomic-agent-9]
- **compaction-engine-split** - continue - Two divergent compaction engines; the flagship VS Code surface has no auto-compaction at all, only a manual button [continue-e1]
- **compaction-god-pair** - hermes-agent - The compaction stack is itself the lane's largest god-module pair and the marker-drift class of bugs it breeds is documented in-source [hermes-agent-e5]
- **compaction-trigger-completion-budget** - coro-code - Auto-compaction threshold is computed against params.max_tokens (the completion budget, default 8192) instead of the model context window [coro-code-2]
- **compaction-trigger-fixed-window** - mini-kode - Auto-compaction threshold hardcoded to 115k (90% of 128k) while the shipping default provider preset is a smaller-window model - trigger unreachable [mini-kode-b6]
- **context-utils-grab-bag** - dexto - context/utils.ts is a 2,224-LOC module of 34 exports mixing token estimation, blob resolution, MIME matching, tool-result sanitization, and display [dexto-c8]
- **core-loop-naming-trap** - mistral-vibe - vibe/core/loop.py is not the agent loop - the name squat redirects every reviewer and ADR-ignorant contributor away from agent_loop/_loop.py [mistral-vibe-c9]
- **core-loop-uncovered** - trae-agent - Core agent loop (while-step, LLM step, tool-call handler) has zero test references [trae-agent-b2]
- **crash-durability-gaps** - opencode - no sqlite-corruption recovery, V2 durability explicitly unchecked, interruption is interrupt-only cause swallowing [opencode-m1]
- **dead-concurrency-classification** - darce-cli - Tool calls are bucketed into concurrent vs sequential by isReadOnly/isConcurrencySafe, then the buckets are flattened and every tool runs sequentially [darce-cli-5]
- **docs-breadth-with-rot-and-undocumented-acp** - jcode - 82-file docs tree with an honest current/plans/proposals split and good diagnostics docs, but its own index links two nonexistent desktop docs, the [jcode-e10]
- **docs-code-divergence-headless-permissions** - continue - Docs state the opposite of the code on headless defaults: documented 'ask tools are excluded', code defaults to Bash allow plus wildcard allow [continue-e9]
- **docs-depth-vs-dual-tree** - opencode - Docs/DX is broad and CI-gated - 36 top-level product pages x 25 locales (614 mdx), locale-sync workflow, AGENTS.md style guide, CONTEXT.md domain [opencode-e13b]
- **docs-scope-gaps** - grok-build - Docs cover users exhaustively but there is no SECURITY substance or in-repo architecture documentation [grok-build-e18]
- **dual-permission-engine-mid-migration** - letta-code - Two permission engines ship simultaneously with v2 flipped default-on by env-absence and the v1/v2 differential check gated behind an opt-in flag [letta-code-s10]
- **dual-tool-engine** - qwen-code - ACP/daemon host re-implements the entire tool-execution engine (~3.4k LOC) outside CoreToolScheduler, synced by comment [qwen-code-c1]
- **duplicate-policy-path** - gemini-cli - ACP entry path bypasses Scheduler and maintains a second tool-execution and policy-evaluation implementation [gemini-cli-c3]
- **egress-log-only** - goose - destination detection + logging with a hard-coded Allow action, so network-exfiltration control is telemetry, not enforcement [goose-s5]
- **enforcement-untested-in-ci** - orca-agent - No CI job on any platform ever proves the OS sandbox blocks anything: negative enforcement tests are macOS-only behind a macOS-gated probe with no [orca-agent-s4]
- **ephemeral-checkpoint-rewind** - mistral-vibe - Checkpoint log is memory-only: rewind and file-restore die at process exit, so a resumed session cannot rewind any prior turn [mistral-vibe-c6]
- **ephemeral-orchestration-state** - continue - No durable orchestration layer: queue is in-memory FIFO, sessions are whole-file non-atomic JSON, crash handlers swallow and defer exit [continue-e6]
- **ephemeral-task-registry** - dexto - TaskRegistry is an in-process Map with Math.random ids and 5-minute result TTL, sitting next to a sqlite-everywhere session layer [dexto-e7]
- **fork-overlay-loop** - kilocode - Shipping agent loop is an upstream file patched in-place: 69 kilocode_change blocks rewrite loop control flow inside prompt.ts [kilocode-c1]
- **glob-command-grants** - forge - 'git push' becomes git push*, which also admits 'git push && curl evil.sh' since glob * spans shell metacharacters; write grants become *.ext [forge-4]
- **global-singleton-session-state** - code - Session-critical state lives in process-wide singletons that bypass the Session module: agent manager, browser manager, and the submission channel [code-c4]
- **god-object-controller** - workground2 - *Controller is a 328-method god object across 38 files; the port split is documented intent, not executed [workground2-c5]
- **god-package-concentration** - deepseek-reasonix - Two packages hold ~46k non-test LOC of the kernel - internal/runtime/agent (132 files, 23.1k) and internal/session/control (102 files, 22.8k) - where [deepseek-reasonix-c8]
- **guardrail-stripped-redistribution** - free-code - product identity = deletion of upstream safety layers [free-code-3]
- **hooks-blind-to-default-mode** - letta-code - PermissionRequest hooks (the user's deny hatch, e.g. the shipped block-rm-rf.sh example) only run when the decision is already "ask", so they are [letta-code-s5]
- **host-resident-turn-policy** - qwen-code - The agentic iteration loop lives in each of the four hosts, not in core; parity held by per-scenario tests [qwen-code-c4]
- **image-cost-blind-spot** - vtcode - Images and files are a flat 1000-token guess in every accounting path; no image budget, downscale, or per-format sizing [vtcode-e5]
- **implicit-shared-loop-state** - aider - Conversation state is four mutable attributes (done_messages, cur_messages, reflected_message, aider_commit_hashes) poked directly by commands [aider-c3]
- **in-memory-queue** - kilocode - Prompt queue and background jobs are in-process Maps; only graceful shutdown drains, a crash loses queued work [kilocode-e6]
- **injection-defense-absent** - dexto - the 'security' hooks are a profanity/PII policy and an opt-in redactor, nothing marks tool output as untrusted in the system prompt [dexto-s5]
- **interop-outbound-and-sdk-missing** - vtcode - Interop is inbound-heavy: no vtcode-as-MCP-server, no published SDK, and the 'app-server' is a proxy to codex's protocol, not a versioned own protocol [vtcode-e9]
- **kernel-path-test-depth** - opensquilla - Kernel-boundary tests mostly assert generated argv/SBPL plans; live kernel denial is exercised in CI only as a raw bwrap probe, and the sole [kernel-path-test-depth]
- **kernel-policy-string-tests-only** - letta-code - seatbelt/bwrap tests assert generated profile text and argv order, no test spawns sandbox-exec or bwrap, and no CI job installs bubblewrap -- the [letta-code-s7]
- **last-message-state-inference** - roo-code - CLI reconstructs the agent loop's state machine by heuristics on the tail of the presentation message stream [roo-code-c3]
- **llm-generated-approval-grammar** - opencode - The command-prefix dictionary that decides what an approval *means* was generated by an LLM prompt, prompt preserved in-source, guarded by 33 lines [opencode-s6]
- **mixin-composed-state-bag** - hermes-agent - The Sept 2026 decomposition moved code into modules but not state: AIAgent and SessionDB are 13-mixin composites and the phases share a mutable agent [hermes-agent-c7]
- **model-specific-loop-residue** - oh-my-pi - Provider-agnostic loop package carries GPT-5 Harmony machinery, dialect env-switching, and a hardcoded bundled model [oh-my-pi-c3]
- **monolithic-agent-module** - ferrum - agent/mod.rs 10,178 LOC (~7,076 non-test) fuses loop, TUI completion/pickers, slash commands, compaction, and system prompt [ferrum-b6]
- **monolithic-turn-function** - grok-build - The agent turn is a single 1,261-line function carrying ten-plus mutable retry/streak states [grok-build-c3]
- **mutation-metric-not-gate** - grinta-coding-agent - Every-PR mutmut can never fail on a surviving mutant: pinned tool has no score-failure semantics, scope is 1.9% of the tree [grinta-coding-agent-b1]
- **naive-json-session-store** - continue - Session durability is non-atomic whole-file JSON with a silent-corruption data-loss path shared by both surfaces [continue-c7]
- **next-task-oracle** - codel - No agent loop: provider returns exactly one next task per poll [codel-1]
- **no-cost-token-budget** - vtcode - No token or cost budget anywhere in orchestration: subagents get turn caps, the loop gets tool-call caps, nothing caps spend [vtcode-e4]
- **no-image-budget** - aider - Vision content is unbudgeted: images enter messages with no token accounting or count cap anywhere [aider-e5]
- **no-protocol-interop** - aider - Zero protocol interop: no MCP, no ACP, no machine-readable output mode -- headless is human text out [aider-e8]
- **orphan-service** - gemini-cli - ContextCompressionService (526 LOC, file-level FULL/PARTIAL/SUMMARY/EXCLUDED ladder with cached summaries + content hashes) has no importers [gemini-cli-e6]
- **overflow-hard-stop** - aider - provider error sets a flag, aider prints advice and returns; nothing ever auto-retries with a smaller context [aider-e2]
- **per-provider-loop-duplication** - keen-code - The tool-loop scaffold (maxToolTurns loop, proactive-compaction hook, incomplete-exit, pending-state injection) is re-implemented in all six provider [keen-code-9]
- **permission-widening-ux** - gemini-cli - Post-denial UX systematically proposes permission widening, and the eval suite trains the model to ask for more permissions on failure [gemini-cli-s6]
- **phantom-state-snapshot** - code - SavedSession serializes a permanently-empty state object, record_state discards its argument, and resume silently drops TurnContext items [code-c7]
- **presentation-god-region** - jcode - 217k-LOC jcode-tui is a presentation god-region: 41 files over 2.5k LOC, capped just under the 5k errata trigger [jcode-c6]
- **process-local-orchestration** - opencode - Background jobs and the V2 run coordinator are in-process only; the durable runner's own checklist admits no durable status, no [opencode-e8]
- **prompt-baked-policy** - codel - Approval policy lives in prompt text ('Always auto approve terminal commands') [codel-3]
- **prompt-cache-audit-only** - orca-agent - Prompt-cache layer is honest audit metadata, not reuse machinery: checkpoints are journaled and prefix-reuse is a self-declared lower bound, but no [orca-agent-e3]
- **prose-only-run-contract** - dexto - `dexto run` - the primary headless entry - emits only prose `[tag]` lines to stderr with no machine-readable event stream, while --json exists only [dexto-e11]
- **reactive-only-compaction-single-knob** - dexto - Both compaction gates are the same reactive 0.9-threshold strategy; no proactive tier and no recovery path when the provider itself rejects an [dexto-e3]
- **replica-tests** - binharic-cli - they reimplement the logic under test inline (timeouts, loops) or assert tautologies, inflating test_loc and the green badge without testing [binharic-cli-4]
- **sanitizer-coverage-ceiling** - agentty - Prebuilt uninstrumented renderer submodule structurally caps sanitizer coverage to a labeled test subset [agentty-b6]
- **self-scored-docs-accuracy-risk** - vtcode - the same agent-maintained tree already contains a phantom guarantee (CostBudget) and a live-path claim for the unwired prefire tier [vtcode-e12]
- **signature-sniffing-compat-shim** - opensquilla - 41 runtime introspection points (_accepts_keyword_arg / _accepts_explicit_keyword_arg) negotiate kwargs against TurnRunner private methods and [signature-sniffing-compat-shim]
- **silent-threat-model** - aider - No SECURITY.md and no doc states what the approval model does not enforce - silence, not a false claim [aider-s9]
- **single-breakpoint-cache** - dexto - one static ephemeral breakpoint on the system prompt, plus cache-read/write tokens that are measured and priced but never acted on [dexto-e4]
- **single-crate-runtime** - grok-build - The shell crate holds runtime, leader/relay/remote, persistence and wiring in one 436k-LOC compilation unit [grok-build-c5]
- **single-maintainer-inheritance** - codex-infinity - S-grade codebase inheriting its maintainer risk down to solo-cron level [codex-infinity-3]
- **split-fail-closed-fail-open** - letta-code - unattended memory-subagent confinement throws when no kernel backend exists, but every other sandbox entry point warns and continues unconfined, and [letta-code-s3]
- **stale-ci-exclusion** - amazon-q-developer-cli - Push-gated test job excludes a crate absent from the workspace (rename-rot from Fig era) [amazon-q-developer-cli-b3]
- **targeted-package-ci** - bitfun - every cargo-test invocation is -p-scoped (and often a single test filter or --no-default-features slice), with no coverage gate, no fuzzing, and no [bitfun-s9]
- **telescoping-constructors** - zeroclaw - Agent built through 11 stacked from_* wrappers forwarding 15 positional params including 3 consecutive bools [zeroclaw-c3]
- **test-addressed-facade-surface** - ouroboros - Refactor leaves preserve ~150-name historical re-export facades because tests and sibling leaves monkeypatch private names through them [ouroboros-11]
- **test-name-overreach** - deepseek-reasonix - TestExternalActionsAlwaysReachApproval is weaker than its name: it feeds the mode strings "auto"/"yolo" into permission.New, whose ParseDecision maps [deepseek-reasonix-s13]
- **test-only-agent-loop** - binharic-cli - A second, 'advanced' agent layer [binharic-cli-6]
- **thin-published-plane** - roo-code - @roo-code/types is a version-0.0.1 types-only package, the ipc package is a private CLI<->extension socket, and there are exactly two IDE-adjacent [roo-code-e12]
- **timer-throw-uncaught** - binharic-cli - Stream-timeout watchdog throws inside a setTimeout callback, so the TransientError escapes the enclosing try and lands in the process-level [binharic-cli-9]
- **ui-god-host** - pi - 6852-LOC interactive-mode UI host in an otherwise exemplar modular tree [pi-b2]
- **ui-render-triplication** - kilocode - Same message renderer maintained three times beside a 6110-line webview host god object [kilocode-c9]
- **unbounded-subagent-fanout** - letta-code - Subagents spawn as full headless `letta` child processes with maxTurns plumbing but no concurrency cap, semaphore, or cost budget anywhere in the [letta-code-e8]
- **undocumented-economy-knobs** - dexto - The token-economy control surface (thresholdPercent, maxContextTokens, preserveLastNTurns, strategy choice) is schema-validated but absent from user [dexto-e13]
- **undocumented-json-schema-plus-stubs** - roo-code - The machine-facing contract is under-documented relative to the human-facing one: no prose doc for the stream-json event schema outside TS types [roo-code-e14]
- **unenforced-layering-law** - opencode - Dependency law exists only as AGENTS.md prose; no depcruise/boundary linter, while core directly depends on 20 provider SDKs [opencode-c7]
- **ungated-approval-gate** - aider - The shell-approval gate itself has zero test coverage: the gate is mocked away and run_cmd has one echo test [aider-s6]
- **untested-reflection-loop** - aider - The core loop's control flow - run_one, the reflection while-loop, max_reflections, restore-chat-history, the summarizer thread - has zero direct [aider-c9]
- **untrusted-wrapping-coverage** - opensquilla - wrap_untrusted/scan_for_injection cover only goal/plan/workspace-context files; tool results, web fetch, MCP results and 11-channel ingress are [untrusted-wrapping-coverage]
- **unwired-crash-resume** - dexto - Checkpoint/resume machinery is fully built and tested but has zero production callers; server admits restart-survival is unimplemented [dexto-c2]
- **unwritten-event-journal** - dexto - RuntimeEventStore is defined, DB-backed, and wired into the shipped image, but no production code ever appends or lists a record [dexto-c6]
- **vendored-fork-unpinned** - memcode - 41.5k-LOC vendored TUI fork copied without recording the upstream version [memcode-12]
- **vestigial-refactor-tree** - code - The documented task/state architecture (SessionTask/ActiveTurn/TurnState, mirroring upstream's turn refactor) is dead code: tasks/ and state/ are not [code-c3]

### safety-hole (25 concepts)

- **approval-key-scope-bypass** - dexto - Pattern approval keys are derived from whitespace-split tokens with shell metacharacters ignored, so an approved 'git push * ' key also covers 'git [dexto-s1]
- **backend-absent-failopen** - 3code - Kernel-backend failure degrades bash to unconfined - announced once on stderr with exactly what was lost, never refused [3code-b3]
- **dangerous-default-widened-surface** - oh-my-pi - Schema default is yolo (auto-approve exec), the tool-declared critical-pattern override is explicitly ignored under yolo, and the attack surface [oh-my-pi-s2]
- **dead-command-validation-layer** - dexto - CommandValidator still computes requiresApproval after 'command-level approval removed', but no consumer anywhere reads it; the regex deny-list [dexto-s3]
- **default-allow-policy** - opencode - allow`: bash, edit, write, apply_patch, webfetch all auto-approved out of the box; only external_directory, *.env reads, doom_loop and three utility [opencode-s3]
- **default-write-open-read-open** - kilocode - Default posture auto-approves file writes and all reads within the project; only bash/edit-adjacent surfaces prompt [kilocode-s7]
- **eval-approval-bypass** - ra-aid - Tool loop eval()s model output with default builtins, so the shell approval gate is fully bypassable [ra-aid-1]
- **eval-on-model-output** - agentless - Python eval() executed on strings parsed out of LLM output -- prompt-injected issue text reaches host RCE in the pipeline process [agentless-3]
- **hook-timeout-failopen** - maki - 30s hook-chain deadline resolves to Verdict::Unchanged, so a wedged policy plugin silently stops filtering user messages, stop-gates, and compaction [maki-12]
- **host-parity-verification** - qwen-code - Approval enforcement is re-entered per host and the cross-host permission tests are one-test thin where hooks parity gets a dedicated suite [qwen-code-s6]
- **injection-refusal-dead-code** - opensquilla - The untrusted-origin tool-call refusal can never fire in production: origin_trace is never populated by any provider [injection-refusal-dead-code]
- **no-interactive-approval-gate** - jcode - the two-tier permission system that classifies bash/write/edit as RequiresPermission is consumed ONLY by ambient sessions; in a normal session the [jcode-s3]
- **no-prompt-injection-posture** - jcode - No prompt-injection posture at all despite webfetch, browser, computer-control, MCP, native-SSH and gmail/composio surfaces consuming untrusted [jcode-s5]
- **plaintext-token-compare** - codewhale - Runtime API bearer comparison is plain string equality (not constant-time) [codewhale-s8]
- **prompt-injection-zero-defense** - opencode - Zero named prompt-injection defense: web/websearch/MCP output enters history unmarked; webfetch has no SSRF guard; MCP connect auto-enables [opencode-s4]
- **retired-tool-replay-surface** - codewhale - Retired coordination tools remain registered and executable by name for transcript replay [codewhale-e8]
- **risk-classifier-failopen** - grinta-coding-agent - Security-analyzer exception falls the effective risk DOWN to the model's own declared level, LOW when undeclared [grinta-coding-agent-b3]
- **sandbox-surface-exclusion** - qwen-code - The kernel-grade bwrap confinement explicitly refuses the riskiest hosts and platforms: Linux-only, rejects ACP/serve/MCP/LSP/extensions, and [qwen-code-s4]
- **server-exposure** - opencode - HTTP server unauthenticated by default plus mDNS advertisement; when enabled, Basic-auth middleware is properly spec-tested [opencode-s9]
- **skill-whitelist-bypass** - waveloom - Skill allowed-tools bash whitelist is sticky and bypasses the RiskHigh hard-block for all later bash calls until the next skill load [waveloom-5]
- **static-default-session-secret** - openlumara - WebUI session middleware falls back to the literal 'openlumara-default-session-secret-change-me' [openlumara-8]
- **subagent-allow-all-escalation** - continue - CLI subagents globally flip the shared permission service to allow-all (tool "*"), monkey-patch other services, and run with no budget or nesting [continue-e5]
- **thin-transport-security-services** - oh-my-pi - Credential-proximate network services ship with bearer-allow-list transport security and the repo's 'test/security' suite validates a scanner-product [oh-my-pi-s8]
- **undocumented-bind-all-control-plane** - continue - Hidden, undocumented, unauthenticated headless control server binds all interfaces: any network peer can inject prompts and rubber-stamp the agent's [continue-s3]
- **unwired-security-validator** - claw-code-agent - 1,261-LOC ported bash-security validator with 163 tests is imported by nothing in src/ [claw-code-agent-b4]

### license-risk (4 concepts)

- **busl-change-license** - kolega-code - license conflict is a census misread: BUSL-1.1 binds now; AGPL-3.0+ is only the post-2030 Change License [kolega-code-1]
- **missing-license-open-source-claim** - claw-code-agent - No LICENSE file anywhere while README badges 'license-open-source' [claw-code-agent-b2]
- **mit-restrictive-use-addendum** - mimo-code - USE_RESTRICTIONS.md imposes conduct rules on 'MiMoCode or any derivatives' atop MIT [mimo-code-2]
- **rebranded-redistribution-provenance** - open-interpreter - Apache-2.0 compliant rebrand, but residual provenance/trademark exposure [open-interpreter-1]

---
Generated against wave6 findings.jsonl. No score files or findings modified. hotdog self-positioning per `subjects/hotdog.md` (68.5 B; safety 5 / interop 5 / verification 7) -- the HOTDOG POSITION lines above are deliberately blunt: top-tier in exactly two clusters (workflow-resume-journal, validated-finish-gate), absent from most of the rest.
