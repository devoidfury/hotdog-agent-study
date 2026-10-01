# hax - boundary re-review (T1, C, MIT, solo)

Provisional 65.0, exactly on the B floor. Contested axes: safety-enforcement 3 (prompt-only
restraint, honestly labeled - codel-2 or above?) and verification (does the ASan/TSan/BSD matrix
plus mock-provider e2e actually BLOCK?). Verdict: **66.0 B CONFIRMED; B floor holds.**

Independence note: subjects/hax.md, scores/hax.json, findings/hax.jsonl NOT read. Evidence gathered
directly; findings ids start at hax-b1 and are new-only. HEAD 6753f30 (2026-09-23), full clone (no
`.git/shallow`), 378 commits, 377 by one author.

Closest anchor: **pi** - solo-vision minimalism, hook-separated loop, documented-by-philosophy
no-MCP stance, exemplary posture honesty. But on safety hax sits three rungs below pi (pi has an
enforced blocking hook + project-trust gate = 6; hax has nothing binding = 3), and its totals
track closer to crush's shape. Place hax as "pi minus enforcement, plus sanitizer theater made real."

## 1. Safety 3 - CONFIRMED, and the honesty ruling (doctrine-critical)

Mechanics, verified absent:
- No approval gate, by design and by grep: `docs/philosophy.md:74-84` ("No permission prompts...
  it is not a security boundary"); the only tool_call hook in either frontend renders, never
  prompts (`src/agent.c:992-1006` dispatch/refuse/skip branches, no user-prompt branch in REPL or
  oneshot).
- Zero enforcement anywhere in src: grep bwrap/seatbelt/landlock/pledge/sandbox = 0 hits; no
  deny-list gating (`src/tools/bash_classify.c` classifies exploration-vs-writer for UI purposes,
  not for blocking); no path containment in edit/write (`src/tools/path_preprocess.c` only
  relativizes for display).
- The "restraint" is prompt text: destructive-git prohibition + ask-before-irreversible
  (`src/agent_core.c:51-53`). User-side controls are Esc/SIGUSR1 pause and `max_turns`, which
  defaults to 0 = unbounded (`src/config.c:93`).

Honesty, verified exemplary:
- README:32-34 tells users wanting permission prompts to go elsewhere; `docs/philosophy.md:83`
  states "control and restraint, not enforcement" and redirects blast-radius containment to the OS
  (container/VM); `docs/configuration.md:194-196` warns that a full `system_prompt` override drops
  the built-in safety guidance - the docs even defend the restraint layer they admit is soft.

**Ruling (unifies with kolkrabbi):** the honesty-credit asymmetry is one rung, one direction.
Codel's 2 is mechanics-identical to hax's - policy IS prompt text - but codel earned the bottom of
rung 2 by deception: README "fully autonomous" contradicting behavior plus an unauthenticated
container-spawning API. hax's disclosure converts an unearned-surprise default into an informed
one, which lifts it out of the misleading band to 3, the ceiling for absent mechanics. It cannot
go higher: nothing binds, so the shipped default - an unrestricted shell with a polite request -
scores as shipped, per the octomind/forge doctrine (shipped-default posture, full stop). Honesty
never substitutes for enforcement; enforcement always outranks honesty (kolkrabbi's 7 stands on
its tested floor and fail-closed jail, not on its SECURITY.md). Confirms the interim doctrine at
calibration-notes.md:698-700 verbatim: honesty mitigates DECEPTION penalties, cannot lift absent
mechanics above 3. Both rulings now on record; corpus-wide rule: **rung 2 = policy-as-prompt AND
misleading; rung 3 = policy-as-prompt honestly labeled; rung 4+ requires binding machinery, honest
or not.**

## 2. Verification 8 - CONFIRMED (the matrix BLOCKS)

- `.github/workflows/ci.yml` runs on every `push` and `pull_request` (:3-5). The `test` job matrix
  includes `ubuntu-asan` and `ubuntu-tsan` entries (:33-38), and `make tests` maps those build dirs
  to `-Db_sanitize=address,undefined` and `-Db_sanitize=thread` respectively
  (`scripts/check.sh:100-107`, `scripts/check.sh:157-163` runs `meson test` for every matrix dir,
  `Makefile:11-12`). A sanitizer report fails the job on the commit - not nightly, not advisory.
- BSD QEMU jobs (`ci.yml:74-86`, FreeBSD 15.1 + OpenBSD 7.9 via cross-platform-actions) run the
  same `make tests` on every push. Same-suite ports, not smoke builds.
- e2e of the real binary: `tests/meson.build:151-166` registers Python scenarios that drive the
  built binary through the mock provider (`tests/e2e/harness.py`, scripted transcripts in
  `scripts/mock/`), inside the same `meson test` run, so the e2e suite runs under ASan and TSan too.
- Test corps asserts properties, not existence: 47.8k test LOC vs ~44.0k product LOC; loop tests
  named as invariants (`tests/test_agent_loop.c` - `test_cancel_keeps_sealed_reasoning_with_tool_call:361`,
  `test_loop_pause_preempts_follow_up_leaves_no_boundary:905`, provider-error/abort repair
  semantics at :224-334); provider wire/body/event tests across all five adapters.
- Why not 9: zero fuzz targets (grep LLVMFuzzer/fuzz = 0 hits across src/tests/scripts/.github),
  no in-CI model evals. The errata's 8 ceiling (codex/cline/pi) applies; hax sits at its floor-to-mid.
  The 8-vs-7 case: nanocoder-7 is "tests assert behavior but TUI/loop paths thin" - hax's loop is
  the thickest-tested path in the repo, and two sanitizer builds gating every push exceeds crush's
  `-race` 3-OS on a C codebase.

## 3. Other lanes (independent, brief)

- **Architecture 8.** Engine loop is 481 LOC of hook-driven machinery (`src/agent_loop.c`) with the
  REPL supplying render/checkpoint/tool hooks (`src/agent.c:1303-1304`); session item model with
  origin stamps (`ITEM_ORIGIN_INTERRUPTED/SKIPPED/REFUSED`); layers for provider/transport/tools/
  system/terminal cleanly separated (no cross-layer includes observed at read depth). Largest file
  1636 (config.c), no >5k files, so the errata docking is moot. Docked from 9: no CI-enforced
  architecture gate (contrast kolkrabbi), and config.c/select.c/input.c carry real weight.
- **Token-economy 7.** Structured-checkpoint compaction (`src/compact.c:19-56`), auto trigger at
  configured %-of-window with careful rounding (`compact.c:70-86`), tools kept advertised during
  compaction to preserve the cached prefix (`compact.c:241`), image budget enforced at ingestion to
  keep prior requests byte-stable (`agent_loop.c:224-230`), `prompt_cache_key` for chat-style APIs
  (`src/providers/chat_body.c:345-346`) and Anthropic TTL marks (`anthropic_body.c:186-227`), exact-vs-estimated
  cost footers (`src/agent_usage.h:50-59`). Short of 8: no cache warming, no branch summarization,
  trigger is fraction-of-limit rather than measured projected request.
- **Orchestration 6.** Managed background tasks with caps and wait/collect/stop (`src/tools/task_registry.c`,
  `task_wait.c`), subagents as `hax -p` subprocesses with depth env guard (`src/tools/bash_env.c:84`)
  and preset-advertised roles (`docs/usage.md:240-243`), `max_turns` bound
  (`agent_loop.c:281`). No queue/daemon/journal; tasks die with the conversation (documented,
  usage.md:236-237); no loop detection.
- **Interop 6.** Documented `--json` stream contract with resume handle, outcome records, and exit
  codes (`docs/sessions.md:96-118`), pipe-safe stdout (usage.md:49-53), PATH composition. MCP/ACP
  absent by declared philosophy (README:32-34). Pi's exact rung and reasoning.
- **Operability 7.** `--resume` + session picker, persistent Ctrl-R history, append-only 0600
  session logs with O_NOFOLLOW reads and a header-preservation rule so a deleted session is not
  resurrected mid-write (`src/session.c:420,538-554`), trace/transcript diagnostics
  (`docs/debugging.md`), env-var-mapped validated config. No crash-recovery middleware, no session
  lock; a segfault is a death, mitigated rather than handled (TSan/ASan in CI).
- **Originality 7.** The corpus's only distro-grade C agent, with first-class BSDs as a tested CI
  tier (`ci.yml:81-86`) rather than a README claim; the byte-stable image-budget ingestion rule;
  mock-provider + wire-trace dogfooding (`scripts/mock/`, `src/trace.c`). Distinctive shape backed
  by real machinery; short of 8 because no single mechanism others should copy defines a category.
- **Durability 5.** Solo (377/378 commits) but active at HEAD-6-days; MIT; Homebrew + AUR
  (README:45-51) + static-binary release workflow gated on tag/version match (`release.yml:25-26`).
  No SECURITY.md, no governance above a `releasing.md` runbook. One notch above nanocoder-4 on
  release discipline and third-party distribution; nowhere near pi-7.
- **Docs-dx 7.** Eight docs whose claims I verified against code (every omission asserted in
  philosophy.md is genuinely absent); install_deps.sh across 8 platforms; honest README target
  section. Docked for no SECURITY.md.

## Weighted

8x15 + 8x15 + 3x10 + 7x10 + 6x10 + 6x10 + 7x10 + 7x10 + 5x5 + 7x5 = 660 -> **66.0, B.**
B floor HELD (provisional 65.0 -> independent 66.0; the pick was verification 8 - the blocking
sanitizer+BSD+e2e matrix is real machinery on every push - offset by durability 5 vs a likely
provisional 4-5). No band movement, no calibration demotions.
