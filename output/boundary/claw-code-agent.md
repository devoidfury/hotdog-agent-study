# Boundary re-review: claw-code-agent

Provisional: 44.0 (D); triage flagged verification 5->6 as the swing to the C floor (45).
Independent boundary scoring: **50.0 (C)**. Tier T1 (src 50,366 Python LOC + 18,315 test LOC, cloc).
Read-only static review; repo never executed. `.git/shallow` present (local rev count = 1) so
durability was remote-verified via GitHub API.

**Closest anchor:** nanocoder (60.5, C) -- both have behavior-testing faux-backend test corps and
regex-denylist safety with no sandbox floor; claw sits below nanocoder on verification (zero CI),
originality (explicit parity port, not homegrown shape) and durability (solo, no license), which
caps it low-C rather than mid-C.

## Provenance: suspected port from leaked/de-obfuscated Claude Code snapshot -- license-risk

This is not merely "inspired by" Claude Code; it is a module-path-level parity port of the *unreleased
TypeScript internals* of the npm package:

- `src/bash_security.py:4` -- "Ported from npm src/tools/BashTool/bashSecurity.ts and related files."
- `src/microcompact.py:3` -- "Mirrors the npm ``src/services/compact/microCompact.ts`` module."
- `src/compact.py:3,60` -- mirrors `services/compact/compact.ts` and `services/compact/prompt.ts`.
- `src/session_memory_compact.py:4`, `src/builtin_agents.py:3`, `src/bundled_skills.py:3`,
  `src/prompt_constants.py:1`, `src/paste_refs.py:3`, `src/agent_runtime.py:253`
  ("Mirror commands/clear/caches.ts"), `src/sandbox_types.py:1` ("Python port of
  entrypoints/sandboxTypes.ts"). 32 such in-code declarations total (grep "npm" over src).
- `PARITY_CHECKLIST.md:1-3` -- a whole-repo checklist "Against npm `src`", tracking internal-file
  parity. `src/port_manifest.py` and `src/parity_audit.py` are dedicated parity tooling.

Claude Code's public repo is docs/issues only; the npm distribution is bundled JS. Verbatim internal
module paths (`tools/BashTool/bashSecurity.ts`, `services/compact/microCompact.ts`) plus internal
constant lists (e.g. `ZSH_DANGEROUS_COMMANDS`, `src/bash_security.py:74-80`) could only come from a
source-map/demangled or leaked snapshot. Compounding: **no LICENSE file anywhere** (GitHub API
`license: null`) while `README.md:19` badges "license-open-source" -- misleading on its face.

Per protocol: this subject gets `license-risk` findings and **no code-adjacent portables**. Any
concepts harvested from it must be pure-concept (its faux-provider test design and cache-gap
microcompaction are generic patterns also present in other subjects, so they may converge on the
concept level only, never by copying this repo's code).

## Verification: 5, not 6

The ~1,233 tests (`def test_` across 82 files, 18,315 LOC) are genuinely **property-proving, not
existence-only**. Sampled proof points, all asserting loop semantics against a scripted fake OpenAI
SSE backend (`tests/test_agent_runtime.py:31-59` fake urlopen):

- compact-boundary insertion at threshold: `tests/test_agent_runtime.py:623`
- total token budget stop (`stop_reason='budget_exceeded'`): `:704`; pre-flight reject before
  backend: `:740`; auto-compact before next call: `:789`
- duplicate tool-call-id not executed twice: `:2842`
- hook policy blocks tool + records permission denial: `:2745`
- nested delegation, dependency batches, child-session resume lineage: `:1629, :2448, :2529, :1909`

Why still 5 and not 6:

1. **No CI exists at all** -- `.github/workflows` absent; nothing runs these 1,233 tests
   automatically, ever. Every anchor rung >= 6 has automation evidence (crush 7: `-race` 3-OS
   matrix; nanocoder 7: coverage-drop CI gate `pr-checks.yml:17-21`). Protocol evidence rule: no CI,
   no enforcement claim. Benchmarks (19 suites under `benchmarks/suites/`) also have no runner.
2. **Inflated corpus**: 163 of the 1,233 tests (`tests/test_bash_security.py`, 878 LOC) exercise
   `src/bash_security.py`, which is imported by nothing in `src/` (grep across the tree: zero
   references outside itself and tests). Testing dead code inflates the count without adding
   enforced assurance -- the actual bash gate is 26 lines of regex (`src/agent_tools.py:1320-1345`).
3. TUI/GUI/loop-adjacent surfaces thinner than runtime core, similar to nanocoder's caveat.

## Safety-enforcement: 4

Real machinery, thin and bypassable, no isolation, plus a misleading dead-security artifact:

- Actual gate: `_ensure_shell_allowed` (`src/agent_tools.py:1320-1345`) -- boolean flags
  (`allow_shell_commands`, `allow_destructive_shell_commands`) + an 11-pattern destructive regex
  deny-list. Trivially bypassed (newlines, `command rm`, backticks, base64 piped to sh). Commands
  then run `subprocess.run(command, shell=True, executable='/bin/bash')` (`src/agent_tools.py:1778-1786`)
  and `Popen` streaming variant (`:3133-3142`).
- **Zero sandboxing**: grep `seatbelt|bwrap|landlock|firejail|unshare` over `src/` = 0 hits.
  `src/sandbox_types.py` is config-type parsing only (a port of the upstream sandbox *settings
  schema*), not enforcement.
- `src/permissions.py` is 20 lines (deny names/prefixes on tool names). No interactive approval
  flow exists at all -- the model is non-interactive with `--unsafe` escape (`src/main.py:80-81`).
- Real positives, tested: secret-bearing-path refusal on read/edit (`src/agent_tools.py:1370-1399`,
  `tests/test_agent_tools_secret_path_guard.py`), sensitive-env filtering into subprocesses
  (`_build_subprocess_env`, `:3549`; `tests/test_agent_tools_security.py`), hook-policy tool
  blocking with denial accounting (`src/hook_policy.py`, `tests/test_agent_runtime.py:2745`).
  Hooks never exec external commands (no subprocess in `hook_policy.py`/plugin runtime) -- the
  "trusted" flag is soft reporting (`src/agent_runtime.py:3163-3172`).
- The 1,261-LOC ported `bash_security.py` with 163 passing-looking tests, while unwired, at least
  doesn't claim to bind; docs make no sandbox claims (unlike codel's misleading "fully autonomous").
  That keeps this at 4 (nanocoder-5 minus: no jail machinery whatsoever) rather than codel-2.

## Architecture: 5

Single clean turn loop (`for turn_index in range(1, max_turns+1)`, `src/agent_runtime.py:528`) with
pre-turn microcompact/snip/compact/preflight cascade (`:529-542`) -- no parallel re-implementations
(above nanocoder's 5). But `LocalCodingAgent` in `src/agent_runtime.py` is 4,431 LOC fusing loop,
delegation, budget, compaction orchestration, and report rendering; `src/agent_tools.py` 3,587 LOC;
`src/main.py` 1,659. Worse fusion than crush's 7 (agent.go 2393 + coordinator.go 1877). Peripheral
module decomposition (mcp_runtime, hook_policy, compact, session_store, agent_manager, ~40
single-purpose runtimes) is good. `PARITY_CHECKLIST.md:3` itself admits large parts of the mirrored
tree "still act as inventory or scaffolding."

## Token-economy: 6

Genuine tiering, mirrors Claude Code's design and is tested: time-based microcompact that clears old
tool results only when the gap since last assistant message exceeds ~60 min, i.e. provider prompt
cache has expired (`src/microcompact.py:1-30`, `src/agent_runtime.py:1530`); snipping with tombstone
mutation history (`tests/test_agent_runtime.py:1268`); LLM compaction with reserved-window
(`src/compact.py:35`) and reactive prompt-too-long retry that drops groups by measured token gap
(`truncate_head_for_ptl_retry`, `src/compact.py:322-354`); prompt-length preflight before backend
calls (`:740` test); cached tokenizer backend with heuristic fallback. Cost/usage tracking with
cache_read/cache_creation token accounting (`src/openai_compat.py:108-111`). No cache-control
breakpoint authoring (OpenAI-compat surface). One notch above crush's single-strategy 6? No -- the
tiers mirror upstream rather than extending it; held at 6.

## Orchestration: 6

Nested delegation with dependency-aware topological batching (`src/agent_runtime.py:2400, 2734`,
tests `:2448, :2529`), agent-manager lineage (`src/agent_manager.py`), child-session resume
(`:1909` test), delegated-task budgets (`:2138`). Workflow/task/team runtimes are manifest-backed
local stores. No queue, no crash-recovery journal, no loop detection. Nanocoder-6 rung.

## Interop: 6

Real stdio MCP client transport -- initialize, resource list/read, tool list/call (`src/mcp_runtime.py`,
880 LOC, 350 LOC tests). Imports rival-harness config surfaces: `~/.claude/CLAUDE.md`,
`.claude/CLAUDE.md`, `.claude/rules/`, `.claude/agents`, `.claude/plugins/cache.json`
(`src/agent_context.py:351-365`, `src/account_runtime.py:15-16`, `src/agent_plugin_cache.py:78`).
Structured `--response-schema-file` headless mode (`src/main.py:95-97`). Local web GUI
(`python -m src.gui`). No MCP server, no ACP, no published SDK.

## Operability: 6

Session save/resume (tested `:361`), file-history journaling with snapshot ids and resume-time
replay reminders (`:1007, :1331, :1426`), worktree runtime with mid-session cwd switching,
config/account/ask-user/team runtimes, query-engine reports. No crash-recovery posture, no
fork/tree navigation.

## Originality: 3

Everything architectural mirrors upstream npm internals by declaration (32 markers). The genuine
deltas are meta-level: zero-dependency stdlib-only constraint, the faux-backend behavioral test
corps, 19 benchmark suites, parity audit tooling (`src/parity_audit.py`, `src/port_manifest.py`).
Real, in-code, but no agent-architecture idea another subject lacks. Distinctive shape without
mechanism scored 5 (codel); claw has mechanism but not its own shape. 3.

## Durability: 3

Shallow clone hides history (local rev count 1) -- remote-verified instead: created 2026-04-01,
pushed_at 2026-06-22 (HEAD matches remote), so ~3 months quiet, not archived. Single author
(Abdelrahman Abdallah), org "HarnessLab" effectively one-person. 546 stars / 224 forks in ~6 months
is real interest. No LICENSE, no SECURITY.md, no CI/release infra.

## Docs-dx: 5

README thorough, TESTING_GUIDE.md is a per-surface manual command checklist (good for humans,
nothing automated), PARITY_CHECKLIST.md unusually honest about scaffolding status. Zero-dependency
install is trivially simple. No SECURITY.md, no architecture docs.

## Weighted total

arch 5*1.5=7.5 | verif 5*1.5=7.5 | safety 4 | token 6 | orch 6 | interop 6 | oper 6 | orig 3 |
dur 3*0.5=1.5 | docs 5*0.5=2.5 => **50.0, band C**.

Divergence from provisional 44.0: the provisional lands below every non-codel anchor with real
tests and real safety wiring; claw enforces more than codel-2 safety and tests behavior like a C
subject. The C placement rests on verified artifacts (loop, cascade, delegation tests, secret
guards), not goodwill. The license-risk does not band-cap per protocol but makes this a
quarantine-candidate: do not port code, and treat even concept harvesting cautiously since the
upstream design provenance is Anthropic proprietary.
