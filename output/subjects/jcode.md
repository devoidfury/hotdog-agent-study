# jcode -- T3 integrator report (run 20260929-0859-t3-giant-review-4)

Subject: `/data/samples/agents/jcode` @ `cc21714` (v0.88.0, Rust workspace, ~752k LOC incl. inline tests). Read-only review; the subject was never executed.

**Verdict: band A, weighted 79.25/100.** Closest anchor: **codex** -- daemon + versioned NDJSON protocol + parity-gated SDK is the codex shape, reached from below by a solo project; the mechanisms layered on top (compaction tiering, cache observability, crash-durable orchestration) exceed the cline/pi rungs while safety-enforcement sits at the nanocoder rung.

**Strongest dimension: architecture (9/10).** The agent loop (`crates/jcode-app-core/src/agent/turn_loops.rs:53`) is separated from every product surface by a process boundary the build itself enforces (`jcode-app-core/Cargo.toml:68-70`, CI run of `scripts/check_dependency_boundaries.py` at `ci.yml:87-89`); the TUI, SDK, ACP, headless and SSH clients all attach over a socket (`jcode-tui/src/tui/backend.rs:325`). Exactly one file repo-wide exceeds 5k LOC incl. tests, so the errata 8->9 docking trigger does not fire.

**Weakest dimension: safety-enforcement (5/10).** The deny tier is the strongest deterministic protection in the corpus -- blast-radius gate bound unconditionally into `BashTool::execute` before spawn (`bash.rs:878-885`), catastrophic paths absolute-denied and unconfirmable (`jcode-command-risk/src/gate.rs:77-89`), verified live in code at head -- but nothing else binds: no kernel isolation (grep bubblewrap|seatbelt|landlock|seccomp = 0), no human approval on the interactive path (`docs/SAFETY_SYSTEM.md:5`; pinned by `tool/tests.rs:1105-1119`), write/edit unprotected while bash redirects to the same path are blocked (`apply_patch.rs:182-188` exists; zero `is_catastrophic_target` hits in `edit.rs`/`write.rs`), zero prompt-injection handling despite webfetch/browser/computer/MCP/SSH surfaces.

## Lane scores and the two dimensions the integrator owns

| dimension | weight | score | source |
|---|---|---|---|
| architecture | 15 | 9 | reviewer-core |
| verification | 15 | 8 | reviewer-safety |
| safety-enforcement | 10 | 5 | reviewer-safety |
| token-economy | 10 | 8.5 | reviewer-econ |
| orchestration | 10 | 8.5 | reviewer-econ |
| interop | 10 | 8 | reviewer-econ |
| operability | 10 | 8.5 | reviewer-core |
| originality | 10 | **8.5** | integrator |
| durability | 5 | **6** | integrator |
| docs-dx | 5 | 7.5 | reviewer-econ |

Weighted total = (9+8)*15 + (5+8.5+8.5+8+8.5)*10 + 8.5*10 + 6*5 + 7.5*5 = 792.5 / 10 = **79.25**.

### originality 8.5 (integrator-scored, code-verified, not marketing)

The test for this dimension is whether the unique-kind findings are real in code. I re-read each flagship at head, independently of the lane that found it:
- `tool/agentgrep/context.rs`: `collect_tool_exposures` (:107) genuinely walks the persisted session for read ranges/prior grep/bash output, and `file_freshness_multiplier` (:781-828) decays stale seen-credit by mtime. Nothing like it in any anchor -- context economy applied at the tool-output layer.
- `jcode-command-risk/src/gate.rs:46-101`: reflection gate with a named affirmations blocklist and blind-retry-invariance tests; deny is genuinely uncircumventable at the Confirm/Catastrophic boundary (verified the `gate()` match directly).
- `jcode-base/src/compaction.rs:12-15` + `compaction-core`: three modes plus cache-accounting-corrected measurement -- matches the module doc, and the 413-payload recovery path (`lib.rs:549-711`) treats byte budget and token budget as separate problems, which no anchor does.
- `kv_cache_monitor.rs:1-15`: daemon-side miss classification so all clients inherit it; the module doc states the TUI-only predecessor explicitly.
- `import.rs:35-40`: cross-harness resume with byte-bounded folds (160 msgs / 256 KiB) -- a migration axis no anchor has.
- `bash.rs:878-885`: gate wired before background dispatch, no config toggle.
Dock to 8.5, not 9+: the top-level shape (client/daemon, versioned protocol, SDK parity) is the codex pattern reproduced, and every original mechanism is an enhancement on that shape rather than a new paradigm; the headline Cargo.toml blurb ("Possibly the greatest coding agent ever built") is marketing, but every mechanism checked above survives contact with the code.

### durability 6 (integrator-scored from census metadata + scout remote verification)

Cadence: created 2026-01-05, remote-pushed 2026-09-29 (same window as head `cc21714`) -- hyperactive, not archived. License: MIT, clean. Institutional backing: none -- one effective maintainer (`1jehuang`), so a hard bus factor of 1 against ~752k LOC and 597 open issues. Community traction is exceptional for solo: 20.2k stars, 2,343 forks. Contributors=1 in the census is partly a shallow-clone artifact, but the remote's single-author commit pattern (scout GitHub API check) supports "hyperactive solo," not a team. Scored 6: nowhere near the institutionally-backed anchors (OpenAI/Anthropic-backed codex and claude-code), above dead or low-traction solo projects; the ceiling is the bus factor, the floor is the activity and community.

## Disagreement and conflict resolution (explicit, per instruction)

The three lanes scored disjoint dimensions, so there are no head-to-head numeric disagreements. Four cross-source tensions existed and are resolved as follows:

1. **Scout predicted safety "at best the 6-rung"; reviewer-safety filed 5.** Resolved to **5**. The scout's ceiling was conditional ("if the gate is well-tested"), and the reviewer's file:line evidence goes deeper: the 6-rung property (approval binds in-loop) is tested-against, not merely absent -- `tool/tests.rs:1105-1119` pins that `request_permission` is unavailable to normal sessions, and the write/edit asymmetry (`jcode-s4`) is a concretely exploitable hole. Scout was reasoning from the gate's existence; the reviewer read who consumes it.
2. **Scout flagged `agent.rs:296` as a std-Mutex of UI-facing state (architecture risk); reviewer-core disproved it after reading.** Resolved to **core's correction**: `agent.rs:283-287` is the soft-interrupt queue, std Mutex by documented design ("accessed without async, even while agent is processing"). No architecture dock; the 9 stands.
3. **Hot self-reload appears in two lanes**: core credits it in operability (above the 9-rung, med confidence, execution forbidden) and econ credits the same reload-recovery ledger in orchestration 8.5. Resolved to **keep both credits**: they measure different planes -- session continuity across a binary swap (operability) vs crash-durable swarm coordination surviving the same swap (orchestration) -- and the underlying records (`reload_recovery.rs` pending/delivered ledger, GC, fsync) are the strongest-verifiable-from-reads kind. Recorded as `cross_references` in merged-findings.jsonl (c3<->e5); no double-count penalty because neither lane scored the other's dimension on the strength of this artifact.
4. **Docs rot vs operability docks look like the same complaint in two lanes.** They are distinct: core docked operability for a missing user-facing session/journal format spec and no archive/delete/queue CLI; econ docked docs-dx for a broken docs index, a dangling harness-API doc ref, and an undocumented 2.1k-LOC ACP adapter. Both docks stand in their own dimension; neither is double-penalized since the dimensions are scored and weighted separately.

Merged findings file: 27 records from 8 (core) + 9 (safety) + 10 (econ). No cross-lane duplicate concepts existed to drop. Two concept-id collisions were corrected with canonical kebab ids and the original recorded in `aliases`: the safety lane stamped five distinct findings `permission-policy` (retagged: computed-blast-radius-deny-tier, reflection-gate-justification, no-interactive-approval-gate, no-prompt-injection-posture, telemetry-default-on-framing) and stamped `jcode-s9` (panic classification) with c8's `checkpoint-revert` id (retagged crash-classified-stop-reasons; c8 keeps checkpoint-revert). Evidence was never merged across records because no two records describe one concept.

## Calibration notes

- Provenance: **original** per scout's GitHub API verification (`fork: false`, no upstream-name remnants; all codex/claude-code mentions are import features). The sync-fork-must-not-outrank-upstream rule does not apply. Subject is live and active, so the archived/dead B-cap does not apply. **No demotions were applied.**
- S-band gate is irrelevant here (79.25 < 88), but for the record: safety-enforcement is 5, not below 5, and verification is 8 -- the gate would pass on the floor test.
- Census caveats honored: verification scored from what tests assert and CI runs, not `test_loc` (which undercounts inline `#[cfg(test)]`); durability from cadence/license/backing, not the shallow-clone contributor count.
- The 5-vs-6 safety hinge is documented for in-loop fixes: jcode-s3 (approval ambient-only) and jcode-s4 (write/edit asymmetry) each individually lift safety-enforcement to 6; nothing else in the safety lane needs to move.
- Hot-reload session continuity is scored on med confidence (static reads only; execution forbidden by the read-only rule). If a future dynamic pass contradicts the reload test suite's claims, operability 8.5 is the exposure, not the band.
