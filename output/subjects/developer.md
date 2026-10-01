# developer (smol-ai) — T0

**Anchor answer:** Closer to codel than anything above it: no agent loop at all, oracle-style generation with zero enforcement or verification; the delta is historical influence, not mechanism.

**What it is.** The 2023 smol-ai "developer": a prompt pipeline, not an agent -- `plan()` shared-deps -> `specify_file_paths()` manifest -> per-file `generate_code_sync()` writing straight to disk (smol_dev/main.py:11-72). Plus a v0/ subtree (code2prompt, modal debugger). ~1.5-2k LOC; census T0 accepted.

**Loop design.** There is none: a fixed three-stage sequential pipeline with streaming to stdout; no tools, no execution feedback, no interactivity between stages (smol_dev/main.py:16-72). The FastAPI wrapper (smol_dev/api.py) exposes the same pipeline.

**Provenance sanity.** Original; MIT manifest agrees with file. Shallow clone resolved remotely: GitHub API `pushed_at 2024-04-07` -- dead 29 months, worse than HEAD date suggested (same lesson class as codel). 12.2k stars, org-backed at the time.

**Scores (weight):**
- architecture 3/10 — clean but trivial: 3 prompt functions + write_file; no state model, no loop (smol_dev/main.py:11).
- verification 0/10 — no tests, no CI.
- safety-enforcement 1/10 — writes generated files anywhere `--generate_folder_path` points, no confirmation; modal-run debugger in v0 is the only isolation, off the main path.
- token-economy 2/10 — the whole trick was scoping prompts per file to fit windows (prompts.py structure), but no runtime accounting, compaction, or context measurement.
- orchestration 1/10 — sequential for-loop over file paths (smol_dev/main.py:51); no resume of a half-written manifest.
- interop 1/10 — OpenAI-compatible endpoint only + FastAPI embed claim.
- operability 2/10 — CLI args + streaming progress counter (smol_dev/main.py:21-32); failures abort the whole run.
- originality 6/10 — spec-first "manifest then one-file-at-a-time" generation, the pattern an entire generation of Devin-clones and spec-driven tools copied; distinctive shape, prompt-only mechanism (codel got 5 with less proven influence).
- durability 3/10 — org repo, 12k stars, but dead 29mo remote-verified; bus factor irrelevant now.
- docs-dx 4/10 — famously thorough multilingual README; docs match behavior.

**Total 21.0 -> D.** No calibration demotions. Census accepted as-is (test_loc 0 accurate).
