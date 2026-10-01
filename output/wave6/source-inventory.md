# Wave 6 - Source Inventory

Audit date: 2026-09-30. Read-only pass; no source file was modified.

## Totals

| source family | files | records |
|---|---|---|
| output/findings/*.jsonl | 80 | 743 |
| output/boundary/findings-*.jsonl | 25 | 151 |
| T3 merged-findings.jsonl (included runs) | 24 | 774 |
| **total** | **129** | **1668** |

Dedup note: `output/findings/gemini-cli.jsonl` (34 records) is a copy of the merged findings of run `20260929-0502-t3-giant-review-3` (verified field-for-field; only JSON string-escaping differs). Unique record count for synthesis = **1634** (keep one copy; recommend the T3 run dir as authoritative).

## T3 run inclusion status

All 24 T3 runs are included. The two runs the original scope excluded as still re-running (20260929-0859-t3-giant-review-3 / workground2, and 20260929-0859-t3-giant-review-8 / mistral-vibe) completed during this audit and were included per the dispatcher's scope update of 2026-09-30. Both now carry integrator.verdict and complete merged-findings.jsonl (41 and 28 records, all parsing).

Caveat: run `20260929-0502-t3-giant-review-3` (gemini-cli) has **no integrator.verdict file**, but its merged-findings.jsonl is complete (34 records, all parse) and `output/scores/gemini-cli.json` exists; included.

Fresh-arrival caveat: `output/scores/workground2.json` (mtime 2026-09-30 20:40) and `output/scores/mistral-vibe.json` (mtime 20:42) appeared during this audit (the initial dir listing showed 102 files; the baseline before those two was 101, matching the dispatch note). Both are included in the score inventory (workground2 78.25 A, mistral-vibe 73.5 B).

## T3 run -> subject map

Subject from run-dir artifact filenames (`scores-<name>.json` / `subjects-<name>.md`), cross-checked against the map.md header.

| run | subject | map.md header | lane counts (core/safety/econ) | merged | integrator.verdict | status |
|---|---|---|---|---|---|---|
| 20260929-0502-t3-giant-review | hermes-agent | Scout map — hermes-agent (run 20260929-0502-t3-giant-review, node scout, attempt 1) | 8/12/12 | 30 | yes | included |
| 20260929-0502-t3-giant-review-2 | qwen-code | Scout map — qwen-code (T3, TypeScript) | 9/11/12 | 32 | yes | included |
| 20260929-0502-t3-giant-review-3 | gemini-cli | Scout map: gemini-cli (run 20260929-0502-t3-giant-review-3) | 10/14/12 | 34 | NO | included |
| 20260929-0502-t3-giant-review-4 | opencode | Scout map: opencode (run 20260929-0502-t3-giant-review-4) | 10/13/13 | 35 | yes | included |
| 20260929-0844-t3-giant-review | code | Scout map — subject `code` ("Every Code", /data/samples/agents/code) | 8/9/10 | 26 | yes | included |
| 20260929-0844-t3-giant-review-2 | kilocode | Scout map — kilocode (run 20260929-0844-t3-giant-review-2) | 9/12/10 | 30 | yes | included |
| 20260929-0844-t3-giant-review-3 | oh-my-pi | Scout map — oh-my-pi (omp), run 20260929-0844-t3-giant-review-3 | 10/12/12 | 32 | yes | included |
| 20260929-0844-t3-giant-review-4 | aider | Scout map: aider (run 20260929-0844-t3-giant-review-4, node scout, attempt 1) | 9/9/10 | 26 | yes | included |
| 20260929-0849-t3-giant-review | bitfun | Scout map: bitfun (OpenBitFun) -- run 20260929-0849-t3-giant-review | 8/10/13 | 31 | yes | included |
| 20260929-0849-t3-giant-review-2 | grok-build | Scout map: grok-build (xai-org/grok-build) | 12/13/18 | 42 | yes | included |
| 20260929-0849-t3-giant-review-3 | continue | Scout map: continue (run 20260929-0849-t3-giant-review-3) | 10/12/10 | 30 | yes | included |
| 20260929-0854-t3-giant-review | zeroclaw | Scout map: zeroclaw (run 20260929-0854-t3-giant-review) | 10/14/14 | 36 | yes | included |
| 20260929-0855-t3-giant-review | codewhale | Scout map: codewhale (run 20260929-0855-t3-giant-review) | 10/12/13 | 34 | yes | included |
| 20260929-0856-t3-giant-review | goose | Scout map: goose (T3) — /data/samples/agents/goose | 7/9/12 | 28 | yes | included |
| 20260929-0859-t3-giant-review | opensquilla | Scout map — opensquilla (run 20260929-0859-t3-giant-review) | 10/10/19 | 38 | yes | included |
| 20260929-0859-t3-giant-review-2 | vtcode | Scout map: vtcode (vinhnx/VTCode) — run 20260929-0859-t3-giant-review-2 | 7/11/12 | 30 | yes | included |
| 20260929-0859-t3-giant-review-3 | workground2 | Scout map: WorkGround2 (subject workground2) | 10/16/15 | 41 | yes | included |
| 20260929-0859-t3-giant-review-4 | jcode | jcode — static scout map (run 20260929-0859-t3-giant-review-4) | 8/9/10 | 27 | yes | included |
| 20260929-0859-t3-giant-review-5 | deepseek-reasonix | Scout map -- deepseek-reasonix (run 20260929-0859-t3-giant-review-5) | 9/15/11 | 35 | yes | included |
| 20260929-0859-t3-giant-review-6 | orca-agent | Scout map: orca-agent (run 20260929-0859-t3-giant-review-6) | 9/15/14 | 38 | yes | included |
| 20260929-0859-t3-giant-review-7 | letta-code | Scout map -- letta-code (run 20260929-0859-t3-giant-review-7) | 10/10/12 | 32 | yes | included |
| 20260929-0859-t3-giant-review-8 | mistral-vibe | Scout map — mistral-vibe (run 20260929-0859-t3-giant-review-8) | 9/8/11 | 28 | yes | included |
| 20260929-0859-t3-giant-review-9 | dexto | Scout map: dexto (truffle-ai/dexto) — run 20260929-0859-t3-giant-review-9 | 12/6/13 | 30 | yes | included |
| 20260929-0901-t3-giant-review | roo-code | Scout map - roo-code (run 20260929-0901-t3-giant-review) | 8/10/14 | 29 | yes | included |

## output/findings counts (80 files)

| file | records |
|---|---|
| 3code.jsonl | 10 |
| SWE-agent.jsonl | 9 |
| aeon.jsonl | 9 |
| agentless.jsonl | 5 |
| agentty.jsonl | 9 |
| amazon-q-developer-cli.jsonl | 12 |
| atomic-agent.jsonl | 12 |
| auto-code-rover.jsonl | 6 |
| binharic-cli.jsonl | 9 |
| claii.jsonl | 2 |
| claude-engineer.jsonl | 2 |
| claurst.jsonl | 9 |
| claw-code-agent.jsonl | 4 |
| claw-code.jsonl | 6 |
| cline.jsonl | 8 |
| codebuff.jsonl | 10 |
| codel.jsonl | 5 |
| codemachine-cli.jsonl | 5 |
| codex-infinity.jsonl | 3 |
| codex.jsonl | 10 |
| coro-code.jsonl | 7 |
| crab-code.jsonl | 11 |
| crush.jsonl | 7 |
| cursor-agent.jsonl | 9 |
| darce-cli.jsonl | 9 |
| deepagents.jsonl | 10 |
| developer.jsonl | 1 |
| devon.jsonl | 9 |
| dvalincode.jsonl | 10 |
| ferrum.jsonl | 10 |
| forge-norvialabs.jsonl | 9 |
| forge.jsonl | 13 |
| free-code.jsonl | 4 |
| g3.jsonl | 10 |
| gemini-cli.jsonl | 34 *dup of T3 run 0502-3* |
| gptme.jsonl | 13 |
| grinta-coding-agent.jsonl | 12 |
| grok-cli.jsonl | 9 |
| groq-code-cli.jsonl | 6 |
| hax.jsonl | 7 |
| ipsupport-code.jsonl | 11 |
| jazz.jsonl | 9 |
| keen-code.jsonl | 11 |
| kimi-cli.jsonl | 12 |
| kimi-code.jsonl | 12 |
| kode-cli.jsonl | 9 |
| kolega-code.jsonl | 9 |
| kolkrabbi.jsonl | 10 |
| maki.jsonl | 15 |
| memcode.jsonl | 12 |
| mimo-code.jsonl | 4 |
| mini-kode.jsonl | 9 |
| minicode.jsonl | 10 |
| mocode.jsonl | 13 |
| molt.jsonl | 9 |
| nanocoder.jsonl | 8 |
| nausicaa-harness.jsonl | 12 |
| neovate-code.jsonl | 7 |
| ob-1.jsonl | 16 |
| octomind.jsonl | 12 |
| open-codex.jsonl | 9 |
| open-interpreter.jsonl | 4 |
| openhands.jsonl | 12 |
| openharness.jsonl | 14 |
| openlumara.jsonl | 11 |
| ouroboros.jsonl | 12 |
| pi.jsonl | 9 |
| picocode.jsonl | 2 |
| plandex.jsonl | 7 |
| prime-agent.jsonl | 10 |
| qqcode.jsonl | 8 |
| ra-aid.jsonl | 9 |
| san.jsonl | 11 |
| smelt.jsonl | 10 |
| tau.jsonl | 7 |
| trae-agent.jsonl | 7 |
| tura.jsonl | 12 |
| waveloom.jsonl | 14 |
| zap-coding-agent.jsonl | 13 |
| zot.jsonl | 7 |

## boundary findings counts (25 files)

| file | records |
|---|---|
| findings-3code.jsonl | 8 |
| findings-agentty.jsonl | 6 |
| findings-amazon-q-developer-cli.jsonl | 6 |
| findings-auto-code-rover.jsonl | 6 |
| findings-claurst.jsonl | 5 |
| findings-claw-code-agent.jsonl | 9 |
| findings-cline.jsonl | 7 |
| findings-codebuff.jsonl | 5 |
| findings-deepagents.jsonl | 4 |
| findings-ferrum.jsonl | 8 |
| findings-gptme.jsonl | 5 |
| findings-grinta-coding-agent.jsonl | 5 |
| findings-hax.jsonl | 6 |
| findings-jazz.jsonl | 8 |
| findings-keen-code.jsonl | 6 |
| findings-kilocode.jsonl | 7 |
| findings-kimi-code.jsonl | 5 |
| findings-kolega-code.jsonl | 4 |
| findings-kolkrabbi.jsonl | 4 |
| findings-mini-kode.jsonl | 7 |
| findings-octomind.jsonl | 7 |
| findings-opencode.jsonl | 9 |
| findings-pi.jsonl | 4 |
| findings-trae-agent.jsonl | 5 |
| findings-zot.jsonl | 5 |

All lines in all listed files parse as JSON (0 malformed lines anywhere).
