# claii (agencyswarm) — T0

**Anchor answer:** Closer to codel than nanocoder: a tutorial-grade Gemini loop with real path guards but zero tests-that-run, zero CI, and a README describing features that do not exist in code.

**What it is.** ~1.1k LOC Python Gemini (google-genai) CLI agent; 3-day burst (created 2025-11-15, last push 2025-11-17, GitHub API), 6 stars. Census T0 accepted; no license -> pure-concepts porting only.

**Loop design.** Bounded tool-call loop: `for _ in range(MAX_AGENT_STEPS=20)` dispatching function calls through a hand-built FUNCTION_MAP, with history pruned to last 200 messages (`_prune_messages`, claii/agent.py:60-68, 272) and full conversation persisted to `.claii_memory.json` between runs (claii/memory.py:7, load/save).

**Provenance sanity.** Original-looking tutorial build; shallow clone resolved remotely, pushed_at matches HEAD (quiet 10mo, under the 12mo death bar but effectively over after a 3-day life). Census test_loc 0 near-correct: `tests.py` is a demo driver and `calculator/tests.py` imports a nonexistent `pkg.calculator` -- nothing runnable, though a literal-minded glob would count them.

**Scores (weight):**
- architecture 3/10 — agent.py duplicates main.py's tool declarations and dispatcher (claii/agent.py:70-80 vs main.py:14-36); no shared core.
- verification 2/10 — tests exist but cannot run (calculator/tests.py:2 imports `pkg.calculator`, absent); no CI.
- safety-enforcement 3/10 — containment guard applied uniformly across all 5 tools (write_file.py:15-19, run_python.py:25-29), but via `abspath().startswith()` which a sibling-prefix path defeats, and run_python executes arbitrary Python with no approval.
- token-economy 3/10 — message-count pruning + cross-run memory file; no token measurement, no compaction.
- orchestration 2/10 — the 20-step cap is the only control (claii/agent.py:24,272); memory is append-all, no resume semantics.
- interop 1/10 — no MCP, no headless contract despite README.md:78 claiming "MCP integration" -- zero mcp references in code.
- operability 3/10 — --verbose/--no-memory/--no-prune flags + .claii_memory.json persistence (memory.py:7).
- originality 3/10 — @kb/ inline expansion and @file: hints at prompt level (agent.py:84-100); nothing else novel.
- durability 1/10 — solo, 6 stars, no license, 3-day lifespan, quiet 10mo.
- docs-dx 2/10 — long README contradicted by code (MCP.md:83 "Multi-Agent Swarms" absent) = present-but-misleading rung.

**Total 24.0 -> D.** No calibration demotions. Census note: test files exist but non-functional; test_loc 0 defensible.
