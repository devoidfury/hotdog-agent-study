# openlumara — T1 review

Anchor question: closest to **crush** — a modular agent core with an opt-in container jail,
one summarization strategy, and a persistent scheduler — but OpenLumara has *no* approval
gate in the loop and *zero* automated verification, so it lands a clear notch below the
crush/nanocoder pair despite a cleaner module split than either.

## Census sanity

- non_test_loc 29,165 matches `cloc` code sum exactly; test_loc 0 is **accurate**: the only
  `*test*` file, `channels/turn_grouping_test.py`, is an interactive TTY viewer for the turn
  stream, not an automated test (no asserts, `while True: input()`).
- contributors 8 confirmed via `git shortlog -sn`: Rose22 authored 1,570 of 1,597 commits
  (98.3%). Census "8 contributors" is technically true but overstates the contributor base.
- head_date 2026-09-12 fresh (not shallow, `.git/shallow` absent); first commit 2026-02-28,
  172 commits since Aug 1: very active ~7-month project. T1 tier correct (29k LOC).
- Provenance: original, self-described and consistent with code; remote
  github.com/Rose22/openlumara. License GPL-3.0 => findings are concepts-only,
  effort_for_us assumes clean-room; no code copying.

## What it is

Local-first personal AI agent framework (Python, ~12.8k LOC core+modules+channels, rest
web UI). Not a coding agent per se; `coder` module adds tree-sitter-validated file ops.
Manager (`core/manager.py`) loads channels + modules; each **channel owns its own
Context/Chat/ToolLoader** (`core/channel.py:49-58`) — per-surface context windows are a
first-class state model. Agentic loop = `Channel.send_stream` (`core/channel.py:582`) ->
`ToolcallManager.process` recursive tool loop (`core/toolcalls.py:94`, recursion at
`:299`), with per-tool `asyncio.wait_for` timeout + real task cancel (`:216-233`) and
`json_repair` on tool args (`:44-66`).

## Per-dimension scores

### architecture 7 (weight 15)
Clean three-layer split (core / modules / channels), everything including shell, memory and
scheduler is a toggleable module; largest files are 968 (channel.py) / 876 (config.py) LOC,
no god files. Loop state model is explicit (per-channel `agentic_loop_start` marker,
`channel.py:57`). Docked: `global_instance` singleton (`manager.py:11,253`), dynamic
`manager.channel` pointer switched on send (`channel.py:204-213`), and send()/send_stream()
maintain two parallel paths through the same semantics.

### verification 1 (weight 15)
Zero automated tests, zero CI (no `.github`, no gitlab config). The single test-named file
is the interactive harness cited above. API layer has broad exception typing
(`core/api.py:302-328`) but nothing asserts anything. Weakest dimension by far; codel got 0
with no tests either — 1 here only because a manual test-channel culture is visible in
commit history ("more manual code review/editing", 985ab9da).

### safety-enforcement 4 (weight 10)
Nothing binds in-loop: no approval flow at all — once a module is enabled its tools execute
unconditionally (`core/toolcalls.py:216`). Safety is (a) module gating — shell modules not
in DEFAULT_MODULES (`core/config.py:196-209`), risky modules carry `unsafe = True` which is
a UI flag only (`core/module.py:53`, `core/commands.py:641`); (b) containment underneath:
docker/podman jail with `--cap-drop ALL`, read-only rootfs, no-new-privileges, cpu/mem/pids
limits, optional gVisor auto-detect (`modules/sandboxed_shell.py:337-350,491-495`) — good
machinery but opt-in, and its `internet_access` setting defaults **True**
(`sandboxed_shell.py:20-22`); (c) content defense in `modules/http.py` (see findings).
Below the 6 rung (which is "tested approval, nothing underneath") because there is no
approval at all; above codel's 2 because the jail and SSRF work are real.

### token-economy 7 (weight 10)
Genuinely distinctive: `tools_load` meta-tool + catalog means the tool array is near-empty
by default, and AI-loaded tool sets persist per chat and restore on resume
(`core/tool_loader.py:186-214,306-321,356-375`; self-healing rejection message
`core/toolcalls.py:200`). Context build measures what would actually be sent (system +
history + end-prompt + tools array, `core/context.py:176-184`) and binary-search trims to a
95% budget with last-user-message reservation (`:186-208`). Manual `/compress` uses a
`SUMMARIZATION_CUTOFF` marker so chat history is kept for the UI and the compaction is
revertible (`context.py:7,58-68`; `core/commands.py:385-401`). AI-visible budget warning at
80% injected via end-prompt (`modules/token_threshold.py:13-34`). Docked: estimator is
`len(text)//4` over the JSON dump (`context.py:280-287`), and prompt-cache warming was
built then disabled as buggy (`core/api.py:126-127`). Above cline's 7 on lazy tooling +
revertible compaction; below pi's 8 (no projected-context trigger, no cache discipline).

### orchestration 5 (weight 10)
Persistent scheduler (msgpack/json store, single dispatcher, recurring jobs with
advance-past-missed policy, one-shot overdue executes, `modules/scheduler.py:59-86`),
per-job prompt strategies trading tokens vs prompt-cache reuse (`scheduler.py:22-33`),
per-channel push queue for AI-initiated messages (`channel.py:110-126`). No subagents, no
budgets, unbounded recursion (toolcalls.py:299), no crash-recovery journal beyond job
storage.

### interop 5 (weight 10)
Five surfaces (webui, cli, telegram, discord with E2EE-capable matrix). No MCP, no ACP, no
SDK. `api_bridge` exposes an OpenAI-compatible chat-completions endpoint — inverted interop
(the agent masquerades as a *model* rival harnesses can consume,
`channels/api_bridge.py:98-104`) — real but niche. Client side works with any
OpenAI-compatible backend (llamacpp/koboldcpp/ollama).

### operability 6 (weight 10)
Chat persistence + auto-resume, per-chat tool-state restore, `/status` `/context`
diagnostics, log broadcast to all channels, `/restart`, migration path that hard-aborts if
the backup write fails (`core/chat.py:110-122` — data-loss-aware). No file checkpoint/revert,
no crash-recovery posture beyond that, chat IDs truncated ULIDs with a known collision
comment (`chat.py:196`). Comparable to nanocoder's 6.

### originality 7 (weight 10)
Verified in code, not marketing: reversible summarization cutoff; lazy tool catalog with
per-chat persistence; model-visible token budget that makes the AI warn the user; per-channel
context windows; 6-layer injection screening with random-delimiter untrusted envelopes
(`http.py:816-848`); TurnCollector segmenting streams into UI-agnostic typed turns
(`core/turns.py:85-120`). crush/nanocoder rung.

### durability 3 (weight 5)
Very active (172 commits in Sept) but bus factor 1 (98.3% one author), no SECURITY.md, no CI,
no funding/institution, 7 months old, governance = sole maintainer + AI-usage ban policy
(CONTRIBUTING.md). GPL-3.0. Slightly below nanocoder's 4: same one-person risk, less external
signal.

### docs-dx 7 (weight 5)
Extensive in-repo docs: dev_docs with a per-core-file reference (16 docs under
`openlumara_docs/dev_docs/core/`), user guide, per-channel docs, plus an embedded `tutorial`
module teaching the AI itself. Install is clone + run.sh. Docked from a potential 8: no
SECURITY.md, and README drifts slightly ("modules module disabled by default" — the module is
on by default, its `toggle` tool is what's gated off, `modules/modules.py:23-24`). 7.

## Totals

architecture 7, verification 1, safety 4, token-economy 7, orchestration 5, interop 5,
operability 6, originality 7, durability 3, docs-dx 7
weighted = 10.5 + 1.5 + 4 + 7 + 5 + 5 + 6 + 7 + 1.5 + 3.5 = **51.0 -> band C**

Strongest dimension: token-economy. Weakest dimension: verification.
No calibration demotions applied.
