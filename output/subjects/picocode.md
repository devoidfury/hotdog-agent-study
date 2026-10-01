# picocode (jondot) — T0

**Anchor answer:** Closer to pi than crush in aspiration (thin core loop, provider-agnostic, everything else minimal), but nowhere near pi's depth; honest placement is a disciplined toy, below the nanocoder rung.

**What it is.** ~1.8k LOC Rust coding agent built on the `rig` framework, pitched at CI workflows and small codemods; MIT; created 2026-01-16, last push 2026-01-30 (GitHub API) -- quiet 8 months at snapshot, not dead >12mo. Census T0 accepted.

**Loop design.** REPL calls `Agent.prompt` with a growing history vec (src/agent.rs:36-53); the tool loop itself is delegated to rig. Every tool is wrapped in a `guard` that prompts for confirmation unless `yolo`, honors an AtomicBool "always" grant, and a regex `bash_auto_allow` list (src/agent.rs:375-384), plus a `tool_call_limit` budget (src/main.rs:33).

**Provenance sanity.** Original, MIT file+manifest agree, no rename suspect. Shallow clone: remote `pushed_at 2026-01-30` confirms HEAD date. **Census correction: test_loc 0 is wrong** -- inline Rust `#[cfg(test)]` module with 7 property tests at src/tools.rs:339-427 (~90 test LOC); the known Rust blind spot.

**Scores (weight):**
- architecture 5/10 — small honest modules (agent/tools/output/config/persona), but own loop is a thin rig wrapper; no state model beyond a Vec.
- verification 4/10 — 7 adversarial tests for `validate_path` incl. `../../etc/passwd`, `///etc/passwd` (src/tools.rs:405-425); CI runs `cargo test` (ci.yml:41); loop/tools untested.
- safety-enforcement 5/10 — component-walking path containment (src/tools.rs:34-58, tested) + per-tool confirmation with bash allowlist; no OS sandbox; yolo bypasses all.
- token-economy 2/10 — `tool_call_limit` only; no compaction, no cache discipline, unbounded history.
- orchestration 1/10 — single agent, no resume/queue/subagents.
- interop 2/10 — headless `run_once` incl. stdin pipe (src/main.rs:159-181), 18 providers via rig; no MCP/SDK.
- operability 3/10 — YAML config + personas; no sessions/resume.
- originality 4/10 — declarative non-interactive "recipes" with `error_if` output gating for CI (picocode.scan-skills.yaml:12-18); persona system.
- durability 3/10 — solo author, 61 stars, CI + release workflows, 8mo quiet (remote-verified).
- docs-dx 4/10 — README + install.sh + annotated config example.

**Total 34.0 -> D.** No calibration demotions. Census correction recorded: test_loc 0 -> ~90 (inline Rust tests).
