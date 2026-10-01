# Agent study - shared review protocol

Read this fully before reviewing any subject. Output root: `/workspace/experiments/agent-study/output/`.

## Hard constraints

1. **Never execute a subject.** Read-only static review. No builds, no running binaries, no `npm install`, no tests of the subject. Git read commands and cloc/tokei/find/wc are fine.
2. **Shallow clones.** If `.git/shallow` exists in a subject, do NOT infer inactivity from HEAD date; either check remote refs or mark "evidence-limited".
3. **Copying rules.** Pure-concepts porting only (never code) for Elastic / BUSL / FSL / no-license subjects. Suspected leaked-snapshot subjects get a `license-risk` finding and no code-adjacent portables.
4. **Evidence.** Every score claim needs `path/to/file:line`. "README says" is NOT evidence. "CI workflow x runs y on every push" IS evidence.
5. **Return size.** Reviewers reply to the dispatcher with at most 6 lines. Artifacts carry the detail.
6. **Before scoring, answer in one sentence:** "Which anchor subject is this closer to, and why?" Anchors live in `anchors/anchor-excerpts.md`.

## Scoring rubric (each dimension 0-10, weighted sum = 100)

| dimension | weight | what it measures |
|---|---|---|
| architecture | 15 | loop structure, module boundaries, god-file presence, state model |
| verification | 15 | tests proving properties (not existence), CI, fuzzing, evals |
| safety-enforcement | 10 | permissions/sandbox that actually bind; honesty about what they don't |
| token-economy | 10 | compaction quality, prompt-cache discipline, lazy loading, cost visibility |
| orchestration | 10 | subagents/workflows/queues, resumability, budgets, crash recovery |
| interop | 10 | MCP/ACP/headless contract, IDE surfaces, import from rival harnesses |
| operability | 10 | sessions/resume/rewind, configuration, diagnostics, crash behavior |
| originality | 10 | ideas no other subject has (verify real in code, not marketing) |
| durability | 5 | maintenance, bus factor, governance, funding/institution backing |
| docs-dx | 5 | install path, error messages, onboarding, reference docs |

Anchor meanings: 0 absent / 2 present but misleading or broken / 4 works on happy paths, known gaps / 6 solid, one standard design, tested / 8 defended: properties enforced AND tested / 10 field-defining; other subjects should be copied from this one.

Bands: **S** >= 88 AND no dimension < 5 on safety-enforcement or verification; **A** 78-87; **B** 65-77; **C** 45-64; **D** < 45.
Calibration rules: (a) a sync-fork derivative must not outrank its upstream (a divergent fork can, on merit); (b) archived/dead caps at B. Record any demotion in `calibration-notes.md`, never silent.

## Finding schema (one JSON per line)

`{"id":"<subject>-<n>","subject":"...","concept":"<canonical-kebab-concept>","kind":"unique|portable|nuance|anti-pattern|safety-hole|license-risk","title":"...","evidence":"path/to/file:line or commit","impact":"high|med|low","effort_for_us":"S|M|L|XL","confidence":"high|med|low","one_why":"..."}`

Concept discipline: before coining a concept id, grep `findings/` for close matches. Seeded canonical ids:
`compaction-tiering`, `cache-monotonic-compaction`, `loop-detection`, `permission-policy`,
`sandbox-delegation`, `hook-trust-scoping`, `lazy-skill-loading`, `memory-files`, `evals-in-ci`,
`faux-provider-testing`, `checkpoint-revert`, `workflow-resume-journal`.
Convergence counts depend on this discipline; when in doubt reuse an existing id and note the nuance in `one_why`.

## Reviewer outputs (per subject)

- `subjects/<name>.md` - the report (T2: must show you read core loop + compaction + permission code, cite lines)
- `scores/<name>.json` - `{"subject", "tier", "lane_scores": {all ten dimensions}, "weighted_total", "band", "closest_anchor", "strongest_dimension", "weakest_dimension", "provenance", "calibration_notes"}`
- `findings/<name>.jsonl` - findings per schema above

## Tier expectations (assigned from census.json, not vibes)

- **T0 triage** (dead >12mo at HEAD, derivative with no delta, or <5k LOC): 10 min, half-page report, scores present so the tier list is complete. Sanity-check the census flag before accepting it.
- **T1 standard** (5k-100k LOC): 30-40 min full-rubric review; read the actual loop, permission, and compaction code, not just docs.
- **T2 deep** (100k-400k LOC, or plausible-S candidate): 60-90 min; mandatory line-level read of core loop + compaction + permission code.
- **T3 coordinated** (>400k LOC or top-15 candidate): run via the `t3-giant-review` workflow, not this protocol.

## Score distribution sanity

If more than 25% of scored subjects land at 80+, calibration was lost: stop and re-anchor. Score against the anchors, not against goodwill.
