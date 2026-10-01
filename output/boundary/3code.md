# Boundary re-review: 3code (Nim, MIT, T1)

Provisional 65.75 B (within 2 of B floor 65). Independent recount: **68.0 B**. B floor HELD.

One sentence before scoring: closest anchor is **crush (69.0)** -- solo-scale, real tested
mechanisms (loop detection, resume), distinctive design positions, verification below the
anchors' 8-corps tier; not pi (no extension surface, no session tree, no RPC plane).

Shallow clone honored: no activity claims made (HEAD 4 days old, `.git/shallow` present).

## Q1. Safety 7: external fence engine + default-OPEN network -- defensible?

**Ruling: 7 stands.** The two doubts raised are real but dock from 8, not to 6.

**Why it is above the 6 rung ("tested approval, nothing underneath").** There is no
approval rung at all -- what exists underneath is a kernel fence that ships ON. Default posture
(sandbox.nim:64-83, threecode.nim:141-163): unless `[settings] sandbox = off` or `--no-sandbox`
(src/threecode.nim:336), `initSandbox` loads the built-in default policy and sets `active =
true`; the bash tool re-execs the binary as `3code sandbox restrict ...` (streamexec.nim:583-652)
applying Landlock/Seatbelt/Windows restricted tokens per launch; the in-process
read/write/patch tools carry their own gate `checkRawPath` (sandbox.nim:499-524) that stays in
force even when the OS backend is gone (threecode.nim:168-169). Enforcement tests are real and
run in CI on runner kernels: `sandbox restrict blocks writes outside the writable path`
(tests/core/test_cli_args.nim:288-328, gated on a backend probe at :264), symlink/traversal
regressions on the in-process gate (tests/core/test_sandbox_traversal.nim), web_fetch network
gate (tests/core/test_webfetch_gate.nim), wall-proxy lifecycle + netns e2e
(tests/core/test_wall_bash.nim). Shipped-default doctrine: enforcement ships ON (contrast
octomind, where EVERY layer shipped disabled -> 5).

**"External engine" -- the framing needs correction.** sandwall is a compile-time library
dependency pinned by revision+checksum in nimble.lock (nimble.lock:14-17, `requires "sandwall >=
0.5.8"` threecode.nimble), and its CLI is folded INTO the 3code binary: "we ship one binary
instead of two... it re-execs *itself* (via `getAppFilename`), so there is no PATH lookup and no
separate sandwall binary to find or bundle" (box.nim:1-10; sandbox.nim findProcbox :305-310).
This is the kimi/pi-tui shape (pinned lib, not a product covariate like openhands/swerex
out-of-process), so the out-of-tree accounting rule applies: zero credit for sandwall's LOC,
zero blame for its internals -- but the 3code-side integration that makes the fence correct IS
in-tree and tested (policy materialization sandbox.nim:120-139, hidden guard rules :404-435,
gate-canonicalization parity :450-497, backend probe :312-358). Residual honest caveat: the
kernel-application code itself (Landlock ruleset construction, seatbelt generation) is not
auditable in this tree; 7 already encodes that ceiling -- 8 requires the codex-grade "enforcement
AND its tests AND its completeness all visible in one tree."

**Default-open egress is the bigger hole and it is exactly why this is 7, not 8.** The shipped
default policy ends with `allow *` (sandbox.nim:82) -- filesystem confinement by default, network
fully open; the wall-proxy net fence activates only "until the effective policy names its first
host" (sandbox.nim:165-167). Exfiltration -- the primary attack path against coding agents -- is
unfenced out of the box. Contrast codex (safety 10): network blocked by default with a separate
egress approval channel (seatbelt_network_policy.sbpl, network_approval.rs). Filed as
safety-hole 3code-b2. It does not drag to 6 because the filesystem half genuinely binds, is
on-by-default, and is denial-tested; per the corpus ladder (7 mocode / 7 forge / agentty), a
default-on kernel fence with disclosed gaps sits at the top of "solid, one design, tested."

**Fail-open fallback, honest variant.** If the backend probe fails (no Landlock, seccomp'd
container, Windows setup not run), `procboxExe` is cleared and bash runs UNCONFINED (threecode.nim:
168; streamexec.nim:555-556 "announced once at startup... not refused"; unconfined setsid path
:653-660), with a magenta stderr line stating exactly what is lost ("bash runs unconfined...
read/write/patch tools stay confined", threecode.nim:477-486). This is the honest pole of the
fail-open family: it degrades AND tells the user what degraded -- the inverse of claw-code's
runtime-lie. Probe-hang = true (unknown-success, :318-325) deliberately errs toward keeping
confinement. Filed 3code-b3.

## Q2. Anti-approval kernel-fence thesis: implemented or manifesto?

**Implemented -- and it is the corpus's first true fence-only tool path.** The thesis lives in
docs/manual.md:1244-1258 ("Kernel enforcement, no questions": judge models = "security by vibes",
approval prompts = "security by interruption... train users to mash yes", with an honest
calibration paragraph admitting "most coding agent sessions run effectively unfenced anyway").
Code check for REMOVAL of approvals, not coexistence: `grep -E "confirm|approv"` across
engine.nim / turns.nim / actions.nim returns ZERO hits in the tool-execution path -- the only
"approval" strings in the tree are OAuth device-flow UX (auth_xai.nim:100, wall.nim:282 UAC).
`runAction` (actions.nim) dispatches tools straight into the fence: bash wrapped by box
(streamexec.nim:591-652), file tools gated by `checkRawPath` (sandbox.nim:499), web_fetch gated
by `checkRawHost` against the same proxy matcher (sandbox.nim:170-178). The prompt layer tells
the model to "pause only before destructive, external, or scope-expanding action"
(prompts.nim:2474) -- model discretion, not an enforced gate; the ENFORCEMENT is entirely the
fence. Design-position census datapoint: corpus now has approval-first (codex), fence+floor
(minicode), and fence-only (3code). Filed 3code-b1.

## Q3. Verification: what blocks?

- Blocking test jobs on push/PR: linux-amd64 test job runs `sh tests/tools/ci_tests.sh 1500`
  with exit-code propagation (linux-amd64.yml:142-181); linux-arm64 (:210) and osx (:170)
  equivalents; windows runs the tty category only (:143) -- coverage asymmetry, noted.
- Notably honest CI engineering: arm64 comment documents that the nimscript task "swallows a
  non-zero testament exit... so a failing test reports green" and calls testament directly to
  fix it (linux-arm64.yml:199-201); the ci_tests.sh wrapper exists to attribute hangs, not hide
  them (linux-amd64.yml:173-179).
- Test corps: 125 test files, ~33.1k test LOC vs ~32.4k src LOC -- near 1:1 ratio; wire-level,
  kernel-level and TTY-level (tests/tty_expect.nim 1,945 LOC expect harness; mock_server.nim
  faux-provider).
- Byte-reproducible builds ARE blocking (linux-amd64.yml:86-97: double clean build, sha256
  compare, mtime-perturbed archive re-pack compare) -- but that is supply-chain, per the amazon-q
  ledger this gets credit in the supply-chain column, not as a property test.
- NOT present: fuzzing of any kind, model evals in CI. ERRATA ceiling of 8 applies; with the
  windows asymmetry and corps size below codex/cline/pi, **7.5**.

## Q4. Ten-dimension pass

- **architecture 7**: deliberate layering with stated contracts -- turns.nim owns turn
  lifecycle, "api.nim should stay transport/protocol focused" (turns.nim header), fatprompt owns
  visual side effects and "api.nim must not import this module" (fatprompt/runtime.nim header);
  library.nim is a headless frontend driving the SAME runTurns as the CLI (library.nim:1-10).
  ERRATA >5k check passes (largest product file prompts.nim 3,988, mostly prompt text). Docks:
  pervasive module-level mutable globals (sandbox.current/active, sandboxEnabled config globals
  threaded everywhere), api.nim 3,444, minline.nim 2,715 second-guesses the editor layer. crush
  rung.
- **verification 7.5**: see Q3. Blocking multi-OS, denial-tests-the-fence, faux-provider, expect-
  TTY; no fuzz/evals; windows partial.
- **safety-enforcement 7**: see Q1. Kernel fence default-on + tested + honestly degrading; egress
  default-open and out-of-tree completeness cap it at 7.
- **token-economy 7.5**: compaction trigger is usage-MEASURED -- `decideContextAction(usage.promptTokens,
  window, ...)` on provider-reported tokens, not a heuristic estimate (turns.nim:847,
  compact.nim:215-224, threshold 0.8, keep-last-8, at-most-once-per-turn); resume replays
  byte-identical wire history explicitly to keep provider prompt caches hot (session.nim:44,892)
  with a dedicated "prompt cache stability" suite (tests/core/test_prompt_cache.nim:4); cached-
  token receipts surfaced in the UI (docs/manual.md:535-582). Docks: single-tier compaction, no
  warming, no projected-context measure (pi tier sits above). Joins usage-measured-compact-trigger
  cluster.
- **orchestration 6.5**: harness-side doom-loop breaker, zero token spend: fingerprint window +
  streak ring + Jaccard near-dup clustering, escalate x3 then abort the turn
  (turns.nim:16-30,228,315; tests/core/test_flail.nim) -- loop-detection rung with a
  fingerprint-similarity nuance. Deliberate no-subagents design (manual.md:1260-1266); oneshot
  `-r` scripting plane is cron-friendly (manual.md:1268-1288). No budgets, no queues, no resume
  journal beyond session files.
- **interop 5**: embeddable-library API (library.nim) + oneshot headless mode; MCP only as a
  client of two keyless web-search endpoints (web.nim:9-11,238); no MCP server, no ACP, no IDE
  surface, no published SDK. Multi-provider OAuth (anthropic/openai/google/xai).
- **operability 7**: per-cwd sessions with byte-exact resume (`3code -r`), `:sessions`/`--list`,
  dmail checkpoint/revert in-turn (actions.nim:110-126, turns.nim:697 owns the revert), auto-
  update channel, startup trace, stale-temp sweeps (sandbox.nim:290-303, streamexec.nim:536-547).
  Crash posture: turn exit code contract for scripting (manual.md:1289-1292). No fork/tree plane.
- **originality 7.5**: fence-only anti-approval product (Q2, verified implemented); policy-scoped
  wall proxy with per-run netns plumbing, env-based proxy injection incl. GIT_SSH ProxyCommand
  (sandbox.nim:256-288); hidden guard rules that lock the policy files read-only inside the
  sandbox and cannot be weakened by any policy text (sandbox.nim:404-435); byte-reproducible
  pinned-toolchain releases (linux-amd64.yml:20-97); a phone-class Termux build with honest
  "Android has no OS sandbox" caveat (README.md:88). Docked from 8+ because several parts have
  corpus priors (landlock-self-sandbox, reproducible-release-builds, loop-detection).
- **durability 4**: solo author (threecode.nimble), no SECURITY.md, no governance surface,
  self-hosted distribution site + forum as infra; shallow clone -- activity evidence-limited, no
  claims made.
- **docs-dx 7.5**: manual.md + reference.md are substantial and match behavior (design-decisions
  section argues positions honestly, including the unfenced-in-practice calibration at
  :1256-1258); installer for curl/irm/termux (README.md:59-88); the fail-open warning text is
  failure-copy engineering (names the cause and the fix-flag). Docked for external-docs-only
  release pointers.

Weighted: 7*15 + 7.5*15 + 7*10 + 7.5*10 + 6.5*10 + 5*10 + 7*10 + 7.5*10 + 4*5 + 7.5*5
= 105 + 112.5 + 70 + 75 + 65 + 50 + 70 + 75 + 20 + 37.5 = **68.0 / B**.

## Band call

68.0 vs floor 65: B floor HELD with 3.0 margin (provisional 65.75 was under-credited on
token-economy: the compaction trigger is provider-usage-measured with a cache-stability test, not
an estimate; and on verification's test-shape: kernel-denial e2e + expect-TTY + faux-provider is
above the crush-7 default). Lenient bound (safety 8) = 69.0; stingy bound (verification 7,
safety 6) = 65.75 -- the whole sensitivity range stays B. No calibration rule applies (provenance
original, MIT both file and manifest, not archived; shallow clone: no activity claims).
