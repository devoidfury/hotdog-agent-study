# claw-code-agent -- differential triage (T0): census "leaked-snapshot:claude-code" needs correction -- it is a PORT, not a snapshot

Anchor sentence: closest anchor is crush (B 69.0) -- terminal agent with a permission service and summarization loop; claw's design, however, is entirely Claude Code's shape.

## Verdict: Python reimplementation ported from the leaked Claude Code npm source (not a renamed snapshot)

- The code is genuinely Python and architecturally original to this repo: 72 modules under `src/` (agent_runtime.py, agent_manager.py, compact.py, bash_security.py...), 20.8k LOC tests with 1,233 `def test` functions. No transpiled-JS smell at triage granularity.
- But the source of truth is the proprietary leak, declared in-code: `src/__init__.py:1` "Python porting workspace for the Claude Code rewrite effort"; `src/bash_security.py:3` "Ported from npm src/tools/BashTool/bashSecurity.ts"; `src/compact.py:3` "Mirrors the npm src/services/compact/compact.ts"; `src/prompt_constants.py:1-20` lists 15 upstream `src/constants/*.ts` files it copies.
- Verbatim proprietary expression embedded: `src/prompt_constants.py:46-49` -- `"You are Claude Code, Anthropic's official CLI for Claude."` shipped as this product's prompt prefix; plus Claude-branded slash commands ("Open the Claude Code sticker order page", `src/agent_slash_commands.py:1886`; guest-passes text `:1990`). Its own runtime prompt hedges at `src/agent_prompting.py:128` ("You are Claw Code Python, a Python reimplementation of a Claude Code-style...").
- No LICENSE file (`ls -a` -- absent). Upstream claude-code is proprietary; a port made from leaked source is not clean-room.
- Activity reality vs census: shallow clone says commits=1, but `git log` HEAD is "Merge pull request #43 from HarnessLab/copilot/fix-issue-42" -> real (small) external contribution flow exists; HEAD 2026-06-22, not dead. No `.github/workflows` -- CI absent; 1,233 tests never machine-validated.

## Scoring (T0 depth; code observed honestly)

arch 6 (clean module split, no god file at triage granularity), verif 5 (big behavior-focused suite incl. `tests/test_agent_tools_security.py`, but no CI runs it), safety 5 (ALLOW/ASK/DENY + PASSTHROUGH validator chain `bash_security.py:27-30,1120-1155` -- real machinery, but regex-class denylist, same anti-pattern as nanocoder), token 5 (compact.py: 9-section summarizer, PTL retry loop, circuit breaker; microcompact.py), orch 5 (agent_manager, background_runtime, hook budget overrides `agent_runtime.py:297`), interop 4 (MCP runtime wired `agent_tools.py:25`, IDE path helpers; no ACP/headless-protocol surface observed), oper 5 (resume `agent_runtime.py:376`, stored sessions), originality 1 (everything visible is Claude Code's design, already familiar in-field), durability 2 (HarnessLab, PR flow exists, alpha 0.1.0, legal cloud), docs-dx 3 (README/TESTING_GUIDE/PARITY_CHECKLIST).

**Weighted 44.0 -> D** (originality+durability floor; borderline C).

## Findings policy

Port-from-leak gets the same treatment as a snapshot: license-risk findings, **no code-adjacent portables**. The only legitimate convergence signal is the regex-denylist anti-pattern (matches nanocoder).
