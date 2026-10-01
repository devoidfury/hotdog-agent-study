# hax — T1 review

C, ~50k LOC src (43,980 .c + 6,158 .h), ~48k LOC tests, MIT, original (upstream
github.com/OleksandrChekhovskyi/hax). Minimalist terminal-native coding agent: one static binary,
deps limited to libcurl + jansson. Five tools (read/edit/write/bash/task_wait), five provider
families + mock. Interactive REPL and `hax -p` one-shot share one provider-agnostic loop.

## Anchor question

Closer to **crush (69.0)** than any other anchor: both are native single-binary agents with a
separated hook-driven loop, behavior-level tests, an unusually rigorous multi-OS CI, and honest
scope statements -- but crush sits above because it ships an enforcing permission service
(permission.go:15-29) and loop detection (loop_detection.go:11-40), while hax deliberately ships
neither (docs/philosophy.md:47-83, src/agent_loop.c has no repeat guard). nanocoder is the wrong
side of the same neighborhood; hax's loop is not UI-coupled at all.

## Census sanity

- `test_loc: 39,917` undercounts: `git ls-files 'tests/*.c'` = 117 files, 47,776 LOC, plus Python
  e2e under tests/e2e/ and shell/text fixtures. Real test LOC ~48k+.
- `non_test_loc: 45,153` plausible for src/ (.c 43,980 + partial headers). Tier T1 stands.
- `contributors: 2` true but 377/378 commits are one author (git shortlog) -- bus factor 1.
- `shallow: false` confirmed; HEAD 2026-09-23 with 29 commits since 2026-09-01: very much alive.
- Provenance `original`, MIT LICENSE file present, no fork flags. Nothing to check under rule (a).

## Dimension scores

### architecture — 8/15
The loop is a frontend-agnostic core with an injected hooks struct: `agent_loop.c:281`
(`for (int turn_n = 0; params->max_turns <= 0 || turn_n < params->max_turns; turn_n++)`) drives
provider streams through a pure turn-assembly state machine (`src/turn.h:10` "Pure state machine
that assembles one provider stream into conversation items") and hands all UI/policy decisions to
`agent_loop_hooks` (`agent_loop.c:200-222`: tool_call, checkpoint, tool_seen). Providers are vtables
(`provider->stream` call sites `agent_loop.c:72`, `compact.c:252`); transport, terminal, render,
system layers are separate directories with per-concern files. Abort/cancel semantics are modeled
explicitly (`agent_loop_absorb_abort`, `agent_loop.c:105-192`, with item-origin tags: interrupted /
skipped / refused / compact-seed). No god files: largest are `config.c` 1636, `select.c` 1504,
`session.c` 1410, `agent.c` 1380 (REPL host). Not 9: `config.c` is a central monolith and the two
frontends still reach into session internals in places (`agent.c:1348` decides auto-compaction
outside the loop, duplicated-ish at `agent_loop.c:450`).

### verification — 8/15
117 C test binaries + meson-registered Python e2e that drive the *built binary* against a
compiled-in mock provider scripted from files (`tests/meson.build:151-166`;
`tests/e2e/harness.py:68-108` sets `HAX_PROVIDER=mock`, `HAX_MOCK_SCRIPT=scripts/mock/*.txt`).
Tests assert loop *semantics*, not existence: cancel-repair matrix in `tests/test_agent_loop.c:244-384`
(cancelled tool call still gets a result; provider error drops the never-run call; thinking-only
cancel leaves no trace; retry usage accumulates), journal properties in `tests/test_session.c:389-603`
(resume appends only new items; a torn final JSONL line is repaired; the resume id is fixed at
open). Numeric edge cases tested (`tests/test_compact.c:16-32`, LONG_MAX threshold arithmetic).
CI is a sanitizer + distro matrix: ubuntu-asan (`ci.yml:33`), ubuntu-tsan (`ci.yml:36`), arm64,
Debian stable / Arch / Alpine containers, macOS with clang-tidy lint (`ci.yml:26-64`), plus
FreeBSD and OpenBSD builds in QEMU guests (`ci.yml:81-85`). This exceeds crush's `-race` 3-OS rung.
Ceiling per errata: no in-CI model evals, no fuzzing -- 8 stands, 9 unreachable.

### safety-enforcement — 3/10
No in-process enforcement of any kind, by explicit doctrine: "no per-command approval gate ...
because it runs inside the process it guards, it is not a security boundary -- isolation ... comes
from the operating system: run hax in a container or VM" (`docs/philosophy.md:74-83`). What
exists: system-prompt prohibitions ("Git: never commit, push, amend ... destructive commands",
`src/agent_core.c:51-53`), Esc pause with escalation on repeated stop
(`src/terminal/interrupt.c:304`), optional `max_turns`. The bash classifier is display-only and
says so ("it never changes execution or model-facing output", `src/tools/bash_classify.h:6-7`) --
so it is NOT a phantom control, but it is also not enforcement. Below nanocoder's 5 (which at
least ships a real opt-in jail); above codel's 2 because the honesty is exemplary and there is no
misleading surface -- but "prompt text is the policy" is still the codel policy shape.

### token-economy — 7/10
One auto-summarize tier with a structured checkpoint prompt (`compact.c:20-60`) triggered on
provider-measured context tokens at a configurable percent, boundary-tested
(`compact.c:68-87`, `tests/test_compact.c:16-52`). Cache discipline is the standout: Anthropic
`cache_control` breakpoints with 5m/1h TTL on tools and last message
(`src/providers/anthropic_body.c:186-225`), stable prompt-cache key for OpenAI-compat
(`docs/configuration.md:348-351`); the compaction request *advertises the full normal tool set so
the provider's cached request prefix survives*, rejecting rather than executing any tool calls the
weaker model emits, up to 4 attempts (`compact.c:240-266`, `TOOL_CALL_REJECTION` `compact.c:208-210`);
the aggregate image budget is enforced at *ingestion* precisely so prior requests stay byte-stable
("so the provider prefix cache survives", `agent_loop.c:227-235`). Cost visibility via model-meta
catalog incl. cache_read pricing (`docs/configuration.md:285`) and `/session`/`/usage`. Not 8: no
cache warming, no branch summarization, trigger keys on last-turn reported usage rather than the
projected outgoing request, single strategy.

### orchestration — 6/10
Distinctive background-task subsystem: bash commands that outlive their foreground window are
adopted into a process-local registry owning the process tree, a drainer thread, a spool file, and
a per-task cursor of what the model has seen (`src/tools/task_registry.h:8-30`); completion is
announced as one-line notes injected at the earliest loop seams (`agent_loop.c:292-300`);
`task_wait` streams output, kills with SIGTERM-grace-SIGKILL, and is Esc-answers
(`task_registry.h:44-58`). Beyond that: nothing in-process -- no subagents (presets "can serve as
subagent roles" is a docs convention), no queues, no loop detection, `max_turns` defaults to
unlimited (CHANGELOG Unreleased). External orchestration via `hax -p` loops is designed and
documented (`docs/philosophy.md:26-36,58-65`) with `--json` JSONL as the supported observation
surface. Crash semantics of the registry are deliberately bounded (tasks never outlive the process)
but nothing resumes a crashed run mid-turn beyond the session journal.

### interop — 5/10
No MCP, no ACP, no SDK, no IDE surface -- all deliberate (`docs/philosophy.md:39-45`). The headless
contract is genuinely good: `-p` with clean stdout and hints on stderr, `--json` emitting the same
versioned JSONL records as session files (`src/cli.c:36,171`; `docs/sessions.md:1-10` "versioned
(`\"version\": 1`) ... supported read surface for scripts and orchestrators"), `--resume=ID`. That
is one-shot level only: no interactive RPC plane (pi's 6 rung has json/rpc modes), so 5.

### operability — 7/10
Append-only, versioned JSONL session journals flushed per line (`tail -f`-able), owner-only files,
pruned by retention config, `/undo` and `/fork` appended not truncating, fork lineage recorded
(`docs/sessions.md:15-30,57`); resume repairs torn final lines with a test (`tests/test_session.c:505-529`);
session picker + per-cwd prompt history (recent commits 5ed64f8, bf1dfb4 share the resume binding
between frontends). Diagnostics: Ctrl+T transcript view of exactly what is sent, `HAX_TRACE`
redacted wire trace (`docs/debugging.md`), `/session` token/cost totals restored on resume. Docked:
no file-level checkpoint/revert -- `/undo` is conversation-level, edits to disk are not rewound
(vs cline/crush 8 rung); no SECURITY.md.

### originality — 7/10
Verified in code, not marketing: (1) compaction that preserves the prompt cache by advertising the
live tool set and answering tool calls with rejections instead of executing them
(`compact.c:240-266`); (2) image-budget-at-ingestion chosen specifically for request byte-stability
(`agent_loop.c:227-235`); (3) background-task adoption with per-task delivery cursors
(`task_registry.h`); (4) the anti-feature doctrine with a written rationale per omitted feature,
including the *honest* gap statement ("What outside scripts cannot express is in-turn interception
... hax omits that deliberately rather than claiming a substitute", `docs/philosophy.md:47-72`);
(5) distro-grade dependency bar: linked deps must be in Debian main, one brew install away, and
not GPL (`docs/philosophy.md:104-113`). Nothing here is field-defining, but several mechanisms are
uniquely this subject's.

### durability — 4/10
Bus factor 1 (377 of 378 commits, git shortlog; census "2 contributors" is technically true).
History ~5 months (first commits 2026-04-24) but dense and alive: 378 commits, 29 in September
alone, HEAD 6 days before review; 5 semver tags with releases 2026-08-07 .. 2026-09-04 plus
Homebrew tap and AUR packaging (README:47-53); CHANGELOG kept as release-notes source. No
SECURITY.md, no institution, no funding signal. Matches nanocoder's 4 more than crush's 8; the
release/packaging cadence is what keeps it from 3.

### docs-dx — 8/10
Nine in-repo docs that match the code: philosophy (rationale per omission), configuration (full
key tables incl. env vars and cache TTL knobs), sessions (format reference), providers, usage,
debugging, releasing, code-style. README install path covers Homebrew, AUR, static binary, source
across Linux/macOS/FreeBSD/OpenBSD, with a dependency script per distro. AGENTS.md/CLAUDE.md dogfood
the tool's own conventions. Docked for no SECURITY.md and no docs index. Above crush's 6 (config+hooks
only), at codex/nanocoder's 7-plus; a near-miss for pi's 9 on volume alone.

## Totals

| dim | score | weight | pts |
|---|---|---|---|
| architecture | 8 | 15 | 120 |
| verification | 8 | 15 | 120 |
| safety-enforcement | 3 | 10 | 30 |
| token-economy | 7 | 10 | 70 |
| orchestration | 6 | 10 | 60 |
| interop | 5 | 10 | 50 |
| operability | 7 | 10 | 70 |
| originality | 7 | 10 | 70 |
| durability | 4 | 5 | 20 |
| docs-dx | 8 | 5 | 40 |
| **total** | | | **65.0** |

Band **B** (65-77). Strongest dimension: docs-dx (relative to corpus norm; verification is the
best in absolute terms). Weakest: safety-enforcement (3) -- prompt-only restraint, honestly labeled.

**Boundary risk**: 65.0 lands exactly on the B/C boundary; safety-enforcement (3) vs interop (5)
is the swing. A reviewer who scores hax's honesty-and-OS-delegation as high as nanocoder's
opt-in jail (5) puts it at 67; one who treats "no mechanism at all" as codel-adjacent-but-honest
(2) puts it at 64/C. The 3 placement is deliberate: mechanisms exist only as social prompts, but
unlike codel there is zero misleading surface.

Calibration: no rule (a) (not a fork); no archived cap; verification held at 8 per errata (no
evals-in-CI/fuzzing despite sanitizer rigor).
