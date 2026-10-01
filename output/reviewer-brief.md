# Reviewer brief (T1/T2 single-subject reviews)

You are one reviewer in the coding-agent field study. Your dispatcher will name ONE subject,
its directory under /data/samples/agents/, and whether it is a T1 (30-40 min) or T2 (60-90 min) review.

Before anything else read:
1. /workspace/experiments/agent-study/output/review-protocol.md  (rubric, anchors, finding schema, hard constraints)
2. /workspace/experiments/agent-study/output/anchors/anchor-excerpts.md  (frozen anchor ladder)
3. The subject's row in /workspace/experiments/agent-study/output/census.json

Method:
- Read-only static review. NEVER execute the subject, install it, or run its tests.
- T2: line-level mandatory read of the agent core loop, compaction/context management, and
  permission/sandbox code, plus a real look at tests (do they prove properties or just
  existence?), CI workflows, orchestration, interop, docs. T1: same surface, shallower.
- Before scoring, answer in one sentence: which anchor subject is this closer to, and why?
- Score all ten dimensions 0-10 against the anchor ladder. Every claim needs path/file:line
  evidence inside the subject. README/CLAUDE.md marketing claims are NOT evidence.
- Sanity-check the census row: census LOC has been wrong (test-file glob misses, vendored
  blobs, shallow clones). Shallow clone => do not assert dead/low-activity from HEAD alone.
- Provenance: if census says fork, verify the delta yourself; calibration rule (a): a
  sync-fork derivative must not outrank its upstream. Divergent forks score on their merits.
- License discipline: Elastic/BUSL/FSL/AGPL/GPL/no-license => findings may describe concepts
  only, effort_for_us values assume clean-room, never recommend code copying. Suspected
  leaked snapshot => license-risk finding, no code-adjacent portables.

Artifacts (write all three, then stop):
- output/subjects/<name>.md -- the report; include the anchor-question sentence, per-dimension
  scores each with its best evidence lines, strongest/weakest dimension named explicitly
- output/scores/<name>.json -- {"subject","tier","lane_scores":{architecture,verification,
  safety_enforcement,token_economy,orchestration,interop,operability,originality,durability,
  docs_dx},"weighted_total","band","closest_anchor","strongest_dimension","weakest_dimension",
  "provenance","calibration_notes"}
- output/findings/<name>.jsonl -- findings per protocol schema, ids "<name>-1" upward.
  Before coining a concept id, grep output/findings/ for close matches; reuse seeded ids
  (compaction-tiering, cache-monotonic-compaction, loop-detection, permission-policy,
  sandbox-delegation, hook-trust-scoping, lazy-skill-loading, memory-files, evals-in-ci,
  faux-provider-testing, checkpoint-revert, workflow-resume-journal). Zero findings allowed; filler is not.

Reply to dispatcher, max 6 lines:
name=weighted_total (band) | closest anchor + why (one clause) | strongest/weakest dimension |
any census correction | any boundary-risk note (within 2 pts of 88/78/65/45).

## Appendix (added mid-study): identity + activity hygiene

- Identity confusion risk in this corpus: claw-code vs claw-code-agent, kimi-code vs kimi-cli,
  grok-build vs grok-cli, forge vs forge-norvialabs, mini-kode vs mocode vs minicode vs
  nanocoder, code vs codex vs codex-infinity. Work only from THIS subject's directory and its
  manifest; never import facts from a similarly named project. If two subjects might be forks of
  each other, say so in provenance and let synthesis do the rule (a) check.
- If census head_date is >10 months old, remote-verify activity before any archived/dead claim;
  shallow clones hide history in both directions.
- Fork subjects: state the delta vs upstream with file:line regardless of tier; synthesis runs
  rule (a) centrally.
