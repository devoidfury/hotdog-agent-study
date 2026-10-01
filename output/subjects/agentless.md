# agentless (OpenAutoCoder/Agentless) — T1 review

**Anchor question:** Closer to **codel (23.0, D)** than to crush, because like codel it has no
agentic loop, no permissions model, no compaction, and zero self-tests or CI; it sits well above
codel because its gates are real and enforced (syntax → lint → dedup → test-arbitrated rerank),
its context discipline is deliberate, and it is the field-shaping anti-agent result -- but it
never acquires harness machinery, so the codel family is the honest anchor, at the top of the D band.

## What this is

The artifact of the "Agentless: Demystifying LLM-based Software Engineering Agents" paper
(arXiv 2407.01489). Not a coding agent by design: three separate CLI stages -- hierarchical
localization (`agentless/fl/localize.py`), N-sample one-shot repair (`agentless/repair/repair.py`),
and patch validation via generated reproduction + selected regression tests
(`agentless/test/*`, `agentless/repair/rerank.py`) -- chained by JSONL artifacts, run per-benchmark-instance
with ThreadPoolExecutor over SWE-bench datasets. There is no interactive session, no tool-use loop,
no compaction (each shot is a single one-shot prompt), no permission system.

## Census sanity (both LOC fields wrong; tier criterion met for T0)

- `non_test_loc: 34068` is ~4x overstated: the 32,032-line data blob
  `classification/swebench_lite_classifications.csv` was counted as code (total of all text files is
  41,568). Real code is ~8,630 Python LOC; largest Python file 1,141 LOC
  (`agentless/util/postprocess_data.py`).
- `test_loc: 1398` is wrong in both glob and semantics: `agentless/test/*.py` (~1,900 LOC) are the
  *benchmark* harness scripts (they run reproduction/regression tests inside SWE-bench docker),
  not tests of agentless. Self-test LOC = 0; no `.github/` at all; only `.pre-commit-config.yaml` formatting.
- `head_date 2024-12-22` + shallow clone: remote-verified per hygiene rule -- GitHub API
  `pushed_at: 2024-12-22T19:29:31Z` (all branches), `archived: false`, 55 open issues, 1 contributor
  locally. **Dead >12 months confirmed (21 months).** Rule (b) cap at B applies; moot since score is far below it.
- Protocol tier criteria: dead >12mo at HEAD => T0. Census says T1; dispatcher ran T1 anyway; full
  rubric completed regardless.

## Provenance

Original, not a fork: census `upstream_remote` matches local `origin`
(`git remote -v` -> https://github.com/OpenAutoCoder/Agentless). No identity collision in this
corpus (no other "agentless*"; do not confuse with any homonymous unrelated tooling). MIT license
file present, GitHub confirms SPDX MIT -- code-adjacent portables permitted.

## Dimension notes

### architecture (15): 6
Clean three-phase separation with explicit file-artifact state; no god files (max 1,141).
Each stage is argparse + ThreadPool + per-instance JSONL append under a `Lock`
(`repair.py:551-557`). Docked for near-parallel re-implementation between stages
(`repair.py` vs `test/generate_reproduction_tests.py` share the same sample/parse/postprocess
skeleton, cf. `repair.py:431-473` vs `generate_reproduction_tests.py:167-200`), and prompts as
module-level template blobs (`repair.py:33-131`). Above nanocoder's 5 (no UI coupling, state is
explicit files), below crush's 7 (no runtime, wiring trivially duplicated).

### verification (15): 2
Zero self-tests, zero CI. The only enforcement anywhere is pre-commit formatting. The evidence
culture is real but out-of-band: complete run artifacts published on the v1.5.0 release
(README.md:74-78) so benchmark claims are externally inspectable -- that plus ast/lint patch gates
keeps it above codel's 0. Nothing checks that agentless itself still works; a 21-month-old repo
with API-drifted endpoints is unverifiable. `agentless/test/` naming is misleading for census
purposes (harness, not tests).

### safety-enforcement (10): 3
No permission model and no claim of one. The only untrusted-code execution path (running repo
tests) is delegated to SWE-bench's docker harness (`test/run_tests.py:10,19,355`) -- borrowed
isolation, not designed here. Active hole: `eval()` on strings parsed out of model output
(`repair.py:188` `eval(edited_file_key)`; `util/postprocess_data.py:849`) -- a prompt-injected
issue body can reach host RCE in the pipeline process. Local patch application and flake8/ast
checks run on host against a playground copy. Above codel's 2 only because nothing here
*misleads*; nothing binds.

### token-economy (10): 5
No conversational compaction (single-shot prompts need none) but real structural economy:
repo-structure skeleton for localization prompts (`util/preprocess_data.py:651 get_repo_structure`,
`util/index_skeleton.py` libcst pass keeping imports/globals/signatures), hierarchical funnel that
only feeds localized files to repair (`localize.py:100 localize_instance` -> `combine.py`), Anthropic
`cache_control: ephemeral` on the leading message so N-sample repair pays input once
(`util/api_requests.py:145-151`, used at `repair.py:431-473`), usage recorded per trajectory with a
post-hoc cost tool (`dev/util/cost.py:37`, hardcoded 5/15 per-M pricing constants -- stale-by-design).
Not tested; below the crush/nanocoder 6 rung's "usage calculator in-loop" only slightly -- scores 5.

### orchestration (10): 4
Deliberately no subagents/queues. What exists: per-instance crash-resume via JSONL append +
`--skip_existing` (`localize.py:399-401,564-566`; `repair.py:540`), plus a nice provenance touch --
each run snapshots its own config and input locs before executing
(`repair.py:536-545` config dump + `used_locs.jsonl`). Stage chaining is manual README copy-paste;
no budgets, no cross-stage scheduling. Equal to codel's restart-tolerant queue rung.

### interop (10): 3
No MCP, no ACP, no SDK, no IDE surface. Headless-by-CLI with JSONL contracts is genuinely
machine-consumable (unix-style), and `util/model.py` is a two-provider DecoderBase abstraction
(OpenAI + Anthropic). One notch above codel's 2 for the artifact contract and provider abstraction.

### operability (10): 4
Per-instance logging, resume flag, cost accounting, a 374-line stage-by-stage reproduction guide
with expected outputs (README_swebench.md). Bare `except:` swallowing exists
(`util/index_skeleton.py:38`). Onboarding is conda + git-pinned swebench + API keys; only a
benchmark workload is operable -- there is no "run on my repo" path at all.

### originality (10): 8
The anti-agent thesis, implemented: hierarchical localize -> top-N sampled one-shot repair ->
test-arbitrated rerank with majority voting (`rerank.py:156 majority_voting`,
`_load_results` folding reproduction + regression outcomes). This is field-defining historically
(it reshaped scaffolding thinking across the whole corpus era) and the mechanisms are verified in
code, not marketing. Not 9-10 because each mechanism is simple prompt engineering and the stance is
now absorbed into hybrid harnesses everywhere.

### durability (5): 1
Remote-verified dead (pushed_at 2024-12-22, ~21 months), 1 contributor, 55 unattended open issues,
no CI, no SECURITY.md. Institutional paper pedigree (OpenAutoCoder org, 2.1k stars, published
artifacts) keeps it above codel's 0 but it will not merge a PR.

### docs-dx (5): 5
README + README_swebench.md are honest and match the code (no marketing-behavior contradictions,
unlike codel's 3); cost reporting documented. Nothing on internals, security, or non-benchmark use.
Between crush's 6 (broad config docs) and codel's 3.

## Totals

6*1.5 + 2*1.5 + 3*1 + 5*1 + 4*1 + 3*1 + 4*1 + 8*1 + 1*0.5 + 5*0.5 = **42.0** -> band **D**.
Strongest: originality (8). Weakest: durability (1).
Boundary: 3.0 below the C line (45); below the 45.5 assigned to auto-code-rover, consistent with
its total absence of tests/CI vs ACR's CI-tested tooling.

## Calibration notes

- Rule (b): archived/dead caps at B -- dead remote-verified, applied, non-binding at 42.0.
- Rule (a): N/A (original, not a fork).
- Census corrections filed: non_test_loc 34068 -> ~8.6k real (32,032-line CSV miscounted);
  test_loc 1398 -> self-tests are 0 (benchmark harness misclassified); T0 criterion met at review
  (dead >12mo) though T1 was run per dispatcher.
- Shallow clone: single-commit `.git/shallow`; activity conclusion rests on remote API, not HEAD.
