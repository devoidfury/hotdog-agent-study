# Wave 6 - Merge Log (findings.jsonl)

Built 1666 canonical records. Raw inputs: findings 81 files / 758 recs; boundary 31 files / 167 recs; T3 merged 24 runs / 774 recs = 1699 raw.
Reconciles with audit: 1634 unique (audit) + 15 hotdog + 16 post-audit boundary (workground2 2, open-interpreter 5, zeroclaw 4, opensquilla 5, codewhale 0, orca-agent 0) + 1 (opencode-e13 split) = 1666.

## Drops (duplicate ids)
34 drops, all `output/findings/gemini-cli.jsonl` copies (kept: run 20260929-0502-t3-giant-review-3/merged-findings.jsonl, per audit recommendation):

ids dropped: gemini-cli-c1, gemini-cli-c2, gemini-cli-c3, gemini-cli-c4, gemini-cli-c5, gemini-cli-c6, gemini-cli-c7, gemini-cli-c8, gemini-cli-c10, gemini-cli-s1, gemini-cli-s2, gemini-cli-s3, gemini-cli-s4, gemini-cli-s5, gemini-cli-s6, gemini-cli-s7, gemini-cli-s8, gemini-cli-s9, gemini-cli-s10, gemini-cli-s11, gemini-cli-s13, gemini-cli-s14, gemini-cli-e1, gemini-cli-e2, gemini-cli-e3, gemini-cli-e4, gemini-cli-e5, gemini-cli-e6, gemini-cli-e7, gemini-cli-e8, gemini-cli-e9, gemini-cli-e10, gemini-cli-e11, gemini-cli-e12

After the re-ids below, zero cross-source duplicate ids remain (verified programmatically).

## Re-ids
- vtcode (run 20260929-0859-...-2): all 30 records carried concept-slug ids; re-idd every one from `aliases[0]` (e.g. vtcode-c1..c7, vtcode-s1..s11, vtcode-e1..e12). Consumes the alias. Fixes the 2 in-file pairs (sandbox-delegation x2 -> vtcode-s1/s2, phantom-safety-control x2 -> vtcode-s5/c5) and the god-file-loop cross-collision with opensquilla (-> vtcode-c2).
- opensquilla (run 20260929-0859-...-1): the 18 bare `e1..e18` ids re-idd to `opensquilla-e1..e18` from their own `aliases[0]` (schema requires `<subject>-<n>`; aliases already carried the canonical form).
Total re-ids: 48.

## Concept merges
STRONG: 27/27 groups applied -> 64 record rewrites (48 distinct displaced ids). Displaced id placed in `aliases`. Includes rival-harness-session-import (cline-b1, open-interpreter-b5) -> foreign-session-import per the ledger's strong ruling, which supersedes the calibration-notes convergence feed for that pair.

MEDIUM (3 groups, applied only where ledger evidence text supports sameness):
- compaction-hooks APPLIED (compaction-collapse-hooks maki-5 + compaction-hooks-absent codewhale-e3 -> compaction-hooks): presence/absence of the same lifecycle-hook mechanism; kind carries polarity, as elsewhere in the ledger.
- sandbox-absent APPLIED (no-kernel-sandbox goose-s6, no-os-sandbox oh-my-pi-s3, no-sandbox-by-design opencode-s2 -> sandbox-absent): all four texts say plainly 'no OS-level containment'; stance-vs-omission nuance preserved in one_why.
- windows-sandbox-gap REJECTED (medium-rejected): the only merge action would be orca-agent-s12 (windows-appcontainer-sandbox), whose text records a WORKING, tested AppContainer/restricted-token backend and explicitly claims the refuse-rather-than-fallback pattern -- the inverse polarity of a 'gap'. Not same-sense; left separate. The two genuine windows-sandbox-gap occupants (deepseek-reasonix-s7, workground2-s2) are unaffected and correct.

WEAK (6 groups; leave separate unless a record's own text settles it):
- usage-measured-compact-trigger APPLIED (3 records): roo-code-e5 'cumulative reported usage' is literally the concept; opencode-e1 and orca-agent-e2 are measurement-based triggers of the same family -- ledger precedent: integrator already folded deepseek-reasonix dual-boundary-compaction-trigger here.
- compaction-tiering APPLIED (1): opencode-e2 'Two genuine tiers (prune + summarize)' -- a tiering instance by its own words.
- workflow-resume-journal APPLIED (1): grok-build-e10 'journal is durable but the run is not' -- the anti-pattern half of the same resumability concept; kind=anti-pattern carries polarity.
- oauth-client-impersonation group REJECTED for mcp-oauth-client (amazon-q-developer-cli-11): the record text is a neutral capability note ('usable enterprise interop'), not the impersonation threat; ledger's own doubt confirmed. NOTE: the canonical oauth-client-impersonation id already covers claurst + qqcode, and post-audit open-interpreter-b3 joins it (x3 as the boundary notes require) -- no new id needed.
- injection-defense pair REJECTED: codewhale-s6 (structural-injection-defense) describes a specific positive design ('no external-content scanner anywhere'); dexto-s5 (injection-defense-absent) records absence of ALL defense. Absence-of-everything is not the same mechanism; also both stay out of injection-screening per ledger.
- untrusted-input-framing (qwen-code-s7) / untrusted-result-framing (hermes-agent-s7) REJECTED: input vs tool-result trust boundaries; texts do not settle sameness.

DO-NOT-MERGE rulings: all 6 honored -- god-file-loop vs god-file-host-wiring, layered-loop-separation vs layered-loop-breakers, llm-compaction-judge vs llm-permission-judge, permission-policy variants, checkpoint-revert vs checkpoint-failopen-degradation, and the three-way compaction family were not merged anywhere in this pass.

Calibration-notes (post-audit) merges:
- opensquilla-b2: permissive-default-vs-docs -> phantom-safety-control
- default-posture: default-posture -> phantom-safety-control
- mode-vocabulary-drift: mode-vocabulary-drift -> phantom-safety-control
- dexto-s2: permissive-default-vs-docs -> phantom-safety-control
- Family canonical decision: the rung-2/deceptive-default family (permissive-default-vs-docs, opensquilla default-posture + mode-vocabulary-drift folded in by boundary records b2) is canon'd under **phantom-safety-control** -- the established 17/13 id whose occupants are exactly the phantom-protection variants (claurst-3/b1, forge-5, codebuff-8, ra-aid-2, ...). permissive-default-vs-docs survives as an alias.
- workground2-b2 keeps concept cache-impact-ci-gate (already canonical from workground2 T3); its corrected-origin nuance rides in the record. open-interpreter-b1 adds the ONE new canonical concept: rival-harness-wire-emulation. opensquilla-b1..b5 kept their already-canonical concepts (injection-screening, permissive-default-vs-docs->phantom-safety-control, generated-code-loc-census, loop-detection, evals-in-ci).
Total calibration-driven rewrites: 4

## Unrecoverable-record dispositions
- 20 opensquilla slug-id records: concept := id per audit (opensquilla-s2 style records keep their slug ids, e.g. god-file-loop, principal-boundary, idempotent-turn-ingress).
  ids: god-file-loop, signature-sniffing-compat-shim, in-flight-loop-decomposition, idempotent-turn-ingress, session-epoch-fencing, steer-during-run, cancellation-race-hygiene, actionable-errors, stream-resume-gap-detection, test-boundary-policing, kernel-enforcement-depth, default-posture, injection-refusal-dead-code, untrusted-wrapping-coverage, principal-boundary, fail-closed-escalation, verification-corps, supply-chain-ci, mode-vocabulary-drift, kernel-path-test-depth
  (default-posture and mode-vocabulary-drift were then merged by the calibration rule above.)
- Fuzzy best-fit assignments (44 records, every one read against title+evidence):
  - zeroclaw-s1 -> sandbox-request-failopen
  - zeroclaw-s2 -> sandbox-absent
  - zeroclaw-s3 -> injection-screening
  - zeroclaw-s4 -> windows-sandbox-gap
  - zeroclaw-s5 -> grammar-parsed-bash-policy
  - zeroclaw-s6 -> fail-closed-default-decision
  - zeroclaw-s8 -> e2e-runs-against-real-sandbox
  - zeroclaw-s9 -> evals-in-ci
  - zeroclaw-s10 -> quality-gates-unwired
  - zeroclaw-s11 -> side-effect-verified-guardrail-tests
  - zeroclaw-s12 -> test-fixture-loc-inflation
  - zeroclaw-s14 -> security-posture-docs
  - zeroclaw-e1 -> prompt-cache-discipline
  - zeroclaw-e2 -> trim-only-context-window
  - zeroclaw-e3 -> cost-accounting
  - zeroclaw-e4 -> usage-measured-compact-trigger
  - zeroclaw-e5 -> workflow-resume-journal
  - zeroclaw-subagent-policy-containment -> subagent-permission-inheritance
  - zeroclaw-e7 -> scheduled-agent-runs
  - zeroclaw-e8 -> durable-queue
  - zeroclaw-e9 -> acp-server-surface
  - zeroclaw-e10 -> mcp-client-only
  - zeroclaw-e11 -> external-coding-agents-as-tools
  - zeroclaw-e12 -> docs-as-shipped-artifact
  - zeroclaw-e13 -> actionable-errors
  - zeroclaw-e14 -> interop-ceilings
  - opensquilla-e1 -> compaction-tiering
  - opensquilla-e2 -> usage-measured-compact-trigger
  - opensquilla-e3 -> prompt-cache-warming
  - opensquilla-e4 -> evals-in-ci
  - opensquilla-e5 -> compaction-pricing
  - opensquilla-e6 -> context-economy-guardrails
  - opensquilla-e7 -> durable-queue
  - opensquilla-e8 -> crash-recovery-posture
  - opensquilla-e9 -> subagent-budgeting
  - opensquilla-e10 -> durable-queue
  - opensquilla-e11 -> loop-detection
  - opensquilla-e12 -> interop-matrix
  - opensquilla-e13 -> headless-contract
  - opensquilla-e14 -> interop-surface-breadth
  - opensquilla-e15 -> interop-ceilings
  - opensquilla-e16 -> docs-match-code
  - opensquilla-e17 -> onboarding-error-dx
  - opensquilla-e18 -> expired-compat-shim-layer
- Notable confirmations: opensquilla-e11 -> loop-detection and e4 -> evals-in-ci are pinned by boundary records b4/b5; zeroclaw-s10 -> quality-gates-unwired and s14 -> security-posture-docs and s12 -> test-fixture-loc-inflation are pinned by boundary records b2/b4/b3 of the same defect; zeroclaw-s1 -> sandbox-request-failopen mirrors codewhale-s4's shape exactly.
- uncategorized: 0 -- none needed; every record found a defensible existing canonical concept.

## Null fields (not fabricated)
- effort_for_us=null on 86 records: {'bitfun': 13, 'zeroclaw': 26, 'goose': 28, 'opensquilla': 18, 'deepseek-reasonix': 1}. The audit's 59 (bitfun 13, goose 28, opensquilla e 18) plus the 26 zeroclaw severity-shape records (also carry no effort) plus deepseek-reasonix-e8 (literal 'n/a' normalized to null). These stay in the file, excluded from roadmap effort ranking until adjudicated.
- confidence=null on 26 records (all zeroclaw severity-shape: field absent at source).
- one_why: 0 nulls -- roo-code e2..e14 recovered from `anchor` field per rule 10 (11 records).

## Field normalization applied
- severity -> impact (26): high->high, medium->med, low->low; **positive->high flagged for spot-check** (14 recs, adjudicated at synthesis per rule 2; strengths credited as high impact), **info->low** (zeroclaw-s12, census note).
- impact 'pos' -> high flagged: continue-e2, continue-e4, continue-e7, continue-e10 (rule 7).
- impact 'medium'->'med': 39 records; confidence 'medium'->'med': 22 records.
- evidence arrays joined with '; '; detail -> one_why; subject derived from run map (26 zeroclaw).
- kind remaps (kind_original retained on all): strength->unique/portable(adjudicated); gap->anti-pattern; mixed->nuance; risk->safety-hole-if-safety-dim-else-anti-pattern; context->nuance; weakness->anti-pattern; note->nuance; neutral->nuance  [counts: {'strength': 22, 'gap': 9, 'mixed': 2, 'risk': 7, 'context': 1, 'weakness': 2, 'note': 1, 'neutral': 2}]
- strengths adjudicated unique only where the finding is a subject-scale fact (opensquilla verification-corps); all other strengths -> portable (mechanisms with a liftable design).
- Deviation from audit lists: audit tallied opencode kinds 'strength x6, gap x2' and 'zeroclaw risk x25, strength x1'; the actual merged files carry strength x22 (opencode 6, zeroclaw 14, opensquilla 2), gap x9, risk x7, weakness x2, neutral x2, note x1, context x1, mixed x2. All non-enum kinds remapped per the rule's intent with kind_original kept.
- kind=mixed: opencode-e13 SPLIT into opencode-e13a (positive docs breadth, nuance, concept docs-dx-mass) + opencode-e13b (dual-tree confusion, anti-pattern, original concept docs-depth-vs-dual-tree) -- NOTE the audit names 'opencode-e15', which does not exist in the file; e13 is the only mixed-kind opencode record. zeroclaw-e10 (also kind=mixed, not in the audit split list) kept as a single record (nuance): its halves (three-transport MCP client / no server+SDK) are one posture finding; the 'no server/SDK' half rides in one_why.

## Dropped annotation fields (per canonical schema)
Kept only the allowed sidecars: lane (383 recs), merged_from (8), cross_ref (2), aliases, kind_original. Dropped as out-of-schema: lane_kind, lanes, dimension(s), anchor (after one_why fill), canonical_id (all self-refs or provenance), merge_note, alias/concept_alias/concept_aliases/lane_concept (harvested into aliases), cross_references, related, corpus_note, concept_note, also_scored_under (records stay single-counted under their own concept per rule 14: roo-code-c4, roo-code-c8, roo-code-s10), area, claim, notes, detail, severity.

## Totals
- findings.jsonl: 1666 records, all single-line valid JSON (parsed every line to verify), zero duplicate ids, every concept field populated, 104 subjects.
- Distinct concepts after merging: 678 (ledger baseline 712 pre-merge).
