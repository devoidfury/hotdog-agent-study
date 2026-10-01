# 3code — T1 review

Subject: 3code (Nim) — "The economical coding agent", capocasa/3code, MIT, v0.8.0.
Non-test .nim LOC: 32,628 (git-tracked); tests .nim: 33,085. Shallow clone (1 commit),
HEAD 2026-09-25 (fresh, 4 days at review) — no activity claims needed, no remote check
required for a currency claim direction only.

## Anchor question

**Which anchor subject is this closer to, and why?** crush (69.0, B): a single-author,
strongly opinionated agent whose distinctive mechanisms (3code: kernel-first fencing with
zero approval prompts, model-driven context folding, reproducible builds; crush: loop
detection, hook scoping) compensate for deliberately thin orchestration/interop, landing
in the same mid-B engineering-quality space rather than near pi's extension-rich A.

## What it is

42 single-purpose Nim modules under `src/threecode/`. One agent loop
(`turns.nim:617 runTurns`) shared by the CLI and a headless library frontend
(`library.nim:1-15` — "same runTurns, same tools, same sandbox"). Providers normalized
to OpenAI-shape wire (`api.nim`) with Anthropic/Google/XAI adapters. Anti-TUI by design:
plain scrollback + a volatile two-row "fatprompt" footer (`fatprompt/runtime.nim`).

## Dimension scores

### architecture — 7
- Module boundaries are documented and directionally enforced in comments:
  "`api.nim` must not import this module" (`fatprompt/runtime.nim:5`); "api.nim should
  stay transport/protocol focused" (`turns.nim:5-6`).
- Single loop consumed by every surface (`library.nim:3-4`), turn state explicit and
  bounded: length/steer/empty retry counters + flail detector at `turns.nim:646-656`.
- Dock: `runTurns`'s body runs ~560 LOC fusing model-call retry policy, empty-reply
  recovery, flail escalation, and transcript commits (`turns.nim:617-1181`); largest
  logic files `api.nim` 3,444 (all transports), `fatprompt/runtime.nim` 2,956 (stream
  hooks + terminal side effects), `minline.nim` 2,715 (line editor). `prompts.nim` 3,988
  is mostly static text, not a god file.
- Above crush (7): no fused loop+policy+wire god file like crush's agent.go; below cline
  (8): the loop proc itself is still the broad spot.

### verification — 7.5
- 33.1k LOC tests, and they prove properties: 26 flail-detector tests
  (`tests/core/test_flail.nim`), 26 dmail tests (`tests/core/test_dmail.nim`), 75 session
  tests (`tests/core/test_session.nim`), 12 sandbox-cascade + traversal tests
  (`tests/core/test_sandbox_cascade.nim`, `test_sandbox_traversal.nim`), 4 wall/egress
  tests (`test_wall_bash.nim`).
- Unique: a self-built cross-platform PTY expect harness driving the real binary under
  openpty (POSIX) / ConPTY (Windows) with screen-snapshot frame artifacts
  (`tests/tty_expect.nim:1-9,36-46`), 53 functional tty tests
  (`tests/tty/test_tty_functional.nim`, 3,885 LOC), SSE tests against a mock server
  (`tests/mock_server.nim`, `tests/stream/test_streaming_sse.nim` 27 tests).
- CI: 6 platform workflows; test runs wrapped in a watchdog that names hung test binaries
  (`threecode.nimble` test task; `.github/workflows/linux-amd64.yml` test job,
  `sh tests/tools/ci_tests.sh 1500`). Build job gates byte-identical rebuilds.
- No in-CI model evals (SWE-bench runs are ad-hoc docs, `docs/cache-analysis.md:3-8`),
  no fuzzing → capped at 8 per errata; lands above crush (7) on the full-binary tty
  corpus, below the 8 trio's breadth.

### safety-enforcement — 7
- Default-on kernel enforcement: filesystem policy applied to every tool call and bash
  jailed via Landlock/Seatbelt/Windows restricted tokens (external `sandwall` library —
  `threecode.nimble` requires sandwall >= 0.5.8; `box.nim:9,35,63`).
- Dual enforcement: in-process read/write/patch checks reload the policy on mtime before
  every restricted op, and the bash subprocess re-loads the policy file itself so a
  launch always enforces the freshest file (`sandbox.nim:36-40`). Policy files are
  appended as hidden guard rules no file rule can weaken (`sandbox.nim:26-31`).
- Network: POSIX host rules funnel bash egress through a CONNECT+SOCKS5 allowlist proxy
  (`sandbox.nim:15-16`, `wall.nim:8-14`); Windows uses a separate net fence (WFP).
- Honest posture doc: rejects judge-model approval ("security by vibes") and approval
  prompts ("security by interruption ... train users to mash yes"), states the real
  baseline honestly: "most coding agent sessions run effectively unfenced anyway"
  (`docs/manual.md` Design decisions, ~1247-1260). Matches code — tested cascade
  ordering, precedence one-file-never-cascade (`sandbox.nim:19-25`).
- Dock: default policy leaves network wide open (`allow *`, `sandbox.nim:82`) and the
  whole project dir writable by design (`sandbox.nim:71-81`); no per-action consent
  layer at all, so a single wrong allow line is unmitigated; the kernel mechanism itself
  lives outside this repo. Well above the 6 "approval, nothing underneath" rung (there
  is something underneath, and it is tested), below codex's 10.

### token-economy — 7
- Three context tiers with the model holding two of them: `dmail(checkpoint, msg)` lets
  the model revert its own conversation to a `[checkpoint N]` marker and carry a note
  forward, filesystem untouched (`prompts.nim:2629,2983-2987`, dispatcher
  `session.nim:705,923,1201`); the `clear` tool ends a chunk with handoff instructions
  for the next session (`actions.nim:106,125`; chunked/cybernetic modes
  `docs/manual.md:886-949`); auto-summarize as lossy fallback at 0.8 of window, keep-8,
  at-most-once-per-turn (`compact.nim:1-6,53-59`).
- Cache discipline without breakpoints: sessions persist verbatim wire bytes so "a
  resumed session re-sends byte-identical history and the provider's prompt cache stays
  hot" (`session.nim:645-655,890-893`); stable conversation-id header per session for
  provider cache affinity (`turns.nim:676-681`); cached-token accounting normalized
  across 6 provider usage shapes (`api.nim:273-283`).
- Cost visibility: live token bar repainted with accurate post-call usage, then
  committed as a "receipt" row in scrollback next turn (`fatprompt/runtime.nim:63-79`);
  published per-task cost comparison vs 5 rival agents (`docs/cache-analysis.md:10-25`).
- Dock: no explicit Anthropic `cache_control` breakpoints anywhere; auto-compaction is
  fixed-fraction, not projected-context measured (pi's 8 rung); skills are lazy
  (`prompts.nim:1145` "do not preload the catalog") which is table stakes.

### orchestration — 5
- Deliberate zero: "3code does not provide sub-agents ... use worktrees with cybernetic
  mode" (`docs/manual.md:951-956`). No queues, no crash-recovery journal, no budgets.
- What exists is in-loop protection and continuity: the zero-token flailing detector
  (fingerprint repetition, no-progress, Jaccard near-dup streaks; escalate ×3 then abort,
  `turns.nim:16-60`, 26 tests) and patient retry — exponential to 2048s, ~36h horizon,
  explicitly designed to outwait a 5-hour subscription window mid-turn
  (`docs/manual.md:588-613`, `api.nim:289-297,394-397`), plus per-directory session
  resume. Above codel's 4 (queue-only), well below crush's 7.

### interop — 4
- No MCP client protocol, no ACP, no IDE surface — stated philosophy: "deliberately no
  MCP-style plugin protocol: a skill file plus command-line tools covers the same ground"
  (`docs/manual.md:928-931`).
- Uses hosted MCP endpoints as a plain JSON-RPC consumer for web search
  (`web.nim:9-11,238-245` — exa, parallel) — consuming, not integrating.
- Real embedding path in-language: library API with events-as-data (`library.nim:1-33`),
  oneshot + `-r` scripting (`docs/manual.md:1268-1296`) but with no machine-readable
  output format — self-acknowledged: "A parseable-output flag is planned" (`:1295-1296`).
  Above codel's 2, below nanocoder/crush 7.

### operability — 7
- Sessions: per-cwd, `--resume` latest / `--list` / `:sessions` (`session.nim:110,141`),
  byte-exact resume incl. prompt draft restoration (`session.nim:222`), identity stamp
  re-check against profile on resume (`session.nim:22`).
- Robustness surface: network-quiet stall and truncated-stream detection with automatic
  retry (`docs/manual.md:613-616`), `:streaming off` fallback for broken provider SSE,
  Esc cancels a 36h wait, interrupts save the session mid-loop (`turns.nim:684-690`).
- Six platforms incl. Termux; self-update, desktop notifications
  (`docs/manual.md:853-866`); broken-stdout exit handled (`tests/test_broken_stdout_exit.nim`).
- No user-side rewind/checkpoint (dmail is model-side only), no diagnostics command set.

### originality — 8
Ideas verified in code, not marketing (the README's "chunked/cybernetic" claims check out
at `actions.nim:106` + skill file `src/threecode/skills/cybernetic-plan.md`):
- `dmail`: model-addressed-to-its-past-self conversation revert — the corpus's
  checkpoint-revert family is user/harness-side everywhere else; here the model does its
  own context surgery mid-turn, filesystem deliberately excluded (`prompts.nim:2983-2987`).
- Anti-approval kernel-fencing thesis with an explicit cost argument against both the
  judge-model and the approval-prompt architectures, plus the honest calibration
  (`docs/manual.md:1247-1260`) — the strongest written position in the corpus on why
  NOT to build approvals.
- Byte-reproducible releases as a CI gate: pinned container digest, apt frozen to a
  snapshot.ubuntu.com timestamp, sha256-pinned Nim, clean-rebuild byte-equality check,
  SOURCE_DATE_EPOCH double-pack check (`linux-amd64.yml:17-135`) — unprecedented among
  reviewed subjects for an agent binary.
- Zero-token flail detection with Jaccard near-duplicate clustering and streak-arm
  thresholds (`turns.nim:16-60`).
- Cross-platform PTY expect harness (ConPTY + openpty through one API) built for its own
  tests (`tests/tty_expect.nim:1-9`).
- Not 9: no new protocol/loop shape; economy is a stance others could copy wholesale but
  the primitives (summarize, skills, sessions) are standard.

### durability — 4
- One contributor, v0.8.0, solo vision, self-hosted release server; MIT; no SECURITY.md;
  no governance. Activity current (HEAD 2026-09-25). Six workflows and pinned toolchains
  show a serious solo maintainer — pi-like intensity, less community. Between nanocoder 4
  and pi 7, closer to 4.

### docs-dx — 8
- `docs/manual.md` ~1.37k lines in-repo: provider guide incl. free paths, sandbox setup,
  context-management, retry semantics, Scripting + Library API, and a Design decisions
  section whose claims match code (`docs/manual.md:1227-1266`); `reference.md` config
  reference; nimdoc dev docs generated from source (`threecode.nimble` devdocs task);
  honest-limitation prose ("the limitation: for now ...", `:1291-1296`).
- Docked vs pi's 9: single manual rather than a doc tree; onboarding depends on picking
  the right provider from a table.

## Census sanity

- `non_test_loc 42103` wrong: git-tracked `.nim` outside tests = 32,628; all tracked
  non-test files = 63,747 (includes committed generated docs HTML) — census is between
  the two, likely a mixed glob. `test_loc 29180` undercounts: 33,085 `.nim` under
  tests/ (missed files like `tests/frame_artifact.nim`, harness utilities, probes).
- `commits 1, shallow true`: real upstream is github.com/capocasa/3code (active, HEAD
  4 days old) — do not read commit count as anything.
- License: MIT file present and matches nimble. Provenance: original, not a fork.
- Tier T1 valid (32.6k non-test LOC).

## Band

Weighted total 65.75 → B (bottom edge, within 2 pts of the 65 C-boundary — flag for
synthesis). No calibration demotions; not a fork.
