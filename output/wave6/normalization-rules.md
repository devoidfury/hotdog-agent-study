# Wave 6 - Normalization Rules (findings)

Canonical schema: `{id, subject, concept, kind(unique|portable|nuance|anti-pattern|safety-hole|license-risk), title, evidence, impact(high|med|low), effort_for_us(S|M|L|XL), confidence(high|med|low), one_why}`

## Record-shape census

| source family | records | already-canonical | with variants/defects |
|---|---|---|---|
| findings | 743 | 724 | 19 |
| boundary | 151 | 151 | 0 |
| t3-merged | 774 | 247 | 527 |

Extra/variant field frequency (all sources): `lane` x383, `aliases` x309, `lane_kind` x178, `dimension` x133, `lanes` x68, `anchor` x38, `canonical_id` x28, `detail` x26, `severity` x26, `kind_original` x23, `merge_note` x18, `alias` x13, `merged_from` x10, `concept_alias` x9, `lane_concept` x7, `cross_references` x4, `corpus_note` x4, `related` x3, `concept_aliases` x3, `also_scored_under` x3, `dimensions` x2, `cross_ref` x2, `concept_note` x1

## Field-rename / repair rules (apply in order)

1. **`claim` -> `title`** (orca-agent run f-safety.jsonl lane shape; already normalized by the integrator before merged, keep the rule for lane files).
2. **`severity` -> `impact`** with value map `high->high, medium->med, med->med, low->low`; value `positive` cannot map (2 zeroclaw records) -> set impact at synthesis adjudication, flag.
3. **`notes` -> `one_why`** (orca-agent lane shape); where an `area` field is also present, fold as prefix: `one_why = "[" + area + "] " + one_why`. (orca's merged records are already canonical; this rule applies to lane files if ever re-ingested.)
4. **`detail` -> `one_why`** (zeroclaw run shape, 26 records).
5. **`evidence` array -> string**: join elements with `"; "` (26 records, all zeroclaw shape).
6. **missing `subject`**: derive from the T3 run dir subject map (see source-inventory.md); applies to 26 zeroclaw safety/econ-shape records.
7. **enum value normalization**: `impact`/`confidence` `"medium" -> "med"` (gemini-cli x8, grok-build x7, opencode x1, oh-my-pi x4, bitfun x1, mistral-vibe x1 -- incl. copies in both findings/ and merged). `impact "pos"` (continue x4) is not a valid value: these are positive highlights -> map to `high` and flag for spot-check.
8. **`kind` remaps (adjudicate, keep `kind_original`)**: `"strength" -> unique-or-portable` (opencode x6, zeroclaw x1), `"gap" -> anti-pattern` (opencode x2), `"risk" -> safety-hole-if-dimension-safety-else-anti-pattern` (zeroclaw x25), `"mixed" -> split-record` (opencode x1: opencode-e15 carries both a strength and a gap half and cannot be one canonical record).
9. **missing `effort_for_us`** (59 records: bitfun run e1-e13 x13, goose merged x28, opensquilla e1-e18 x18): no value is mechanically derivable. Set `effort_for_us=?` and exclude from roadmap impact/effort ranking until adjudicated at synthesis.
10. **missing `one_why`** (roo-code e2-e14 x11): records carry an `anchor` field instead; set `one_why = anchor` text, mark `confidence` unchanged, flag low-fidelity.
11. **missing `concept`** (64 records): opensquilla slug-id records (x20, e.g. id=`god-file-loop`): set `concept := id` (recoverable). opensquilla `e1`-`e18` (x18) and zeroclaw severity-shape (x26): **unrecoverable** -- concept must be assigned by a human reading title/evidence; these records are absent from concept-ledger.md counts.
12. **annotation pass-through** (do not drop, sidecar them): `lane`/`lanes`/`lane_kind` (lane attribution), `dimension(s)` (reviewer dimension tag), `merged_from` (lane-record provenance for dedupe), `merge_note` (integrator note), `canonical_id`/`related`/`cross_references`/`concept_note`/`corpus_note` (cross-links), `kind_original` (pre-remap kind), `anchor` (closest-anchor reference).
13. **alias harvesting**: `alias`/`aliases`/`concept_alias`/`concept_aliases` values feed the concept alias ledger (see concept-ledger.md); never let an alias mint a second convergence bucket.
14. **`also_scored_under`** (roo-code x3): the finding is counted under another concept; at synthesis count it ONCE, under the referring record's concept.
15. **`lane_concept` -> alias for the ledger, never a concept**: 7 mistral-vibe records carry the lane-coined id alongside the canonicalized `concept` (e.g. lane-coined `cache-aware-compaction`, folded by the integrator). `concept` is canonical; `lane_concept` joins the alias ledger. One lane_concept value embeds a prose note ("no-loop-detection (lane kind was gap; kind fixed to anti-pattern per protocol enum)") -- strip the parenthetical when harvesting the alias.
16. **`cross_ref`** (mistral-vibe x2): annotation, pass through.
17. **structured `aliases` entries**: some aliases are objects, not strings. `{concept, coined_by, merged_into}` (workground2, deepseek-reasonix, 16 entries) is an integrator-resolved lane->canonical merge: harvest `concept` -> `merged_into` into the alias ledger. `{id, lane, concept}` (opencode-m1) is lane-record provenance: keep as sidecar.

## ID collision fixes

- vtcode merged file uses concept slugs as ids: `sandbox-delegation` (lines 7 and 9), `phantom-safety-control` (lines 12 and 28) collide **within** the file, and `god-file-loop` (line 2) collides with opensquilla's line 1 record. Fix: re-id from each record's `aliases[0]` (e.g. `vtcode-s1`, `vtcode-s2`, `vtcode-s5`, `vtcode-c5`, `vtcode-c2`); these are distinct findings, not duplicates.
- gemini-cli: 34 identical ids across `output/findings/gemini-cli.jsonl` and run 0502-3 merged. Drop one copy (recommend dropping the `output/findings/` copy).
- After rules above, no duplicate ids remain across included sources (37 raw cross-source duplicate ids = the 34 gemini copies + 3 collisions; 3 raw within-source duplicates = the vtcode pairs).

## Unrecoverable / needs-adjudication record list

| record(s) | source | problem | disposition |
|---|---|---|---|
| zeroclaw severity-shape records: s1..s24, e14, subagent-policy-containment (26 records; crash-recovery-middleware is canonical + annotations, NOT in this group) | run 20260929-0854 | no concept/subject/impact/effort/confidence/one_why; kind=risk x25, strength x1; severity=positive x2; evidence array | normalize fields 2,4,5,6,8; concept needs human assignment |
| opensquilla e1..e18 | run 20260929-0859 | no concept, no effort_for_us | concept + effort need human assignment |
| opencode-e15 | run 20260929-0502-4 | kind=mixed (two findings in one record) | split at synthesis |
| continue-e2,e4,e7,e10 | run 20260929-0849-3 | impact=pos | map to high, spot-check |
| bitfun e1..e13 | run 20260929-0849 | no effort_for_us | adjudicate effort |
| goose merged (all except the -m1s) | run 20260929-0856 | no effort_for_us (28 records) | adjudicate effort |
| roo-code e2..e14 | run 20260929-0901 | no one_why | fill from anchor field |
| gemini-cli findings/ copy (34 records) | output/findings | duplicate ids with run 0502-3 | drop one copy |
