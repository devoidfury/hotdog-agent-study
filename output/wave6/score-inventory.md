# Wave 6 - Score Inventory

Sources: `output/scores/*.json` (103 files), `output/anchors/scores-*.json` (6), `output/boundary/scores-*.json` (25). Baseline before this session was 101 main files; `workground2.json` (78.25 A, mtime 2026-09-30 20:40) and `mistral-vibe.json` (73.5 B, mtime 20:42) arrived during the audit and are included per the dispatcher's scope update.

## Key-style normalization (scores)

- `lane_scores` keys appear in two styles: `kebab` (e.g. `safety-enforcement`) and `snake` (`safety_enforcement`). Snake-style files: 62 in output/scores + 2 in output/boundary (boundary snake: scores-3code.json, scores-kolkrabbi.json). Canonical rule: `key.replace('_','-')`; weights are position-independent after that.
- 6 files nest lane scores as objects `{score, weight, weighted, lane}` instead of bare numbers: aider, code, continue, kilocode, opencode, roo-code (all T3 outputs). Canonical rule: take `.score`. In `code.json` the embedded `weighted` fields are wrong by 10x (e.g. architecture weighted=75.0 for score 5 x w15); the file's `weighted_total` is nonetheless consistent with score x weight/10.
- `weighted_total` check (recompute sum score*weight/10): all consistent within 0.01 except **goose: stated 73, recomputed 74.5** (band B either way).
- Band-field anomalies: **open-interpreter: weighted_total 78.5 but band recorded `D`** (band_of(78.5)=A; no calibration demotion documented in the file); **mimo-code: band recorded `C (provisional)`** -- non-canonical band string (78->no; 60.0 -> C, band ok, string needs normalizing).
- Only S-band subject: codex (88.5); S rule satisfied (safety-enforcement 10, verification 8).

## Distribution stats (over the 103 output/scores rows)

- 80+: **8 / 103 = 7.8%** -- sanity rule (>25% => re-anchor) **PASSES**
- 80+ subjects: codex 88.5, deepseek-reasonix 86.0, gemini-cli 84.5, codex-infinity 83.5, hermes-agent 83.0, grok-build 82.75, oh-my-pi 82.0, qwen-code 80.0
- Per band: S=1, A=15, B=49, C=19, D=18, C (provisional)=1

- Within 0.75 of a band boundary (88/78/65/45) -> **15 boundary subjects** needing fresh-reviewer adjudication per tier-list protocol: 3code (65.75, B), amazon-q-developer-cli (65.0, B), auto-code-rover (45.5, C), claurst (65.0, B), codewhale (78.0, A), codex (88.5, S), grinta-coding-agent (78.5, A), hax (65.0, B), kilocode (77.5, B), open-interpreter (78.5, D), opensquilla (77.5, B), orca-agent (78.75, A), pi (78.5, A), workground2 (78.25, A), zeroclaw (78.25, A)
- All other 88 subjects are comfortably off-boundary (flagged OK in the table).

## Table (subject, weighted_total, band, boundary flag)

flag: `BOUNDARY` = within 0.75 of 88/78/65/45; `ok` otherwise. src: B = has output/boundary/scores-<name>.json, A = has output/anchors/scores-<name>.json.

| subject | weighted_total | band | flag | src |
|---|---|---|---|---|
| 3code | 65.75 | B | BOUNDARY | B |
| SWE-agent | 59.0 | C | ok |  |
| aeon | 69.0 | B | ok |  |
| agentless | 42.0 | D | ok |  |
| agentty | 76.5 | B | ok | B |
| aider | 60.5 | C | ok |  |
| amazon-q-developer-cli | 65.0 | B | BOUNDARY | B |
| atomic-agent | 74.25 | B | ok |  |
| auto-code-rover | 45.5 | C | BOUNDARY | B |
| binharic-cli | 40.0 | D | ok |  |
| bitfun | 74.5 | B | ok |  |
| claii | 24.0 | D | ok |  |
| claude-engineer | 25.5 | D | ok |  |
| claurst | 65.0 | B | BOUNDARY | B |
| claw-code | 53.5 | C | ok |  |
| claw-code-agent | 44.0 | D | ok | B |
| cline | 76.5 | B | ok | BA |
| code | 68.5 | B | ok |  |
| codebuff | 67.0 | B | ok | B |
| codel | 23.0 | D | ok | A |
| codemachine-cli | 42.5 | D | ok |  |
| codewhale | 78.0 | A | BOUNDARY |  |
| codex | 88.5 | S | BOUNDARY | A |
| codex-infinity | 83.5 | A | ok |  |
| continue | 59.75 | C | ok |  |
| coro-code | 42.0 | D | ok |  |
| crab-code | 72.0 | B | ok |  |
| crush | 69.0 | B | ok | A |
| cursor-agent | 33.5 | D | ok |  |
| darce-cli | 32.5 | D | ok |  |
| deepagents | 76.0 | B | ok | B |
| deepseek-reasonix | 86.0 | A | ok |  |
| developer | 21.0 | D | ok |  |
| devon | 33.5 | D | ok |  |
| dexto | 73.5 | B | ok |  |
| dvalincode | 71.0 | B | ok |  |
| ferrum | 63.0 | C | ok | B |
| forge | 71.25 | B | ok |  |
| forge-norvialabs | 75.0 | B | ok |  |
| free-code | 38.5 | D | ok |  |
| g3 | 55.0 | C | ok |  |
| gemini-cli | 84.5 | A | ok |  |
| goose | 73 | B | ok |  |
| gptme | 76.0 | B | ok | B |
| grinta-coding-agent | 78.5 | A | BOUNDARY | B |
| grok-build | 82.75 | A | ok |  |
| grok-cli | 58.5 | C | ok |  |
| groq-code-cli | 36.5 | D | ok |  |
| hax | 65.0 | B | BOUNDARY | B |
| hermes-agent | 83.0 | A | ok |  |
| ipsupport-code | 71.75 | B | ok |  |
| jazz | 76.5 | B | ok | B |
| jcode | 79.25 | A | ok |  |
| keen-code | 63.0 | C | ok | B |
| kilocode | 77.5 | B | BOUNDARY | B |
| kimi-cli | 73.0 | B | ok |  |
| kimi-code | 76.5 | B | ok | B |
| kode-cli | 70.5 | B | ok |  |
| kolega-code | 76.5 | B | ok | B |
| kolkrabbi | 77.0 | B | ok | B |
| letta-code | 75.25 | B | ok |  |
| maki | 74.5 | B | ok |  |
| memcode | 75.5 | B | ok |  |
| mimo-code | 60.0 | C (provisional) (!) | ok |  |
| mini-kode | 46.5 | C | ok | B |
| minicode | 72.5 | B | ok |  |
| mistral-vibe | 73.5 | B | ok |  |
| mocode | 70.5 | B | ok |  |
| molt | 62.0 | C | ok |  |
| nanocoder | 60.5 | C | ok | A |
| nausicaa-harness | 73.5 | B | ok |  |
| neovate-code | 62.0 | C | ok |  |
| ob-1 | 67.5 | B | ok |  |
| octomind | 79.5 | A | ok | B |
| oh-my-pi | 82.0 | A | ok |  |
| open-codex | 36.0 | D | ok |  |
| open-interpreter | 78.5 | D | BOUNDARY |  |
| opencode | 77.0 | B | ok | B |
| openhands | 72.0 | B | ok |  |
| openharness | 69.5 | B | ok |  |
| openlumara | 51.0 | C | ok |  |
| opensquilla | 77.5 | B | BOUNDARY |  |
| orca-agent | 78.75 | A | BOUNDARY |  |
| ouroboros | 75.5 | B | ok |  |
| pi | 78.5 | A | BOUNDARY | BA |
| picocode | 34.0 | D | ok |  |
| plandex | 55.0 | C | ok |  |
| prime-agent | 72.0 | B | ok |  |
| qqcode | 60.0 | C | ok |  |
| qwen-code | 80.0 | A | ok |  |
| ra-aid | 49.5 | C | ok |  |
| roo-code | 69.5 | B | ok |  |
| san | 74.5 | B | ok |  |
| smelt | 74.5 | B | ok |  |
| tau | 56.5 | C | ok |  |
| trae-agent | 44.0 | D | ok | B |
| tura | 72.0 | B | ok |  |
| vtcode | 76.0 | B | ok |  |
| waveloom | 72.5 | B | ok |  |
| workground2 | 78.25 | A | BOUNDARY |  |
| zap-coding-agent | 56.0 | C | ok |  |
| zeroclaw | 78.25 | A | BOUNDARY |  |
| zot | 67.0 | B | ok | B |

## Reconciliation: subjects with BOTH output/scores/<name>.json and output/boundary/scores-<name>.json

25 subjects have boundary re-reviews. Identical totals (5): cline, gptme, kolkrabbi, opencode, pi.

**Differing totals (20) -- synthesis must pick one with justification:**

| subject | main | boundary | delta | crosses band? |
|---|---|---|---|---|
| 3code | 65.75 (B) | 68.0 (B) | +2.25 | no |
| agentty | 76.5 (B) | 77.5 (B) | +1.00 | no |
| amazon-q-developer-cli | 65.0 (B) | 59.5 (C) | -5.50 | YES - band flip |
| auto-code-rover | 45.5 (C) | 41.5 (D) | -4.00 | YES - band flip |
| claurst | 65.0 (B) | 54.0 (C) | -11.00 | YES - band flip |
| claw-code-agent | 44.0 (D) | 50.0 (C) | +6.00 | YES - band flip |
| codebuff | 67.0 (B) | 66.0 (B) | -1.00 | no |
| deepagents | 76.0 (B) | 76.5 (B) | +0.50 | no |
| ferrum | 63.0 (C) | 62.75 (C) | -0.25 | no |
| grinta-coding-agent | 78.5 (A) | 74.0 (B) | -4.50 | YES - band flip |
| hax | 65.0 (B) | 66.0 (B) | +1.00 | no |
| jazz | 76.5 (B) | 74.75 (B) | -1.75 | no |
| keen-code | 63.0 (C) | 63.75 (C) | +0.75 | no |
| kilocode | 77.5 (B) | 78.5 (A) | +1.00 | YES - band flip |
| kimi-code | 76.5 (B) | 77.0 (B) | +0.50 | no |
| kolega-code | 76.5 (B) | 72.75 (B) | -3.75 | no |
| mini-kode | 46.5 (C) | 45.25 (C) | -1.25 | no |
| octomind | 79.5 (A) | 74.25 (B) | -5.25 | YES - band flip |
| trae-agent | 44.0 (D) | 36.0 (D) | -8.00 | no |
| zot | 67.0 (B) | 66.0 (B) | -1.00 | no |

Band flips among the differing: 7 (amazon-q-developer-cli, auto-code-rover, claurst, claw-code-agent, grinta-coding-agent, kilocode, octomind).

Anchors vs main: all 6 anchor score files (cline, codel, codex, crush, nanocoder, pi) match their output/scores counterparts exactly -- no reconciliation needed.
