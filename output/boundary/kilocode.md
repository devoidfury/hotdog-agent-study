# Boundary re-review: kilocode (divergent-fork:opencode) - T3, A-floor (77.5 provisional, -0.5)

Independent pass per review-protocol.md; anchors + ERRATA + calibration doctrines read.
Independence note: subjects/kilocode.md, scores/kilocode.json, findings/kilocode.jsonl NOT read.
Concept dedup done by concept-id grep only, per boundary-grep norm (claurst/keen disclosure).

**Pre-scoring anchor sentence:** closest anchor is **pi (78.5)** - same upper-half shape: a strong
single core engine consumed by thin product surfaces, honest safety self-assessment, cache-aware
economics; cline (76.5) is the safety-rung reference ("tested approval, nothing underneath" = 6).

## Identity / activity
- Head 7d977bce (2026-09-26, "Merge PR #14526"); shallow clone - remote checked per codel lesson:
  `git ls-remote origin HEAD` = 490246ec, AHEAD of snapshot -> active, rule (b) n/a.
- Provenance divergent-fork:opencode holds; fork is in-tree annotated (`kilocode_change` markers)
  and annotation drift is CI-gated (check-opencode-annotations.yml:1-40, PR-blocking on
  packages/opencode/** and shared paths); upstream releases polled every 30 min
  (watch-opencode-releases.yml:1-2). Rule (a): divergent fork may outrank upstream on demonstrated
  divergence; divergence is real and verified below (bash-ask default, opt-in kernel jail, prompt
  queue, fork governance).

## Scale (recount; census-lessons applied)
- All TS ~925.7k LOC; tests 1,678 `*.test.ts` files / 407,237 LOC (opencode package alone: 218,300
  test LOC). Largest PRODUCT TS file: session/prompt.ts 2,673; then provider.ts 2,210,
  transform.ts 2,020 - ERRATA >5k product-file check PASSES.

## Dimension findings (file:line)

### architecture 8
- Engine/services/surfaces separation intact from the opencode-8 shape: one loop
  (session/prompt.ts:1543 `while (true)`), Effect service layering, sqlite state
  (@opencode-ai/core/database), all surfaces (TUI, run.ts headless, VS Code 257k-LOC ext,
  JetBrains, ACP acp/service.ts, httpapi server) drive the same session engine.
- Kilo additions are mostly new namespaces/packages (kilocode/*, kilo-memory, kilo-sandbox,
  kilo-sessions) rather than fusions into the core. Docking considered: prompt.ts weaves product
  features (prompt-queue projection :1554, plan-followup :1586-1601, review telemetry, goal
  tracking) into the loop - real weaving, but cline earned 8 with a 2,825-LOC fused runtime host;
  2,673 with annotation-visible seams is the same rung, not 9. Opencode itself re-scored 8 at its
  boundary; 8 does not credit the fork above where upstream scored (oh-my-pi rule-(a) discipline).

### verification 9 (FLAGGED: weakest blocker class that qualifies)
- 407k-LOC test corps, blocking push/PR matrix on Linux+macOS+Windows
  (.github/workflows/test.yml:74 matrix; the only continue-on-error steps are Defender
  exclusions :106, setup-node :131, artifact upload :255 - not the test steps).
- Real kernel-confinement property tests EXECUTE IN CI:
  test/kilocode/sandbox/macos-confinement.test.ts:189 `describe.skipIf(darwin)` runs real /bin/sh
  inside real seatbelt, asserts write-inside OK / write-outside FAIL / write-.git FAIL
  (:216 expect `["confined", false, false]`), plus deny-write-subtree vs rename
  (:232-262). `it.live` is an ungated plain test (test/lib/effect.ts:77-78) - not a skipped tier.
- **Verification-9 qualifying evidence**: model evals that BLOCK release.
  publish.yml:584-597 `smoke-test: Smoke Test (pre-publish gate)` runs Harbor eval tasks
  (smoke-test.yml:116,126 run_eval.sh, real agent, cost-capped < $0.50, private kilo-bench)
  against the built release binaries; publish.yml:599-606 `publish: needs: [..., smoke-test]` -
  no continue-on-error. Per the corpus rule (gptme refinement: blocks-on-failure counts) and
  precedent class (prime-agent 28-task release gate = clean 9; openhands acceptance smoke = 9
  flagged), this qualifies. Task set is thin (2 tasks) -> ledger rank near the bottom, flagged
  "acceptance-grade release gate". Zero fuzzing (9->10 gate unmet).
- No in-PR model evals; suite is property-shaped (approval, ordering, confinement, wire tests).

### safety-enforcement 7 - see DOCTRINE RULING below
- Shipped default is NOT allow-all for the dangerous tool: bash ships
  `{"*": "ask", ...read-only allowlist}` (kilocode/agent/index.ts:52-53) merged over the
  tool-level base `'*': 'allow'` (agent/agent.ts:137-138 via :162-163 Permission.merge). Allowlist
  ~30 read-only binaries + gh read subcommands; a defense-in-depth denylist catches operator/write/
  exec-flag abuse of allowed commands, honestly labeled: "This is defense-in-depth, not a sandbox -
  the durable fix is OS-level sandboxing" (index.ts:104-134). Also ask-by-default: MCP servers
  (getMcpRules index.ts:364), external_directory (:139-141), `*.env`/`*.env.*` reads (:151-156),
  doom_loop loop-detection (:138 + processor.ts:525), kilo_memory tools (:378-383).
- Opt-in jail: enabled:false by default (kilocode/sandbox/config.ts:42) - honestly off.
  Fail-closed at BOTH gates: enabling with unavailable backend is refused
  (policy.ts:411 `if (enabling && !status.available) return status`), and exec inside an active
  jail FAILS rather than falling back to plain sh (kilo-sandbox/src/backend.ts:69-71,93-95
  `Effect.fail(unsupported(...))`). Escape attempts route through an explicit user escalation
  prompt `sandbox_escalation` "Run outside the sandbox" (cli/cmd/run/permission.shared.ts:101-111).
- Network inside jail defaults deny (config.ts:41,44); proxy mode pins destinations by exact
  public DNS host (destination.ts parseDestination; config.ts:10-17 schema filter); TLS ClientHello
  SNI must match the authorized destination exactly and ECH/early-data are refused
  (tls-client-hello.ts:73,88-94); non-public addresses rejected (destination.ts:54,72).
  Bundled setuid-adjacent bwrap shipped + version/license-validated in publish.yml:253-306.
- Jail config scope is monotone-narrowing: repo-sourced config may only tighten
  (sandbox/config.ts:48-58 `scope()` keeps only `enabled:true` and `network:deny` from local
  sources; allowed_hosts/writable_paths never widen from repo); untrusted configs can't expand
  `{env:}` interpolation and `{file:}` reads are confined (config/config.ts:335-346).
- Holds at 7, not 8: default posture has zero kernel enforcement (jail off), write/edit default
  allow, and SECURITY.md is stale about the jail (finding b3). Status events report enabled/
  available truthfully (policy.ts:64-77) - no runtime-lie variant.

### token-economy 7.5
- Upstream caching tail (transform.ts:385-406) + kilocode provider-specific cache-breakpoint
  support (supportsPromptCacheBreakpoint, transform.ts:408-411). Kilo additions: prune reasons
  tagged to cache-invalidating boundaries (compaction.ts:44-46,98-99), old tool-output clearing
  ("[Old tool result content cleared]" :88), 4MB payload-limit compaction recovery that strips
  tool outputs/media and re-runs (kilocode/session/compaction-payload-recovery.ts:12-27),
  cost propagation to parent for parallel subagents with lost-update locks
  (kilocode/session/cost-propagation.ts:7-15). Still no projected-context trigger (pi pillar)
  and no realized-cache-hit validation -> below the 8 cluster (san/jazz measured economics).
- opencode boundary declined 7->8; kilocode's additions are a half-rung, not the full pillar set.

### orchestration 7.5
- Retained prompt queue with chronology-based loop state (prompt.ts:1554,1566-1568,
  KiloSessionMessageOrder "compare chronology, not generated IDs"), session fork command
  (kilocode/session/fork.ts, fork-command.ts), background process tool + supervisor
  (kilocode/background-process/index.ts 1,270), experimental shared agent board
  (kilocode/board/, agent/index.ts:380), task subagents with cost propagation, session-resume
  service 1,184 LOC, continuation/goal machinery, interrupted-orphan handling in loop
  (prompt.ts:1590 isOrphanedInterruptedTool). Crash semantics tested at unit level but no
  journal/queue plane of codex grade; no budgets.

### interop 8
- MCP client (mcp/index.ts 1,164), ACP server (acp/service.ts 1,107 + acp/permission.ts bridging
  approval incl. sandbox escalation), published SDK packages (sdk, sdk-next), surfaces: VS Code
  (257k LOC), JetBrains (Kotlin + test-jetbrains.yml + publish-jetbrains.yml), web UI, headless
  server (httpapi groups), GitHub handler for issue/PR invocation (cli/cmd/github.handler.ts
  1,624), npm-plugin system, opencode-config compatibility inherited. cline-8 rung plus JetBrains;
  no ACP-client role, no LSP server export.

### operability 8
- sqlite sessions + revert/snapshot (snapshot/index.ts 1,041, revert.ts), /fork, share via cloud
  ingest queue (kilo-sessions/kilo-sessions.ts + ingest-queue, remote-sender.ts 1,511), session
  export incl. compaction capture (compaction.ts:733-738 SessionExport.compaction), debug cmds,
  ACP duration profiling. Docker publish validated on both unix and windows (publish.yml
  validate-cli-*).

### originality 7
- Real, code-verified fork-deltas: opt-in fail-closed jail with SNI-pinned egress proxy + ECH
  refusal (own machinery; note @anthropic-ai/sandbox-runtime is a dep but the profile/proxy/
  SNI-authorization plane is kilo-written), monotone jail-config scoping, prompt-queue semantics,
  payload-limit recovery, and the corpus-unique fork-annotation governance (b4). Docked from 8:
  jail+egress has codex prior art on both axes; most surface is upstream lineage.

### durability 8
- Funded org, hyperactive (remote ahead; PR #145xx), 30+ active workflows + a disabled/ quarantine,
  CodeQL, SBOM actions, Dependabot auto-merge, multi-channel releases (npm/vsix/JetBrains/OCI),
  disclosure program in SECURITY.md. Shallow clone -> contributor count evidence-limited (census
  caveat); not 9: not codex/cline-scale governance visibility.

### docs-dx 8
- Full kilodocs site (packages/kilo-docs, markdoc/next), in-repo TESTING/REVIEW/AGENTS/CONTEXT/
  PRIVACY, human-readable schema annotations on config incl. sandbox defaults text
  (sandbox/config.ts:19-34), tips naming doom_loop (tui tips.ts:157). SECURITY.md staleness docks
  (b3), otherwise cline-8.

## DOCTRINE RULING (mandate 2: safety-enforcement doctrine adjudication)

**Corpus doctrine as stated:** nanocoder "real machinery + wrong default = below the 6 rung";
octomind "every layer ships disabled -> 5"; dispatcher reinforcement "shipped default posture,
full stop." Apparent collision: kilocode lane 7 rests on `agent.ts:137 '*':'allow'` + OPT-IN jail;
if doctrine forced 5-6, total drops to 75.5-76.5 and the outranks-opencode (+0.5) result reverses.

**Findings of fact (independent, this pass):**
1. The collision frame is factually incomplete. `'*':'allow'` is the tool-level base; kilocode's
   own defaults patch bash to `'*':'ask'` + read-only allowlist + honest denylist layer
   (agent/index.ts:52-53,86-134), MCP ask, external-dir ask, .env ask, doom_loop ask. The DEFAULT
   posture binds approvals on the dangerous tool and is tested (test/permission/ suite). Default
   posture therefore earns the 6 rung outright - kilocode is NOT the nanocoder/octomind shape
   ("nothing binding by default").
2. The opt-in layer is engineered against exactly the failure modes the doctrine punishes:
   refuse-to-enable without backend (policy.ts:411), FAIL-CLOSED at exec inside the jail
   (backend.ts:69-71,93-95 - no silent fallback to plain sh), explicit sandbox_escalation prompt
   for anything deliberately outside (permission.shared.ts:101-111), default-deny + SNI-pinned
   egress (tls-client-hello.ts:73,88-94), repo configs can only tighten the jail
   (sandbox/config.ts:48-58).
3. Enforcement is tested in CI, including real-kernel confinement on the macOS shard
   (macos-confinement.test.ts:189; ungated live tier per test/lib/effect.ts:77-78; macos job runs
   on push/PR per test.yml:74) and bundled-backend validation in publish.yml:253-306.
4. The SECURITY.md defect runs the OPPOSITE way from every rung-2 tell: it UNDERSTATES protection
   ("No Sandbox", sandbox escapes out-of-scope) while a kernel jail + bundled bwrap ships - never
   mentioning the jail. It is honest about the DEFAULT (correct: by default nothing kernel binds)
   and stale about the CAPABILITY. There is no runtime false-safety indicator (status events are
   truthful, policy.ts:64-77). Per the hax doctrine honesty never substitutes for enforcement; the
   symmetric statement is that a disclosure defect never re-rates enforcement machinery. Docked as
   security-posture-docs finding (b3), not by collapsing rungs.

**DECISION: (b) - there is a defensible distinction; the doctrine is amended with scope, not forked.**
The shipped-default doctrine governs which rung the DEFAULT posture can earn; the below-6 cap
attaches to SILENT degradation (nanocoder: jail exists, fails open to plain sh - wrong default AND
wrong failure mode) or to a default that binds NOTHING (octomind: no approval prompt in the tool
path). It does not force below-6 on a subject whose default binds tested approvals and whose opt-in
layer fails closed, is CI-confinement-tested, and is not deceptively advertised.

**General rule for the remaining T3 giants (same shape expected; gemini-cli's "enforcement
DEFAULTS" gap is the reference case):**
> Rung = rung earned by the binding DEFAULT posture, plus at most ONE rung for an opt-in
> enforcement layer that passes all three gates: (i) FAIL-CLOSED - both refuse-to-enable when the
> backend is unavailable and refuse-to-run (never fall back to unprotected) when the enabled path
> breaks; (ii) ENFORCEMENT TESTS IN CI - confinement/escalation properties executed by a blocking
> job, not merely present; (iii) NON-DECEPTIVE - no claim that the default is protected (stale
> underdisclosure of an existing jail = docs finding, not a rung collapse; runtime false-safety
> indicator remains a rung-2 tell per claw-code).
> Caps that hold regardless of opt-in quality: silent fail-open degradation of the opt-in path ->
> below 6 (nanocoder); zero binding default -> 5 (octomind). hax honesty rules unchanged
> (upward-only out of the misleading band, never above mechanics).
This keeps "shipped default posture, full stop" intact for what the default DOES, while making the
opt-in +1 an auditable mechanical test instead of vibes. Enforcement default ON (gemini-cli,
agentty) still outranks opt-in-max: the +1 is capped, never a substitute.

**Applied to kilocode:** default = 6 (bash-ask + path/env asks, tested) + gates (i)(ii)(iii) pass
=> **7 stands**. Gaps to 8: no kernel enforcement in the default, write/edit default-allow, jail
opt-in by design.

## Rule (a) check
Divergent fork outranking upstream 77.0 by +1.5 requires demonstrated divergence: verified above
(default bash-ask policy absent upstream, kilo-sandbox jail, prompt-queue/fork/board, fork
governance; no upstream-synced code credited above where opencode itself scored - arch 8 = opencode's
own 8). No sync-fork markers found (no daily-merge script; watch = release polling only).

## Scores
| dim | score | w | weighted |
|---|---|---|---|
| architecture | 8 | 15 | 12.0 |
| verification | 9 (flagged) | 15 | 13.5 |
| safety-enforcement | 7 | 10 | 7.0 |
| token-economy | 7.5 | 10 | 7.5 |
| orchestration | 7.5 | 10 | 7.5 |
| interop | 8 | 10 | 8.0 |
| operability | 8 | 10 | 8.0 |
| originality | 7 | 10 | 7.0 |
| durability | 8 | 5 | 4.0 |
| docs-dx | 8 | 5 | 4.0 |
| **total** | | | **78.5** |

**78.5 A - A floor CROSSED.** Sensitivity (disclosed, not hidden): verif->8 gives 77.0 (B, exact
tie with opencode); arch->7.5 alone gives 78.0 (still A); both together = 77.5 (exact provisional
match). The band turn rides entirely on the release-blocking smoke gate qualifying as verification-9
under the corpus's own blocking-only rule and the prime-agent/openhands/gemini-cli precedent class.
Distribution: kilocode joins A cohort as the study's first band-flip boundary that moved UP
(prior flips: down x2, confirm x many); flagged for the synthesis sweep of the B-ceiling plateau.
Outranks opencode: YES (78.5 > 77.0, divergence demonstrated; safety +1 differential survives the
doctrine challenge on the facts, not on leniency).
