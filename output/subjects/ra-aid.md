# ra-aid (RA.Aid) -- T1 review

Subject: /data/samples/agents/ra-aid @ e71bb83 (shallow, HEAD 2025-06-16)
Manifest: pyproject.toml `ra-aid`, Python >=3.10, LangChain/LangGraph-based autonomous coding agent.
License: Apache-2.0 (file). Provenance: original (census flag agrees; no fork markers, upstream remote = origin).

## Anchor question (one sentence)
Closer to **nanocoder (60.5, C)** than to crush: both are solo-led agents with genuinely
distinctive machinery whose score is dragged by wrong safety defaults and thin enforcement --
except ra-aid's bypass channel is the loop itself (model output is `eval`'d), which puts it a
notch below nanocoder's "real jail, opt-in" shape.

## Sanity checks
- **Shallow clone**: `.git/shallow` present, 1 local commit. Remote-verified:
  `pushed_at 2026-01-30T16:24:40Z`, `archived:false`, 61 open issues, 2220 stars
  (api.github.com/repos/ai-christianson/RA.Aid). Last push ~8 months before review date:
  quiet, **not dead (>12mo), not archived => rule (b) does NOT apply.**
- **Census corrections**:
  - `contributors: 1` is wrong -- remote contributors API shows ~10+
    (ai-christianson 818, ariel-frischer 254, willbonde 77, pztrick 41, leonj1 18, ...).
  - `test_loc: 12542` undercounts: `tests/` = 19,143 py LOC + 1,045 LOC nested tests inside
    `ra_aid/` => ~20.2k.
  - `non_test_loc: 69433` inflated ~2x: `ra_aid/` non-test py = ~28.7k, frontend TS/TSX =
    6.5k => ~35-36k realistic. T1 tier stands either way.
  - `head_date 2025-06-16` understates activity (see pushed_at above).

## Core loop (read line-level)
`ra_aid/agent_backends/ciayn_agent.py` (1131 LOC) -- "CIAYN" (Code Is All You Need): the LLM
emits Python expressions which are AST-validated and `eval`'d against a globals dict of tool
functions:
- Loop: `stream()` at ciayn_agent.py:929 -- `while True` with should_exit checks (:936,951,983),
  model.invoke (:953), empty-response ladder (3 strikes then `mark_agent_crashed`, :996-1013),
  tool exec + fallback (:1090-1111).
- Execution: `globals_dict = {tool.func.__name__: tool.func ...}` at :345, then
  `eval(call.strip(), globals_dict)` at :533 and `eval(code.strip(), globals_dict)` at :691.
  **No `__builtins__: {}` and no AST allowlist** -- validation
  (`validate_function_call_pattern`, :45-90) only requires a single `ast.Call` expression, so
  `__import__('os').system("...")` passes validation and executes, bypassing the shell approval
  prompt entirely.
- Repeat-call breaker: fingerprint `(tool_name, sorted(param_pairs))` compared against the
  single previous call only (:505-528 bundled path, :583-660 single path -- ~120 LOC duplicated
  between the two), for a hardcoded `NO_REPEAT_TOOLS` list (:137-150). Two alternating calls
  defeat it; AST parse failures are silently swallowed (:531-537 "just continue").
- Compaction: `_trim_chat_history` (:872-904) = count cap + **head-drop by token estimate**,
  no summarization. Estimator is `len(text.encode("utf-8")) // 2.0` (:910-927). No
  prompt-cache discipline anywhere (grep: no cache_control/cache usage).
- Class docstring claims "Sandboxed code execution environment" (:122) -- there is no sandbox
  anywhere in the tree (grep sandbox|bwrap|seatbelt|landlock hits only that docstring line).

## Compaction / context (second read)
Per-model quirk table `ra_aid/models_params.py` (1421 LOC) holds token limits and
`attempt_llm_tool_extraction` (:483-495 model-scoped LLM tool-call rescue);
`ra_aid/anthropic_token_limiter.py` uses litellm `token_counter` (:20) for anthropic-family
paths, so real counting exists at the provider edge while the in-loop trim uses bytes/2.
Cost discipline is above-average: Decimal cost tables + `--max-cost` with interactive
continue/stop decision (`default_callback_handler.py:454-497`), `--exit-at-limit`,
usage scripts (`ra_aid/scripts/last_session_usage.py`, `all_sessions_usage.py`).

## Permission / safety code
- Shell: `ra_aid/tools/shell.py:89-109` -- per-command y/n/c prompt, **default "y"**, "c"
  flips `cowboy_mode` for the session from inside the prompt (:104-105). Approval is genuinely
  tested: `tests/ra_aid/tools/test_shell.py:96-110` (cowboy => `Prompt.ask.assert_not_called`)
  and :125-140 (approval iff exact prompt text).
- File writes: no approval at all (`grep confirm|Prompt|approv` in tools/write_file.py,
  tools/file_str_replace.py: zero hits).
- Server: binds **0.0.0.0 by default** (`__main__.py:489-490`) with an unauthenticated
  `/v1/spawn_agent` endpoint (`server/api_v1_spawn_agent.py:306`); cowboy+server combination
  warns but runs (`__main__.py:1179-1181`). No SECURITY.md.
- Net: tested approval exists (rare, good) but it is a sieve -- eval-bypass + no write approval
  + open server default. Below the "6 = tested approval, nothing underneath" rung because what's
  underneath is actively negative, and the docstring misrepresents it.

## Verification
- ~20.2k test LOC, 692 `def test` in tests/, CI on push/PR ubuntu/py3.12
  (`.github/workflows/tests.yml:36-38`) running `make test` (pytest + coverage report).
- Tests assert behavior not existence: trim semantics (test_ciayn_agent.py:52-180), shell
  approval iff-policy (test_shell.py), crash propagation (tests/ra_aid/test_crash_propagation.py),
  fallback model switching (test_fallback_handler.py), token-usage tracking.
- Gaps: single OS/py-version matrix; coverage gate commented out in `Makefile`
  ("for future consideration append --cov-fail-under=80"); mock-heavy (console/prompt/run mocks);
  swebench dataset generator in-repo (`scripts/generate_swebench_dataset.py`) but no CI workflow
  references evals (grep .github: zero hits) => eval-harness-outside-ci.

## Orchestration
Research -> plan -> implement pipeline (`agents/research_agent.py`, `planning_agent.py`,
`implementation_agent.py`) driven as nested agent-tools (`tools/agent.py:55,301,391,562`);
expert mode (`tools/expert.py:57,158`); **memory GC agents** (`agents/research_notes_gc_agent.py`
386 LOC, `key_facts_gc_agent.py`, `key_snippets_gc_agent.py`) that garbage-collect accumulated
notes/facts -- distinctive, real in code. Cancellation via `should_exit(session_id)` polled in
loop + thread registry (`utils/agent_thread_manager.py:12-68`). Session/trajectory/human-input
state in per-project sqlite (peewee repositories, `database/repositories/`). No durable queue,
no resume/fork journal, no turn budgets.

## Interop
MCP client via `MultiServerMCPClient_Sync` sync-wrapper (`utils/mcp_client.py:8-35`), but wired
only through a user-written `--custom-tools` python file exporting an `mcpServers` config
(`examples/custom-tools-mcp/README.md`) -- no native config file. FastAPI server + generated
OpenAPI (`docs/ra-aid.openapi.yml`, `scripts/generate_openapi.py`) + web UI
(`frontend/web`) + thin VS Code shell (`frontend/vsc/src/extension.ts`, 132 LOC). No ACP, no
MCP server mode, no published SDK.

## Originality
- CIAYN code-as-actions via eval (distinct shape; the flip side is the safety hole above).
- Leaderboard-frozen fallback chain: on tool-call failure, falls back to models picked from a
  hardcoded snapshot of the Berkeley gorilla function-calling leaderboard
  (`tool_leaderboard.py:1-5` "Data extracted at 2/10/2025", `fallback_handler.py:43-49`),
  experimental flag-gated (:42).
- Memory GC agents; models_params quirk table encoding per-model tool-calling failure modes.
- Persistent project memory (key facts/snippets/related files, `tools/memory.py:230-505`)
  survives the head-drop trim because it lives in the DB.

## Durability / docs
Solo-led (top author 818 of ~1250 contributions), no SECURITY.md, dependabot present
(`.github/dependabot.yml`), release workflow publishes PyPI + GitHub releases with tests as a
release gate (release.yml:39-40). 27-page docusaurus tree incl. usage walkthroughs and generated
API reference; decent CHANGELOG.

## Scores (against anchor ladder)
| dim | score | best evidence |
|---|---|---|
| architecture | 5 | CiaynAgent god-class ciayn_agent.py:100-1131 with ~120 LOC duplicated repeat-check blocks (:505-528 vs :583-660); __main__.py 1658 LOC; good repo/tool separation (database/repositories/) |
| verification | 6 | 692 behavior-asserting tests + CI (tests.yml:36-38, test_shell.py:96-140); no evals/fuzz in CI, cov-gate commented (Makefile) |
| safety-enforcement | 3 | tested shell approval (shell.py:89-109) negated by eval-bypass (ciayn_agent.py:533,691 + validate :45-90) + unauth 0.0.0.0 server (__main__.py:489) |
| token-economy | 4 | head-drop trim bytes/2 estimator (ciayn_agent.py:872-927); models_params per-model limits; --max-cost gate (default_callback_handler.py:454-497); no summarization, no cache discipline |
| orchestration | 5 | nested agent-tools (tools/agent.py:391,562), GC agents, should_exit cancel (:936-941); no queue/resume/budget |
| interop | 5 | MCP client via custom-tools glue (utils/mcp_client.py:8), FastAPI+OpenAPI+web UI; no ACP/MCP-server/SDK |
| operability | 5 | sqlite session/trajectory API (api_v1_sessions.py:98-298), usage scripts, persistent config; no resume/fork/checkpoint, no crash-recovery posture |
| originality | 6 | CIAYN eval loop, leaderboard-frozen fallback (tool_leaderboard.py:1-5), memory GC agents |
| durability | 4 | solo-led, ~10 contributors, not archived, last push 2026-01-30; no SECURITY.md |
| docs-dx | 6 | 27-page docusaurus incl. usage walkthroughs + generated API docs; release-gated tests |

Weighted total: (5+6)*1.5 + (3+4+5+5+5+6)*1 + (4+6)*0.5 = 16.5+28+5 = **49.5 => C**

Strongest dimension: **verification** (highest-weight dimension at 6, real property-asserting
tests of the approval gate and trim semantics).
Weakest dimension: **safety-enforcement** (3; approval gates that exist are bypassable by the
loop's own execution model, and the codebase claims otherwise).

## Calibration notes
- Rule (b) NOT applied: remote-verified `pushed_at 2026-01-30`, `archived:false`; shallow HEAD
  understates activity by 7.5 months. No demotion recorded.
- Census row wrong on contributors (1 -> ~10+), test_loc (12.5k -> ~20.2k), non_test_loc
  (69.4k -> ~35k realistic). Tier T1 unaffected.
- Provenance: original, Apache-2.0 => normal copying rules apply, though the eval-loop shape is
  NOT something to port.
