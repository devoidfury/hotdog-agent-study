# codex-infinity -- differential triage (T0) vs codex (anchor S 88.5)

Anchor sentence: closer to codex than any other anchor -- it *is* codex's tree plus a patch layer.

## Verdict: sync-fork (confirmed), with a real feature delta on top

- Census flag `sync-fork:codex` upheld. Mechanism: `scripts/daily-upstream-sync.sh:1-40` -- a personal cron (hardcoded `/home/lee/code/codex`, `GIT_SSH_COMMAND` with `~/.ssh/codex_agent_key`) that fetches `https://github.com/openai/codex.git` and merges upstream/main daily. LICENSE and NOTICE byte-identical to upstream (`diff -q codex/LICENSE codex-infinity/LICENSE` clean).
- File-level diff vs `/data/samples/agents/codex`: 1,430 files differ, 163 only-in-fork, 232 only-in-upstream (consistent with 7,006/8,428 hash-identical at a 4-day-behind snapshot). Core-loop drift is small and mechanical: `codex-rs/core/src/session/turn.rs` 3,167 vs 3,125 lines; delta is upstream's newer `preempt`/steer-drain and `cyber_access_program` plumbing absent in the fork (turn.rs:637-643, 1360-1371 upstream) -- i.e., mostly version lag, not rewrite.
- Genuine fork-side additions (absent from upstream tree): auto-continuation loop `core/src/auto_next_prompt.rs` + `core/src/goals.rs:1-5` ("persisted thread goals"), `core/src/arc_monitor.rs:15-25` (end-of-turn monitor POSTing context for SteerModel/AskUser outcomes -- the "runs forever" pitch, README.md:8), `core/src/tasks/undo.rs`, `tools/handlers/search_tool_bm25.rs`, `core/src/plugins/manager.rs`, multi-provider chat wire (`codex-api/src/endpoint/chat.rs`, cerebras etc. README.md:59-82), `core/src/seatbelt_platform_defaults.sbpl`, vendored bwrap (`linux-sandbox/src/vendored_bwrap.rs`). New modules ship with colocated `*_tests.rs` (arc_monitor_tests.rs, goals area, undo) -- fork discipline follows upstream test conventions.
- Compaction/permission spot read: `core/src/compact*.rs`, compaction suite tests present and only drift-by-version; sandbox crates (linux-sandbox, windows-sandbox-rs, seatbelt) intact plus one added sbpl. No weakening found at triage granularity.

## Scoring (T0: dimensions carried from anchor where verified in-tree, not re-derived line-by-line)

Codex's dimensions hold for the carried tree (arch 9, verif 8 on 441k test LOC + nextest CI workflows, safety 10 sandbox crates present, token 9, orch 9 + goal/auto-next, interop 9, oper 9). Docked: durability 9->2 (single maintainer, personal-cron sync infra, `/home/lee` paths, contributors=1 with the shallow caveat), docs-dx 7->6 (fork README is a product pitch), originality 9->8 (auto-next/goals/arc-monitor are real but small vs inherited corpus-familiar codex design).

**Weighted 83.5 -> A.** Rule (a) respected (83.5 < 88.5 upstream). Not promoted to T3: the delta is a patch series, not an independent architecture.

## License

Apache-2.0 clean: LICENSE + NOTICE identical to upstream, no stripped attribution. No license-risk finding.
