# subjects-aider: integrated T3 review (run 20260929-0844-t3-giant-review-4, node integrator)

Subject: aider (`/data/samples/agents/aider`, upstream main @ `5dc9490bb`, v0.86.3.dev-53, HEAD 2026-05-22).
Lanes integrated: reviewer-core (9 findings, arch 6 / op 6), reviewer-safety (9 findings, safety 5 / verification 6), reviewer-econ (10 findings, token 7 / orch 5 / interop 5 / docs-dx 9).
Merged findings: `merged-findings.jsonl` -- 26 records from 28 inputs, deduped by concept (see Dedup log).

## Verdict

aider is the band's best-documented, most cache-disciplined, most single-maintainer coder: a genuinely original product idea (repo-map PageRank context, edit-format dialects, architect/editor relay) executed inside a 2,485-LOC god class whose core-loop control flow has zero direct tests, with an approval-only safety model that fails closed at exactly one boundary (shell) and open at all others. Weighted total **60.5 -> band C**. Closest anchor: **pi**.

## Anchor placement (resolved across lanes)

The lanes' placements are per-dimension, not contradictory; integrated stance: closest overall subject is **pi** (econ's explicit call), with core's placement (closer to crush than codex/pi/nanocoder on architecture+operability) retained for those two rungs only, and safety's placement (below the cline/pi/crush 6-rung, nanocoder-adjacent) retained for safety-enforcement. aider is above every anchor on docs-dx, at or near the top on token-economy's cache half, and below all except codel on safety-enforcement, orchestration and interop.

## Score reconciliation (explicit, per disagreement)

1. **overflow-hard-stop impact: core med (aider-c8) vs econ high (aider-e2).** Resolved **high**. Both lanes describe the same mechanism (`base_coder.py:1464-1466, 1536-1547, 1628-1676`); econ's record carries the anchor contrast (codex's `compact_token_budget.rs` tier vs aider's advice-only `check_tokens :1396-1417`) and this is the single ceiling separating aider from the 8/9 token rungs -- sessions terminate, they do not degrade. Kept e2, aliased c8, folded c8's unique `FinishReasonLength` path (:1495-1499) into the record.
2. **One summarizer, two findings: core aider-c4 (equality-guarded-summarizer-race) vs econ aider-e1 (background-summarizer).** Same mechanism (`base_coder.py:1002-1038`, `history.py:33-96`). Merged into one canonical record `equality-guarded-background-summarizer` keeping econ's budget/model-ladder evidence and core's lock-free race-guard + cache-warm-sharing evidence. No score conflict: core treated it as architecture nuance, econ as the token-economy 6-rung feature -- both readings survived in the merged record.
3. **Operability: safety's "crush-8-style recover posture" gloss (aider-s8) vs core's operability 6.** Resolved **6**. The crush 8-rung posture requires recover middleware and durable resume; aider-e7 shows none exist (no queue/journal/sqlite; resume = markdown reload, rewind = single-step `/undo`). Aider's posture is interactive-CLI-conservative (double-^C :986-999, per-message containment :1507-1512, opt-in-only transmission) -- the nanocoder rung, edges slightly better, exactly as core scored.
4. **Verification corpus: census 76k vs recount 12,381.** Both lanes and the scout recount; the census number is the website doc tree (scout §0). Scoring uses **12,381 LOC / 473 tests / 41 files**; the census figure is rejected study-wide, consistent with the crush/cline/nanocoder errata.
5. **Docs-dx 9 mass caveat.** Econ's 9 stands (docs match code: caching.md documents the real chunk order in `chat_chunks.py`, watch.md documents the real `ai`-comment regex at `watch.py:70`), but the record and the cross-subject note flag that a large share of the 84-page mass is the website/blog/benchmark tree; LOC-weighted comparisons of this metric will mislead against pi's denser docs.
6. **Summarizer race severity.** Core rated the silent-discard branch a med-confidence design ceiling, econ treated the equality guard as a feature credit. Both kept: the guard earns the 6-rung credit (it is a correct-if-crude optimistic publish, aider-e1) and the silent-drop + deferred-join branch is recorded in the same record as the documented ceiling.

## Scores (rung scale 0-10 each; weight in parens)

| dimension | score | weight | weighted | basis |
|---|---|---|---|---|
| architecture | 6 | 15 | 9.0 | core: real strategy factory + typed chunks (aider-c2) above nanocoder 5; god class + implicit shared state + 824-line main() (c1, c3, c5) keep it under crush 7 |
| verification | 6 | 15 | 9.0 | safety: real property tests at approval/edit/undo sites, but 12.4k-LOC corpus, mocked providers, gate-site untested (s6, s7, c9); 8 unreachable per frozen errata |
| safety-enforcement | 5 | 10 | 5.0 | safety: approval-only, zero sandbox/allowlist/injection fencing (s1, s3); credited for the tested shell fail-safe and unconditional git gates (s2, s4); below the cline/pi/crush 6-rung, level with nanocoder for opposite reasons |
| token-economy | 7 | 10 | 7.0 | econ: real cache breakpoints + warming + cost-correct math (e3) + budget projection (e4) = 7; no projected/auto trigger, overflow hard stop (e1, e2), images unbudgeted (e5) |
| orchestration | 5 | 10 | 5.0 | econ: one hard-wired architect->editor relay with cost/hash pullback (e6) beats codel 4; no queue/daemon/subagent/journal (e7) keeps it under nanocoder 6 |
| interop | 5 | 10 | 5.0 | econ: best-in-class model-side breadth (litellm + 3,128-line settings + OpenRouter DB, e9) but zero protocol surface -- no MCP/ACP/json (e8); under pi's 6 whose headless contract is machine-usable |
| operability | 6 | 10 | 6.0 | core: visible crash posture, defensive /undo guard set (c7, s8); session-scoped markdown+git recovery only, no store/tree/crash-resume (c6) |
| originality | 7 | 10 | 7.0 | integrator, verified in code (below) |
| durability | 6 | 5 | 3.0 | integrator, from census + git metadata (below) |
| docs-dx | 9 | 5 | 4.5 | econ: pi's 9-rung (e10), mass caveat noted |
| **weighted_total** | | 100 | **60.5** | **band C (45-64)** |

## My two dimensions, scored with code evidence

### Originality: 7/10 (weight 10)

Verified the ideas are in code, not just prose, by direct read-only inspection:
- **Repo-map PageRank** -- `repomap.py:365-543`: builds a tree-sitter tag reference graph and runs `networkx.pagerank` with personalized teleport (`:525`, fallback `:529`), token-budgeted against the window (`models.py:782-789`, `repomap.py:115-127`). Real, load-bearing, and the most-copied context idea in the agent world.
- **Edit-format strategy factory** -- `base_coder.py:125-201`: registry walk over `coders.__all__` (`:189`), `UnknownEditFormat` with enumerated valid formats, history re-summarization on dialect switch (`:160-168`) to prevent prompt contamination; per-dialect prompt modules and per-dialect tests (`test_editblock.py` 618 LOC, `test_udiff.py`, `test_wholefile.py` 359).
- **Typed cache-boundary context model** -- `chat_chunks.py:5-44`: chunk taxonomy with explicit `all_messages()` order and `add_cache_control_headers` placing breakpoints on the stable prefix; `caching.md` documents exactly this order (code-first, not marketing-first).
- **Watch mode** -- `watch.py:70`: the `ai`-comment regex filter is real and drives file-add -> run (`:194, :210`).
- **`explicit_yes_required` fail-safe** -- `io.py:867`: `res = "n" if explicit_yes_required else "y"` under `--yes-always`; a genuinely clever inversion of the approve-everything flag, tested (`test_io.py:180-183`).
Docking from 8-9: the scaffolding itself is conventional (litellm wrapper, prompt_toolkit REPL, pexpect shell); aider's signature ideas have been absorbed into industry defaults, so within this band they no longer differentiate mechanism-wise -- `search_replace.py` (757-line legacy gpt-3.5 engine) shows the accretion model; the architect relay (e6) is a fixed pipeline, not a general idea. Product-level originality high; mechanistic originality relative to this band: 7.

### Durability: 6/10 (weight 5)

From census + `git` metadata on the snapshot:
- **Cadence**: 13,138 commits total; last 12 months (2025-06..2026-05) = 332, but H1-2026 (90) is down ~62% from H2-2025 (242), with 2026-05 at 5 commits and the snapshot 4 months stale at run date. Alive at snapshot, decelerating.
- **Contributors**: 199 named, but Paul Gauthier across three aliases holds 12,614/13,138 = **96%** of all commits; next contributor has 32. Severe bus-factor.
- **License**: Apache-2.0 (LICENSE.txt, OSI classifier in pyproject.toml) -- clean.
- **Institutional backing**: real org remote `Aider-AI/aider`, release plumbing in-tree (release.yml, docker-release.yml, check_pypi_version.yml), onboarding/OAuth (onboarding.py:46-81), maintained leaderboards. No SECURITY.md, no maintenance policy (aider-s9).
- Not archived and not dead at snapshot, so the archived/dead B-cap does not fire; the deceleration and 96% concentration dock it from the 7-8 range to 6.

## Strongest / weakest (report requirement)

- **Strongest: docs-dx (9/10, 90% of its weight).** Evidence: 84 in-repo doc pages whose substance tracks code (`caching.md` <-> `chat_chunks.py:8-41`; `watch.md` <-> `watch.py:70`; `troubleshooting/token-limits.md` <-> `check_tokens`/`show_exhausted_error` advice text at `base_coder.py:1400-1414, 1656-1675`), deprecation shims with migration strings (`deprecated.py:1-21`), tested scripting docs (`scripting.md` <-> `test_scripting.py`). Caveat: mass includes the website tree.
- **Weakest: safety-enforcement (5/10; tied numerically with orchestration and interop, named here because it carries the risk).** Evidence: zero sandbox/allowlist/injection machinery (grep 0 hits, aider-s1/s3); shell = pexpect of a raw string (`run_cmd.py:111`) gated by one confirm; `--yes-always` auto-approves the whole edit surface and API mode enables it silently (`main.py:546-547`, aider-s5); the gate at the dangerous call site is mocked away in tests (`test_coder.py:994`, aider-s6); no SECURITY.md honesty statement (s9). What saves it from the codel 2-rung: the tested shell fail-safe (`io.py:867`, s2) and unconditional git gates (s4).

## Dedup log

- `aider-c8` + `aider-e2` -> one `overflow-hard-stop` record, canonical `aider-e2`, alias `aider-c8` (econ evidence stronger: anchor contrast + impact high; c8's `FinishReasonLength` path folded in).
- `aider-c4` + `aider-e1` -> one record, canonical kebab `equality-guarded-background-summarizer` kept at id `aider-e1`, alias `aider-c4` (economy record had the budget/ladder depth; core's race-guard and lock-free-sharing evidence folded in).
- Kept distinct on purpose: `aider-c7` (checkpoint-revert mechanism) vs `aider-s4` (approval-independent enforcement gates) -- they share the `/undo` guard-set citation but score into different dimensions (operability vs safety); `aider-c6` (transcript resume fidelity) vs `aider-e7` (absence of queue/journal machinery) -- persistence vs orchestration.

## Calibration notes

1. aider is the **upstream original**, not a sync-fork (scout §2: origin Aider-AI/aider, continuous 13k-commit history, MCP absence native per `git log --all -S mcp`); the "derivative must not outrank upstream" rule cannot fire and no demotion was applied.
2. No archived/dead cap: active through the snapshot (HEAD 2026-05-22, release CI green pattern preserved), deceleration recorded in durability 6 instead of a band cap.
3. Census artifact corrected study-wide: verification and any corpus-size claims use the 12,381-LOC/473-test recount, not census 76k (website docs tree); docs-dx 9 carries the same mass caveat for cross-subject comparisons.
4. Weighted total formula: each dimension scored on the 0-10 anchor-rung scale, contribution = score/10 x weight; weights sum to 100. 60.5 lands mid-band C; S-band floor conditions (>=88, no safety/verification below 5) are moot at this total, and verification is exactly at the 5-line, not below.
5. Cross-lane verdicts: all three lane reviewers returned `pass`; no lane output was overridden on evidence, only merged/re-ranked per the reconciliation section.
