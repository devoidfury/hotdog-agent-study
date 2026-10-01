# cursor-agent -- T1 review

Subject: /data/samples/agents/cursor-agent (civai-technologies/cursor-agent, MIT, Python)
Reviewer tier: T1. Static read-only; nothing executed.

## The dispatcher's question first: wrapper or harness?

**Real (small) harness, not a wrapper.** Zero invocations of Cursor's closed
`cursor-agent` binary anywhere in the package (grep for subprocess/exec in
`cursor_agent_tools/` hits only `rg` for search -- `tools/search_tools.py:149` --
and shell command execution -- `tools/system_tools.py:96`). The package talks
directly to Anthropic/OpenAI/Ollama SDKs (`pyproject.toml` dependencies) and
implements its own tool suite. Its only Cursor tie is (a) branding/aspiration
("replicates Cursor's coding assistant capabilities", README.md:11) and (b) an
optional model route "Cursor via caller-configured OpenAI-compatible gateway"
(`cursor_agent_tools/factory.py:166-196`), which needs the user's own
`CURSOR_API_KEY` and base_url -- it does not bundle or shell out to any Cursor
product. Implications: the Python tree IS the harness and gets scored as such;
interop with the Cursor ecosystem is effectively nil (gateway-only), and the
name collides with cursor.com's `cursor-agent` CLI -- corpus identity hazard,
see finding cursor-agent-1.

## Anchor question

Closest anchor: **codel (23.0)** -- both lack a managed agent loop and any
context plane (the core "chat" is a single API call and overflow is an
error message), though cursor-agent is meaningfully healthier: it has a
working, tested, confirmation-by-default permission layer and live CI that
codel entirely lacks.

## What the code actually is

~6.8k LOC package `cursor_agent_tools`: an abstract `BaseAgent`
(`base.py:52`) with three near-parallel provider implementations --
`claude_agent.py` (687), `openai_agent.py` (751), `ollama_agent.py` (688);
claude/openai text similarity ~0.58 (difflib over full files). `chat()` is a
**single completion plus at most one tool-exec round** by design
(`claude_agent.py:344-346`: "Multi-step continuation belongs to
run_agent_interactive (harness)"). The actual loop lives in `interact.py`
(1243 LOC), which iterates by injecting a mechanical continuation prompt
(`interact.py:710-739`) and decides termination via `is_task_complete`
(`interact.py:633-691`): an explicit `terminate_agent_process` sentinel
(good) with fallback to prose phrase-matching -- "task complete", "in
conclusion" + "all requirements" etc. (brittle).

Permissions: `permissions.py` PermissionManager, **needs-confirmation by
default** (`:246`), denylist checked first (`:196-203`), delete-file
protection survives yolo (`:205-207`). Wired into tools: `edit_file`
(`tools/file_tools.py:153-165`), `delete_file` (`:243-250`),
`run_terminal_cmd` (`tools/system_tools.py:67`). No sandbox of any kind
underneath; commands run `shell=True` (`system_tools.py:96-97`).

Context: `conversation_history` grows unbounded (`base.py:82`); no
compaction, summarization, or token accounting anywhere (grep `compact|summariz`
over the package: only unrelated hits). Overflow handled by catching
"Too many tokens" and returning an error string (`interact.py:244-246`).
`trim_context_history` (`interact.py:1040-1063`) sounds like compaction but
only trims the `user_info` display lists (last 10 tool calls, last 5 commands)
-- phantom context management.

## Per-dimension scores

| dim | score | best evidence |
|---|---|---|
| architecture | 4 | provider triplet duplicated chat() (claude/openai sim 0.58); loop fused into 1243-LOC CLI `interact.py:297+`; single-round chat defers loop to harness `claude_agent.py:344-346`; back-compat re-export shadow tree `cursor_agent_tools/agent/`; repo junk at root (`factorial.py`, `divide_function.py`, `fix_whitespace_errors.py`). Credit: clean `base.py` ToolResult normalization + host hooks (`base.py:270-355`). |
| verification | 4 | CI runs flake8+mypy+pytest across a 3-Python-version matrix with coverage (`.github/workflows/test.yml:31-45`), but `--ignore=tests/requires_api/` names a nonexistent dir and `-k` excludes file-tool and image tests (`test.yml:41-44`); most provider tests are `skipif` on live API keys (e.g. `test_claude_agent.py:68`). Real behavioral tests do exist offline: `test_tool_execution_records.py` (9 no-mock executor tests), `test_permissions.py` (6, mock-callback). Existence-plus-some-properties, thin and heavily filtered. |
| safety-enforcement | 4 | confirmation-by-default + denylist-first + delete protection is a sane policy module (`permissions.py:196-246`) and is mock-tested (`test_permissions.py`); but approval-only, nothing underneath (no sandbox, `shell=True` `system_tools.py:96`), substring allow/deny matching (`permissions.py:206-212,222-226`), and a hardcoded `"rm -rf /"` branch that exists to pass a test (`permissions.py:217-221`, comment at :219 "ensures our test passes"). |
| token-economy | 2 | no compaction at all; unbounded history `base.py:82`; overflow -> error string `interact.py:244-246` (same rung as codel's 1); docked further to... kept at 2 not 1 because trim_context_history + per-list caps exist, but they manage display, not model context (`interact.py:1040-1063`) -- present-but-misleading rung. |
| orchestration | 3 | single agent; session-wide tool-call cap with user opt-in to exceed (`interact.py:972-1020`), non-TTY stops instead of blocking on stdin (`interact.py:989-997`); no subagents, no resume, no journal, no budgets. |
| interop | 3 | importable Python library via `factory.create_agent` (`factory.py:106`), Cursor-model gateway passthrough (`factory.py:166-196`); no MCP, no ACP, no IDE surface, no headless JSON contract; zero real Cursor-product interop despite the name. |
| operability | 3 | interactive CLI modes (`run_agent_interactive`, `run_agent_chat`), logger module, `.env.example`; provider errors swallowed into chat strings (`claude_agent.py:419-460`); no sessions/resume/rewind/diagnostics. |
| originality | 3 | genuinely good glue ideas: registration-owned arg contract with declared `arg_aliases` + schema filtering + required-arg validation into structured errors (`base.py:150-290`); dual-emit ToolResult + host event hooks `on_tool_event`/`on_http_event` (`base.py:240-268`); exact-match terminate sentinel (`interact.py:646-649`). Nothing at the agent-loop level. |
| durability | 3 | 1 contributor, 1-commit shallow clone -- do NOT read as dead (head 2026-09-11, 18d old); release cadence real (version 0.1.43 in `setup.py:38`, bumpversion `pyproject.toml`, publish CI `publish.yml`), CODEOWNERS + SECURITY.md present. Solo project, no institution. |
| docs-dx | 4 | big README + examples/ tree + `docs/permissions_guide.md`; but `constraints.md` lists summarization/backoff "workarounds" that are not implemented (`constraints.md:11,17,20-21`) -- docs contradict code. |

**Weighted total: 33.5 -> band D.** Not "dead" D (project is alive and MIT-clean)
but thin-D: no managed loop, no context plane, approval-only safety, and a CI
gate that filters away most of its own suite. Between codel (23) and
nanocoder (60.5); closer to codel by shape, roughly 1/3 of the way up.

## Census sanity

- `test_loc: 1662` understated: tests/ + root test_*.py = 2,753 (`wc -l`).
- `manifest_file: null` wrong: `setup.py` present (name `cursor-agent-tools`,
  version 0.1.43) plus `pyproject.toml`.
- non_test 8,643 plausible (package 6,817 + root scripts; examples 2,671
  counted or not depends on glob).
- shallow clone: 1 commit at HEAD (release 0.1.43, 2026-09-11). Fresh; no
  activity claims made without remote.
- provenance `original`: confirmed -- no upstream fork markers, no Cursor
  binary dependency. Not a rename-clone; but brand-adjacent naming vs
  cursor.com's `cursor-agent` CLI (finding cursor-agent-1).

## Calibration notes

No rule (a) exposure (no fork). No archived cap needed (head is 18 days old).
Not within 2 pts of any band boundary. MIT license -- portable findings may
describe code-adjacent ideas, though this corpus's value here is concepts, not
code.
