# claude-engineer (Doriandarko) — T0

**Anchor answer:** Closer to codel than nanocoder: same demo-grade zero-verification/zero-enforcement posture, with the delta being a genuine Claude tool-use loop and the self-authored-tools idea.

**What it is.** 2024 single-contributor Python CLI (+ Flask web UI) around Claude 3.5/3.7 tool use; ~4.8k non-test LOC census verified (~5.3k Python actual). No project tests: `test.py` is a generated demo artifact (sum/median exercise), not a test of the harness.

**Loop design.** REPL -> `Assistant.chat()` -> `_get_completion()` which loops on `stop_reason == "tool_use"`, dispatching via dynamic import of the `tools/` package and appending results (ce3.py:360-374, `_execute_tool` ce3.py:261-279). History is an unbounded list; context pressure is handled by a soft `total_tokens_used >= MAX_CONVERSATION_TOKENS` warning/reset gate (ce3.py:353-356), not compaction.

**Provenance sanity.** Original upstream; census agrees. T0 accepted. Shallow clone resolved remotely: `pushed_at 2024-12-12` (GitHub API) — dead 21 months, HEAD date honest. **No license file** (GitHub `license: null`) -> pure-concepts porting only.

**Scores (weight):**
- architecture 4/10 — one 528-LOC `Assistant` god-class, but clean `BaseTool` plugin contract (tools/base.py:1-26).
- verification 0/10 — no tests, no CI (no `.github/`).
- safety-enforcement 1/10 — zero approval code anywhere (grep confirm/approve/permission: no hits); tool calls execute local file writes/subprocess unconditionally (ce3.py:368); e2b sandbox exists only for one opt-in code tool (tools/e2bcodetool.py:2).
- token-economy 3/10 — per-turn token accounting with a max-token gate (ce3.py:310-313); no compaction, no cache discipline.
- orchestration 2/10 — nothing beyond refresh/reset REPL commands (ce3.py:423-429).
- interop 2/10 — CLI + web UI only; no MCP/headless contract.
- operability 3/10 — .env config, tool refresh at runtime; no session persistence.
- originality 6/10 — model writes new tool classes into `tools/` at runtime then hot-reloads them (tools/toolcreator.py:44-65 + refresh command); real in code, era-distinctive.
- durability 2/10 — 1 contributor, no license, dead 21mo despite 11.2k stars.
- docs-dx 3/10 — readable README/onboarding; behavior matches docs.

**Total 25.5 -> D.** No calibration demotions applied. Census corrections: none (test_loc 0 is correct per census definition; test.py is not harness tests).
