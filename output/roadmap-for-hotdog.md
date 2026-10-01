# Roadmap for hotdog -- impact/effort ranked from the field study

Built only from `findings.jsonl` (1,666 records, 104 subjects, recomputed 2026-10-01) plus our own
scored row (`scores/hotdog.json`, `subjects/hotdog.md`: 15 findings, hotdog-1..15). Candidate pool =
`kind:portable` or `kind:safety-hole` (754 records). Raw arithmetic: `wave6/roadmap-ranking.md`; this
is the ranked conclusion -- 16 items, 8 non-moves.

## Where we are

hotdog is 68.5 / band B, closest anchor pi, strongest dimension originality (8: marker mangler
`hotdog-1`, lane ledger `hotdog-3`, workflow journal `hotdog-4`), weakest durability (4). The three
gaps holding us out of the A-band are the three the corpus argues about loudest:

- **safety-enforcement 5** -- the S-gate blocker. `user-gate.enabled` defaults false, so stock
  hotdog mediates no tool call; the always-on `Workspace` containment covers file tools only and bash
  bypasses it by construction (`hotdog-8`, filed against ourselves). The amended doctrine
  (calibration-notes:1007) keys the rung to the binding *default* posture, plus at most one rung for
  an opt-in layer passing three mechanical gates (fail-closed; enforcement tests in a blocking CI job;
  non-deceptive), and caps zero-binding-default shapes at 5 -- the octomind shape. Our machinery
  already passes gates (i) and (ii); it just ships off.
- **verification 7** -- 184 test files / 72.7k LOC that assert behavior, plus an independent-oracle
  conformance tier (`hotdog-15`), docked for one ubuntu CI job, coverage run-not-gated, and
  everything in-process: nothing drives the shipped binary.
- **interop 5** -- MCP client only, own undocumented websocket protocol, no ACP, no SDK, no IDE.
  This is the gap we intend to leave mostly open (see non-moves) and close only on one axis.

Token-economy 7, orchestration 8, operability 7, architecture 8, docs-dx 8 -- the two 8s that are
already A-band-shaped get protected by the non-move list rather than improved.

## Ranking formula

`score = convergence_subjects x thesis_fit_sum`, where convergence_subjects is the number of
distinct subjects carrying the concept (all kinds, identical to the convergence.md headers) and
each thesis contributes 0/1:

- **T1 zero-dep-compatible** -- buildable with what is in-tree; a new parser or service dependency
  scores 0, so the tree-sitter bash exemplars (ferrum-1, kimi-code-2, memcode-1) FAIL T1 and port as
  concepts onto hotdog's hand tokenizer (`hotdog-7`) only.
- **T2 solo-maintainer-sustainable** -- no protocol churn, platform matrix, or support contract a
  single maintainer (944/954 commits) carries forever. **T3 user-paid-tokens benefit** -- the payoff
  lands on the user's tokens, not our CI bill. **T4 machine-checked-orchestration fit** -- gates that
  fail mechanically (verdict files, blocking CI jobs), not advisories that warn.

Ties break on evidence quality, then lower effort; effort is `effort_for_us`, "(est)" where null.
AGENTS.md binds every item -- no new dependencies, no speculative flags, Bun only -- which is why
several large-concept candidates never become items. `hotdog-8` is `kind:anti-pattern` and so is
*excluded* from the pool by the rule: per calibration-notes:1001 it anchors the moves-vs-non-moves
analysis, and the item that closes it (rank 3) is sourced from permission-policy portables.

## Close the safety floor (ranks 3, 8, 11, 12, 13)

The floor is not a jail. It is a decision that binds by default, uncertainty that lands on
deny-or-human, and no advertised control that fires nowhere. Every item here is scoped so it can
land without a dependency and without a new flag: changing a default is not adding a knob.

**3. permission-policy -- default-on approvals** | 61 subjects (85 findings, 45 of them candidates)
| T1100 = 2 | score 122 | effort M (the flip itself is S) | evidence: crab-code-3, dvalincode-1,
atomic-agent-4 | our gap: `hotdog-8`
Build: flip `enabled` to true in `src/extensions/user-gate/extension.json`, with the existing
default-deny path (which already prints the exact config line that would have allowed the call) as
the documented opt-down. Landing module: **user-gate** + `src/utils/approvals`.
Fit sum 2 and still rank 3: no token benefit (T3=0), mediation rather than orchestration (T4=0),
but 61 subjects is the heaviest concept we are on the wrong side of. The corpus sides with
default-bind (kolkrabbi-4, minicode-1, kilocode-b6 ships bash ask) and the doctrine caps
zero-binding-default at 5 (octomind, nanocoder). Our fail-closed matrix
(`tests/extensions/user-gate.test.ts:245-388`) already passes the +1 opt-in gates, so the flip moves
the default rung instead of adding machinery; cost is first-run UX plus docs, no new code path.

**8. hook-trust-scoping** | 25 subjects (26 findings, 20 candidates) | T1101 = 3 | score 75 |
effort S | evidence: bitfun-s4, claurst-5, pi-4 (memcode-6 is the safety-hole pole) | our gap:
convergence records hotdog's position as absent; calibration-notes:544 upgraded this from "check"
to a must-have trust gate plus test for anything workspace-sourced
Build: a content-hashed trust store outside the repo, keyed on the resolved bytes of every
workspace-sourced extension path, hook definition, and project config; untrusted means not loaded
(refuse, don't sandbox), and a change to any byte revokes. Landing module: **extension loader**
(`src/extensions/extensions.ts` path-spec seam) + hooks.
Rules from the evidence: a repo can never grant itself trust (claurst-5); the gate covers settings,
packages and extensions before the first prompt (pi-4); an approval waives only the interactive
prompt and is re-checked after Deny (bitfun-s4). Clone-and-run is a 7-safety-hole cluster
(memcode-6, waveloom-4, claurst-b3, cline-b2, mini-kode-b3, kode-cli-6) and hotdog loads extensions
by path spec ungated, so a poisoned `.hotdog/` in a cloned repo is live today.
**11. grammar-parsed-bash-policy -- concept-only, fuzz it** | 16 subjects (18 findings) |
T0111 = 3 (**T1=0** as implemented by the exemplars) | score 48 | effort L | evidence: ferrum-1,
smelt-2, memcode-1 (kimi-code-2, vtcode-s7 are the tree-sitter pole) | our gap: `hotdog-7`
Build: keep the hand tokenizer -- importing tree-sitter-bash would break the zero-dep rule, so
these exemplars port as a *policy shape*, not a parser. Extend `src/utils/approvals/bash.ts` with
the fail-closed edge set the cluster agrees on (wrapper/`env` fixed-point normalization, line
continuations, heredoc bodies, env-assignment prefixes all produce a reason rather than silence),
then add seeded fuzz targets that run in the blocking CI job. Landing module: **approvals/bash.ts**.
The cluster concludes "parse-and-fail-closed beats match-and-hope" (ferrum-1 denies on
parser-unavailable, parse-fail, syntax-error, size and depth limits; waveloom-7 routes every
AST-vs-tokenizer disagreement to ASK). We are right on polarity -- unsure means ask, and with rank 3
landed a bail denies -- and convergence.md ranks us last of the 16 on evidence tier, not design. The
gap is a fuzz corpus, not a grammar.

**12. security-posture-docs** | 24 subjects (27 findings) | T1100 = 2 | score 48 | effort S |
evidence: ferrum-9, openhands-8, minicode-10 (deepagents-9 automates it) | our gap: docs-dx is 8
but the safety lane has no per-mode table naming what does not bind
Build: one page, mode by mode (stock, gate-on, gate-on+bash-triage), grading each control and
naming the non-boundaries in the same voice that already says "nothing below it enforces any
boundaries". Landing module: **docs/** (SECURITY.md + a generated section).
Tie-break against rank 11 at the same score: all 13 candidate findings here are effort S, and our
honesty posture is already an asset the doctrine pays for (honest absence earns one rung out of the
misleading band, never a substitute for enforcement). After rank 3 this page has to be rewritten
anyway -- writing it as a graded table with named omissions is what makes the rung auditable instead
of aspirational.

**13. phantom-safety-control -- docs-match-code contract test** | 15 subjects, 17 safety-holes |
T1101 = 3 | score 45 | effort S | evidence: binharic-cli-2, claurst-b1, dexto-s2, opensquilla-b2
(code-m1 is the flagship: a documented Windows sandbox whose harness does not exist) | our gap:
none today -- this item buys the invariant that keeps us there
Build: a CI test that every config knob whose schema description claims enforcement resolves to a
call site, and that every documented default equals the shipped default. Landing module: **CI test
tier** next to the existing config/rescue diagnostics.
The failure census: a PermissionsManager with zero call sites (binharic-cli-2), a `/sandbox-toggle`
read by nothing but its own status display (claurst-3/b1), a TUI rendering approve prompts no code
emits (tura-5), a fail-open to bare `sh` announced as "filesystem isolation is still active"
(claw-code-2). The doctrine never rewards this shape with the +1, and the claurst boundary ruled
phantom protection worse than plain absence -- our 5 rather than 2 rests on shipping the gate visibly
off, and rank 13 makes that structural rather than a habit.

## Raise verification toward 8/9 (ranks 1, 4, 14)

Doctrine first. The corpus rule is mechanical and audit-confirmed: a gate counts only if the job
**blocks** on failure -- continue-on-error evals are existence of a harness, not enforcement (gptme
ruling, calibration-notes:862-868). The published definition (calibration-notes:1047) is verification-9
= "a live-model (or CI-executed oracle'd-fuzz) gate on a path that cannot stay green when behavior
regresses", tolerant steps plus a hard verdict count; continue-on-error, exit-0-on-regression and
manual-trigger gates are telemetry. hotdog's 7 is honest -- big property-asserting corpus, no blocking
eval, no fuzz, one job -- and 8 is reachable with no eval at all by fixing the wiring, which is exactly
what quality-gates-unwired (11 subjects of warn-only "mechanically enforced" jobs, vtcode-c4) and
test-suite-without-ci (15 subjects) show happens when you don't.

**1. faux-provider-testing -- drive the shipped binary** | 47 subjects (48 findings) | T1111 = 4 |
score 188 | effort M | evidence: ob-1-4, SWE-agent-3, cline-2, hax-4, grok-build-s10, tura-9 |
our gap: `hotdog-15`
Build: keep the in-process stream mocks, add a tier that spawns `bin/hotdog` as a subprocess against
a loopback scripted-wire server and asserts what reaches the provider; keep the vendored
openai-openapi fixtures as the oracle (derivation stays "never from hotdog output"). Landing
module: **tests/** conformance tier, plus `-p` paths in **ui-one-shot**.
We are mid-cluster (~rank 44/48) precisely because everything is in-process, and the tension in the
cluster is exactly this: in-process mocks prove loop logic but never exercise the artifact. The
independent-oracle half of our story is corpus-unique; the real-binary half is missing.

**4. evals-in-ci -- one blocking, cost-capped live gate** | 29 subjects (32 findings) | T1111 = 4 |
score 116 | effort M/L | evidence: ouroboros-5 ($30 cap, honest skip), kilocode-b5 (publish needs
the smoke edge), jcode-s7 (greps the log for skip messages and fails) | our gap: verification 7 row
-- no in-CI evals
Build: one nightly (then release-needs) lane that runs a handful of real tool-calling scenarios
through our local-first providers with an explicit dollar ceiling and a loud, job-failing skip when
the cap or key is missing; zero `continue-on-error`. Landing module: **.github/workflows/test.yml**
plus a small scenario runner under **tests/**.
This is the item that can carry a future 9, and the honest caveat is that ours must be
snapshot-auditable: thresholds checked into the repo, not a private harness (see non-move 5).
Pair it with the cheap CI hardening that the 7-rung dock actually cited: matrix, coverage gate
(nanocoder-8 fails PRs on coverage drops), and the subprocess tier from rank 1.

**14. architecture-contract-test** | 11 subjects (12 findings) | T1101 = 3 | score 33 | effort S |
evidence: kode-cli-8 (reverse-edge ratchet that may shrink), kolkrabbi-7 (ratchet paid to zero,
fails on fixed-but-still-listed violations), minicode-7 (docs-map drift gate) | our gap: the
architecture 8 is verified by grep today, not by CI
Build: assert the core-never-imports-extensions law and freeze a per-file LOC ceiling so
`websocket/server.ts` (1,510) and `task-manager` can only shrink. Landing module: **CI test tier**.
Low score, high defensive value: the cheapest way to keep "largest file 1,660 LOC" true while ranks
1-13 add code, and T4 by construction.

## Economy + cheap interop (ranks 2, 5, 6, 7, 9, 10, 15, 16)

Token-economy is the dimension where solo projects match institutions, and we are already on the
correct side of both live disputes: our trigger projects the outgoing request at wire size
(`hotdog-10`, measured through the resolved WireFormat) rather than lagging on last turn's usage,
and our overflow rescue is LLM-free with a misclassification guard (`hotdog-9`). So these items are
refinements of a position we hold, not acquisitions. Interop appears once, deliberately (rank 15).

**2. compaction-tiering** | 43 subjects | T1110 = 3 | score 129 | effort M | evidence: crab-code-1
(7-level, no-LLM first, circuit breaker), minicode-3 (anti-thrash breaker, compaction spend booked
into the session), jazz-b8 (config-enforced rung ordering)
Build: order the existing five strategies behind explicit cheapest-first rungs with a breaker that
stops re-compacting a session that stopped shrinking, and book summarizer spend against the session.
Landing module: **compaction extension** (registry + `utils.ts`).

**5. loop-detection** | 36 subjects | T1110 = 3 | score 108 | effort S-M | evidence: 3code-2
(four-signal fingerprint window, Jaccard near-dup clustering, zero tokens), crush-1, kimi-cli-2
(graduated reminders 3/5/8, stop at 12) | our gap: `hotdog-14` (~#21/38)
Build: add result-byte equality and a bounded near-duplicate window to the canonical-arg-hash
detector, keep nudge -> stronger nudge -> stop. Landing: **loop-detect/detector.ts**.

**6. prompt-cache-marking** | 29 subjects | T1110 = 3 | score 87 | effort S | evidence: memcode-5
(stable/volatile split, clone-only decoration, no-mutation property test), ob-1-8 (cached stable
block + uncached tail), keen-code-2 (plan-mode denial wraps tools so the tool-def prefix survives)
Build: split the system prompt into stable and volatile blocks and place provider breakpoints for
the Anthropic-class formats; the property test (decoration clones, never mutates) is the deliverable.
Landing module: **llm-client/serialize.ts** + prompt assembly. Today `cache_control` appears nowhere;
KV-warm placement (`hotdog-6`) is a local-first substitute, not the same claim.

**7. workflow-resume-journal -- extend our own top-tier cluster** | 19 subjects | T1111 = 4 |
score 76 | effort L | evidence: kolega-code-2 (resume matched by content key, survives script edits
and resume-of-resume), qwen-code-e5 (a surviving checkpoint IS the interrupted-run signal) | our
position: `hotdog-4` is #1 in this cluster
Build: content-addressed node keys so a resumed run survives workflow-file edits, then journal the
plain-subagent path the same way -- today only workflow runs survive a crash. Landing module:
**workflows engine** + subagents. Highest T4 in the doc: the orchestration we ship, made edit-tolerant.

**9. usage-measured-compact-trigger** | 23 subjects | T1110 = 3 | score 69 | effort S | evidence:
opencode-e1, kilocode-e1, gemini-cli-e2 (real prompt-token counts + inflation veto) | our gap:
`hotdog-10` is projection-family on chars/4 arithmetic, no provider-usage feedback
Build: feed the provider's reported prompt-token usage back as a correction factor on the wire-size
projection, and keep ferrum-3's rule -- if the post-compaction projection still does not fit, block
the send rather than shipping it. Landing module: **compaction/utils.ts** + `token-tracker.ts`.

**10. cache-monotonic-compaction** | 19 subjects | T1110 = 3 | score 57 | effort M | evidence:
atomic-agent-3 (stable prefix byte-identical, tests assert byte-equality), ouroboros-4 (append-only
byte-prefix invariant, every unsanctioned cache break counted), mocode-3 (summarizer fork reuses the
parent prefix so compaction itself is a cache hit)
Build: make the compaction seam the only sanctioned cache break and count the others; summarizer
requests reuse the parent prefix. Landing module: **context/compaction**. Pairs with rank 6 --
breakpoints without a monotonic prefix just measure your own churn.

**15. headless-contract -- the one interop move** | 7 subjects | T1101 = 3 | score 21 | effort M
(est) | evidence: opencode-e10 (raw event stream, resume/fork flags, export with secret redaction),
grok-build-e14 (documented machine-parseable output formats, script-safe resume), roo-code-e11
(typed NDJSON with requestIds) | our gap: interop 5, `-p/--json/--json-schema` undocumented
Build: publish the existing one-shot surface as a versioned contract -- stable event schema, exit
codes, documented resume semantics, redaction -- tested by golden files. Landing module:
**ui-one-shot** + docs. Cheapest honest interop: no protocol to negotiate, no external counterparty.

**16. turn-budget-accounting** | 5 subjects | T1111 = 4 | score 20 | effort S | evidence:
codewhale-e6 (per-turn budget never reported as success, fast-fail BudgetSnapshot), code-e4
(coordinator turn cap hard-stops runaway runs) | our gap: orchestration 8 docks "no token budgets on
tasks"
Build: a per-run token/spend cap enforced in admission, exhaustion surfaced as its own TurnEndReason
rather than completion. Landing module: **TaskManager** + lane ledger. Small concept, maximum fit
sum, and the thing that makes a nightly eval lane (rank 4) safe to run.

## Non-moves (8), with the data that killed them

These are real clusters with real convergence. They lose on thesis fit, not on evidence, and each
is a place where chasing a scored gap would cost the properties that earn our 8s.

**N1. Full ACP server surface.**
Data: 13 subjects carry an acp-* concept (14 findings); the server-side candidates are thin --
acp-server-surface is 2 subjects (zeroclaw-e9, workground2-e10), acp-internal-contract 5
(grok-build-c2, prime-agent-9, nanocoder-6, qqcode-8, kimi-cli-9) and its pattern, "every product
surface is a client of one session actor", is a refactor of our host/TUI split rather than a protocol
file. vtcode-e10 shows the attached cost: Zed workspace-trust defaults to FullAuto.
Mismatch: T2 outright -- version negotiation, capability matrices and a compatibility promise across
editors is a contract a solo maintainer cannot carry; T3 zero. Rank 15 buys the machine-consumable
benefit at a fraction of the surface.

**N2. MCP-server mode (hotdog as a server).**
Data: mcp-client-only is 5 subjects (opencode-e11, grok-build-e15, zeroclaw-e10, jcode-e8,
letta-code-e11) and interop-ceilings is 4 (bitfun-e10, zeroclaw-e14, opensquilla-e15,
orca-agent-e10) -- filed as ceiling gaps against the top anchors, not as portables with evidence of
demand. The strongest datapoint is negative: oh-my-pi-e10, the most interop-maximal subject in the
corpus, still ships no MCP server and no IDE extension.
Mismatch: T3 zero, T2 negative (a second protocol surface plus its auth story), and it drags a trust
question rank 8 has to answer first. Our client is already conformance-tested; being consumed by
rival harnesses returns no tokens to the user.

**N3. Published SDK / protocol artifacts.**
Data: this barely exists as a convergent concept -- no-published-sdk is 1 subject (hermes-agent-e10,
nuance) and interop-outbound-and-sdk-missing is 1 anti-pattern (vtcode-e9). Where SDKs do ship, the
findings are about the tax: codewhale-e10 needs version-parity gating in release CI to publish a
runtime SDK, and code-e7's shipped SDK directories are verbatim upstream artifacts pointing at
another product (unrenamed-manifest-identity, 4 subjects).
Mismatch: T2 -- an SDK is a support contract with a deprecation policy. Our distribution posture
(`hotdog-13`) is the corpus's strongest precisely because nothing else is versioned in lockstep.

**N4. IDE surfaces (VS Code / JetBrains / webview hosts).**
Data: god-file-loop is 25 subjects and god-file-host-wiring 13, the corpus's loudest structural
anti-patterns, and the host-sprawl evidence is specific: code-c2's TUI is a 44,921-LOC single file
with 1,048 methods and a 2,262-line event handler; vtcode-c1 runs three live turn engines
(interactive, headless, ACP) agreeing by comment above vtcode-c2's 2,330-line session loop;
continue-c1 welds the loop into a Redux thunk while continue-c2 re-implements compaction and
permissions in the CLI; kilocode-c9 keeps the message renderer three times beside a 6,110-line
webview host. interop-surface-breadth (7) is where these get counted as wins.
Mismatch: T2 fatal. Architecture 8 rests on a 928-LOC loop, core-never-imports-extensions and a max
file of 1,660 LOC -- rank 14 defends that. An IDE surface is how band-A harnesses become god-crates.

**N5. Private / benchmark-farm eval infrastructure.**
Data: eval-harness-outside-ci is 7 subjects, all anti-pattern (agentless-4, ob-1-12, ra-aid-9,
waveloom-14, amazon-q-developer-cli-b5, jazz-b5, vtcode-s9) -- harnesses that exist but gate
nothing. The trust cost of the opposite extreme is on the record: the study's own verification-9
re-audit kept kilocode's gate but retained the flag that "pass-threshold scripts live in private
kilo-bench, snapshot-unverifiable" (calibration-notes:1040), and grinta's 9 fell because its trigger
pipeline did not block (calibration-notes:1010); no-fuzzing-advisory-evals adds 3 more.
Mismatch: T3 inverted -- a bench farm spends our money, not the user's -- and T4 only holds when the
gate blocks and is auditable. Rank 4 is the honest subset: one capped, blocking, in-repo lane
(ouroboros-5's written $30 cap is the model).

**N6. OS-sandbox-first (kernel jails before the default gate).**
Data: sandbox-delegation is 32 subjects / 34 findings but the candidate efforts are L and XL
(codex-2 XL, cline-7 L, crush-4 L); sandbox-absent is 5 subjects all XL; windows-sandbox-gap is 3
subjects (workground2-s2 fails open to unwrapped execution on Windows and bwrap-less Linux;
deepseek-reasonix-s7 is XL and has nothing on Windows). The hidden-off rows are the cautionary set:
claw-code-2 announces a fail-open to bare `sh` as "filesystem isolation is still active",
zeroclaw-s2 compiles its Landlock/bubblewrap backends out of the default build, vtcode-s1's jail is
real but `enabled` defaults false -- phantom-safety-control (15 subjects, 17 safety-holes) is what
accrues when a jail is advertised rather than bound.
Mismatch: T2 -- seatbelt + Landlock + seccomp + a Windows story, re-verified against every host
upgrade, is not solo-sustainable, and the doctrine does not pay for it: a jail behind an
off-by-default gate still takes the zero-binding-default cap of 5. Rank 3 first; only then is a jail
an opt-in layer that could earn the +1 rather than a substitute for the rung.

**N7. Provider breadth beyond the wire-format abstraction.**
Data: provider-plane-fork is 2 subjects (open-codex-1, open-interpreter-4), rival-subscription-
transport 4 (atomic-agent-11, grinta-coding-agent-10/b4, molt-7, zot-b3), and
oauth-client-impersonation 3 -- including the safety-hole variant open-interpreter-b3, which ships
rival client identity headers ("claude-cli/2.1.158 (external, sdk-cli)"). The sprawl endpoint is
continue-e7: 67 non-test provider adapter files, counted as breadth and paid for in license-risk and
ToS findings.
Mismatch: T2 and T3 -- each transport is maintenance plus impersonation risk, while the llm-client
WireFormat seam and `hotdog-1` mangling already cover what actually differs per provider (how markers
and roles ride the wire). Add a provider when a user needs one, not to raise a concept count.

**N8. Injection screening layers (content-fences and LLM judges on tool output).**
Data: injection-screening is 14 subjects / 14 findings but exactly **one** portable; the rest are 6
safety-holes and 6 nuances, and the evidence is self-defeating -- vtcode-s6's probe is LLM-based,
full-auto-only and swallows its own failures; continue-s5's sanitizer has exemplary real-shell tests
and is called by nothing; gemini-cli-s7 and zeroclaw-s3 sit in the same band.
Mismatch: T3 -- per-turn screening spends user tokens to approximate what we get structurally:
`hotdog-1` aliases every protected marker to a per-session CSPRNG value at the serializer, so
forging a harness message means guessing a 16-char alias that never appears in model-visible text,
and `hotdog-2` covers the human-render side. A heuristic layer on top adds cost, false negatives,
and a phantom-control surface rank 13 would then have to police.

## The three-move minimal program

Weighted contributions per the rubric weights in `scores/hotdog.json` (verification 15, safety 10,
token-economy 10): each safety rung is worth 1.0, each verification rung 1.5, each token-economy
rung 1.0. Current total 68.5, band B (3.5 above the B/C floor, 9.5 below the A floor).

**Move 1 -- ship the gate on (rank 3, plus rank 13 as its guard).** safety-enforcement 5 -> 6.
The flip is small because the fail-closed machinery and its matrix tests already exist; what changes
is the default and the prose. Under the amended doctrine the binding default posture earns the 6,
and the +1 opt-in credit we cannot currently take becomes irrelevant to the rung.
Delta: 1.0 x 1 = **+1.0 -> 69.5**. Honesty constraint: this move is only legitimate with rank 13
attached; a default we cannot prove bound is how the corpus's phantom rows get scored rung 2.

**Move 2 -- make the tests drive the binary and the CI block (ranks 1 + 4, plus rank 14).**
verification 7 -> 8. The dock that produced 7 was named precisely: one ubuntu job, coverage run-not-
gated, everything in-process, no fuzz. Subprocess conformance against `bin/hotdog` behind a scripted
wire, a coverage gate, a matrix, and boundary/LOC ratchets clear 8 without any eval at all -- and the
corpus's blocking-only rule means the nightly live lane (rank 4) is what later makes a 9 arguable on
a path that cannot stay green through regression.
Delta: 1.5 x 1 = **+1.5 -> 71.0**.

**Move 3 -- cache discipline with property tests (ranks 6 + 10, with 9 as the trigger fix).**
token-economy 7 -> 8. `cache_control` appears nowhere in src, so today the story is local-first only:
measured wire-size trigger (`hotdog-10`), five strategies, LLM-free overflow rescue (`hotdog-9`),
net-of-cached accounting. Stable/volatile prompt splits with provider breakpoints, a byte-identical
prefix invariant whose tests assert byte-equality, and counted cache breaks is the cluster where solo
projects match institutions -- T3 in the purest sense, every hit discounts the user's bill.
Delta: 1.0 x 1 = **+1.0 -> 72.0**.

Total after three moves: **72.0**, still band B, 6.0 under the A floor instead of 9.5. Two gaps stay
open on purpose: interop 5, because the only honest move there is rank 15 (1.0, buying no gate) and
N1-N3 explain the rest; and durability 4, 2.0 weighted points no code can buy. Sequence is
load-bearing -- rank 3 makes the safety claims true, rank 13 makes them provable, rank 14 keeps the
architecture 8 intact while all of it lands.
