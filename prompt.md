# Task: Open-source coding-agent field study

As a stress test of our own new graph workflows, and also an interesting result to publish for other researchers, we're going to conduct a study across ~100 of the most popular agent harnesses.

Corpus: /data/samples/agents/ (one directory per subject).

Our harness: hotdog at /workspace (read its AGENTS.md first; it gets scored with the same rubric).

Deliverables, all under /workspace/experiments/agent-study/output/:
- `subjects/<name>.md` + `scores/<name>.json` (schema below) for every subject
- `findings.jsonl` -- one JSON record per finding across the whole study
- `tier-list.md` -- calibrated ranking with drift notes
- `convergence.md` -- concept clusters with exact project counts and per-concept top-3 designs
- `roadmap-for-hotdog.md` -- impact/effort ranked, every item citing findings by id

## Phase 0 -- corpus census and provenance (mechanical, before any review)
For every subject produce one row: name, language, non-test LOC (cloc/tokei or find+wc),
test LOC, contributor count + commit date of HEAD, `git log --oneline | wc -l`,
whether `.git/shallow` exists (history is truncated), license, manifest name (catch rename-clones:
package.json/module name pointing elsewhere). Run a cheap cross-corpus provenance scan: hash shared
distinctive files, compare prompt corpora and built-in agent-name strings between subjects and against known upstreams
(claude-code, codex, opencode, pi, cline/roo, trae-agent, gemini-cli). Output `census.json`. No opinions yet.

## Review tiers (assign from census, not from vibes)
- **T0 triage** (dead >12 months at HEAD, derivative with no delta, or <5k LOC):
  10 min, half-page report, scored only so the tier list is complete.
- **T1 standard** (5k-100k LOC): one reviewer, 30-40 min.
- **T2 deep** (100k-400k LOC, or any subject that could plausibly make S): one reviewer,
  60-90 min, mandatory read of at least core loop + compaction + permission code, not just docs.
- **T3 coordinated** (>400k LOC or top-15 candidate: think cline, hermes-agent, continue, codex, cline-family giants):
  a 3-reviewer graph per subject -- see graph shape.

## Scoring (every dimension 0-10, weighted sum = 100)
| dimension | weight | what it measures |
|---|---|---|
| architecture | 15 | loop structure, module boundaries, god-file presence, state model |
| verification | 15 | tests proving properties (not existence), CI, fuzzing, evals |
| safety-enforcement | 10 | permissions/sandbox that actually bind; honesty about what they don't |
| token-economy | 10 | compaction quality, prompt-cache discipline, lazy loading, cost visibility |
| orchestration | 10 | subagents/workflows/queues, resumability, budgets, crash recovery |
| interop | 10 | MCP/ACP/headless contract, IDE surfaces, import from rival harnesses |
| operability | 10 | sessions/resume/rewind, configuration, diagnostics, crash behavior |
| originality | 10 | ideas no other subject has (verify it's real, not marketing) |
| durability | 5 | maintenance, bus factor, governance, funding/institution backing |
| docs-dx | 5 | install path, error messages, onboarding, reference docs |

Anchor meanings:
- 0 absent / 2 present but misleading or broken / 4 works on happy paths, known gaps
- 6 solid, one standard design, tested / 8 defended: properties enforced AND tested
- 10 field-defining; other subjects should be copied from this one

Tier bands from weighted total: S >= 88 AND no dimension < 5 on safety-enforcement or verification;
A 78-87; B 65-77; C 45-64; D < 45.

Rubric total decides tier, except:
(a) a sync-fork derivative should not outrank its upstream; a divergent fork can, on merit;
(b) archived/dead caps at B -- record the demotion in `calibration-notes.md`, never silent.

## Finding taxonomy (every finding is one line in findings.jsonl)
{"id":"<subject>-<n>","subject":"...","concept":"<canonical-kebab-concept>",
 "kind":"unique|portable|nuance|anti-pattern|safety-hole|license-risk",
 "title":"...","evidence":"path/to/file:line or commit","impact":"high|med|low",
 "effort_for_us":"S|M|L|XL","confidence":"high|med|low","one_why":"..."}
Canonical concept discipline: before creating a concept id, grep findings.jsonl for close
matches (a seeded list exists after Phase 1: compaction-tiering, cache-monotonic-compaction,
loop-detection, permission-policy, sandbox-delegation, hook-trust-scoping, lazy-skill-loading,
memory-files, evals-in-ci, faux-provider-testing, checkpoint-revert, workflow-resume-journal...).
Convergence counts are only as good as this id discipline.

## Anchor calibration (Phase 1, before mass review)
Have one strong reviewer score six spread subjects (one suspected-S, two A, one B, one C, one D)
strictly against the anchors. Freeze their reports as the reference set; attach the
relevant anchor excerpts to every subsequent reviewer's prompt. Before scoring, reviewer
answers one question: "which anchor subject is this closer to, and why?"

## T3 graph shape (per giant subject; reuse as workflow_dispatch YAML)
nodes:
- scout: repo map, area split with file lists, provenance verdict, top-risk areas. Output map.md.
- reviewer-core (depends scout): architecture + agent loop + state/context model. Output f-core.jsonl + r-core.md.
- reviewer-safety (depends scout): permissions, sandbox, injection defense, test/verification culture, crash behavior. Output f-safety.jsonl + r-safety.md.
- reviewer-economy (depends scout): compaction, cache discipline, orchestration/subagents, interop, ecosystem, docs. Output f-econ.jsonl + r-econ.md.
- integrator (depends all three): merge findings (dedupe by concept), resolve score disagreements between the three reviewers explicitly,
  produce final subjects/<name>.md + scores/<name>.json. Verdict must name the strongest and weakest dimensions with evidence.
Reviewers must not score the whole subject outside their lane; integrator owns the total.

## Constraints and traps
1. Never execute a subject. Read-only static review.
2. Shallow clones hide history: if `.git/shallow`, do not infer inactivity; check remote or say evidence-limited.
3. Copying allowed by pure concepts only, never by code, for Elastic/BUSL/FSL/no-license subjects; leaked-snapshot subjects get license-risk findings and no code-adjacent portables.
4. Every score claim needs file:line evidence; "README says" is not evidence, "CI workflow x runs y on every push" is.
5. Reviewers return <= 6 lines per subject; artifacts carry the detail.
6. Score distribution sanity: if >25% of subjects score 80+, calibration was lost; re-anchor.
7. hotdog is scored last, same rubric, by reviewers who have seen the anchors, with its self-profile as input, not as truth.

## Synthesis phases
- convergence.md: group findings.jsonl by concept; per concept: count of subjects, best 3 implementations with evidence,
  the design tension if implementations contradict (e.g. recall-instead-of-compact vs compact-harder), and hotdog's current position.
- tier-list.md: rubric totals + calibration rules + boundary re-review (subjects within 2 points of a tier band get one fresh reviewer before the band is final).
- roadmap-for-hotdog.md: rank only from findings with kind=portable|safety-hole, weighted by concept convergence count x our-thesis fit
  (zero-dep, solo-maintainer, user-paid tokens, machine-checked orchestration); include an explicit non-moves list with data for each.
