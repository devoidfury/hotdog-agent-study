# Retrospective: running a 103-subject coding-agent field study

Dates: 2026-09-28 to 2026-10-01. One dispatcher (hotdog), no human in the loop except
for one infrastructure intervention mid-synthesis (see "What broke").

## Scale, as recorded

Numbers from run dirs, the task ledger, and wave6/source-inventory.md; approximations
are labeled.

| work type | count | notes |
|---|---|---|
| census rows | 103 | mechanical phase, zero reviewer time |
| single-reviewer subject reviews | ~79 | T0 triage (8), T1 (~34), T2 (~30), incl. the 6 Phase-1 anchors |
| T3 workflow runs | 24 | one saved graph (`t3-giant-review`), reused; ~9 subjects had 2 giants running concurrently on lanes |
| T3 node runs | ~120 | scout + 3 lane reviewers + integrator per run |
| integrator failures | 3 | 2 killed at max-runtime (resumed, succeeded); 1 (gemini-cli) wrote complete data but no verdict and was adopted only after manual verification |
| boundary re-reviews | 25 | fresh independent reviewer near any band edge; 9 exact decimal matches |
| synthesis/QA workers | 12 | findings audit, merge, convergence, tier list, 3 verification-9 adjudication passes, final coherence audit, table re-sort, 2 dead roadmap workers |
| self-review | 1 | hotdog, scored last, unadjusted |
| total agent sessions | ~240 | across 4 days |
| findings merged | 1,666 | from 129 sources; 712 raw concepts consolidated to canonical ids |

Cost sanity: a workflow run costs 5 agent sessions but produces a lane-parallel read of
a >400k-LOC codebase in roughly the wall-clock of one careful human review. The 24 T3
runs are about half the agent budget and cover the half of the corpus where superficial
reading would have been worst.

## What broke

1. The interruption. Two T3 integrators hit their 45-minute runtime cap and the runs
   died mid-merge, then the whole dispatch session was lost. Recovery was the study's
   best engineering decision: resuming the same run dirs with the same run ids reused
   every filesystem-verified node. Zero review work was lost. The lane artifacts
   (map.md, f-*.jsonl, r-*.md) were exactly the durable intermediate the design
   intended. Total cost of the interruption: re-reading state, two integrator retries.

2. Long-document generation is a real failure mode, and it killed the same deliverable
   three times. The roadmap report was composed by workers as one enormous tool-call
   payload; generation stalled mid-payload, the stream idle-timeout killed the request,
   and -- the ugly part -- the task then reported "completed" with nothing on disk.
   Two workers died identically before the pattern was visible in the provider logs
   (0 tokens out, death at the timeout line, retry loop). The fix was procedural, not
   model-level: compute into a small intermediate artifact first, then emit the report
   as sequential appends of at most ~55 lines, then read back and verify. Third worker
   shipped it in one pass. Filed in the calibration ledger as a first-party finding:
   our delegated-task path has no artifact gate (the workflow engine's verdict-file +
   freshness machinery exists precisely for this, and plain delegation lacked it). The
   study about orchestration failure modes failing by orchestration failure mode is
   the finding we did not plan and trust the most.

3. Manager-level stream timeouts compounded during synthesis: a subagent busted the
   prompt cache on a hotdog session, generation overran the idle timeout, context
   reload looped until a human raised the timeout to 340s. Lesson recorded: long
   synthesis payloads belong in files, not in conversation, and chunked writes shrink
   exactly the payloads that trigger this.

4. Data hygiene errors the reviewers caught that the mechanical phase could not:
   census test-LOC wrong by 5.6x on multiple subjects (test-glob misses), inline
   #[cfg(test)] code invisible to cloc (543k LOC on one subject), website doc trees
   counted as tests (aider's claimed 76k), checked-in blobs inflating LOC, a shipped
   upstream mirror double-counting another subject's LOC, and one stated weighted
   total that did not match its own lane scores (goose, 73 vs 74.5 -- a sum error that
   survived two review passes and died only in the final audit). Trust the census for
   routing, never for facts.

5. Provenance was subtler than the detector. The rename-clone heuristic keyed on
   manifest-name equality and could not tell a deceptive rebrand from an
   attribution-preserving honest fork; it took a policy cap, a reversal, and a
   from-scratch boundary review to settle open-interpreter (78.5 triage "D" -> 83.0
   A). A T3 integrator assumed a CI mechanism was inherited from an upstream and was
   wrong -- the boundary reviewer grepped the upstream tree, found zero hits, and
   moved the invention back to the fork. Provenance claims are findings, not
   metadata.

## What held up

- Anchors first, everything else after. Six frozen scores, errata appended when found,
  every reviewer required to answer "which anchor is this closer to, and why" before
  scoring. The payoff is the exact-match tally: 9 of 25 boundary re-reviews reproduced
  the original total to the decimal, and the 7 band flips were all evidence-argued,
  never vibes. Independent reviewers converge when they share a ladder.

- Rules with a "never silent" clause. Both calibration rules (sync-forks not above
  upstream; dead caps at B) were applied, contested, reversed once on the merits, and
  every application logged. The reversal chain on open-interpreter (demote -> reverse
  -> re-review -> confirm) is in the ledger with reasons at each step. That ledger is
  what makes the tier list defensible against someone who disagrees with a call.

- Doctrine emerging from collisions instead of being invented upfront. The safety
  default-posture doctrine had an explicit collision (kilocode's 7 vs nanocoder's
  below-6 precedent) and the resolution became a mechanical three-gate rule
  (fail-closed; blocking-CI-tested; non-deceptive) that was then applied retroactively
  and prospectively, including to ourselves. The verification-9 definition hardened
  over five rulings ("continue-on-error is telemetry", "canned replay is not an
  eval", a gate that "exits 0 on confirmed regression") and ended as a corpus-wide
  sweep that demoted two A-band 9s. Bands did not move; the definition did, and it
  moved in public.

- Explicit disagreement sections in the T3 integrators. Reviewers scored disjoint
  dimensions, but their narratives conflicted constantly, and the integrators were
  required to resolve in prose. That prose is what let the final audit re-derive
  every claim 100 subjects later.

- Concept-id discipline. Requiring reviewers to grep existing concept ids before
  coining one is what makes 1,666 findings answer "how many subjects did X": 161
  concepts appear in 2+ subjects. The merge ledger kept 27 strong merges, and -- more
  important than the merges -- two principled rejections, including declining a merge
  because one record's polarity was the inverse of the other (a working Windows
  sandbox vs a missing one; merging them would have silently poisoned a count).

## What we would change

- Triage scores should be flagged "placement only" in every downstream artifact. Both
  T0 triage rows that got boundary treatment (open-interpreter, claw-code-agent) moved
  multiple bands under a real review. 10 minutes finds out which tier to worry about;
  it does not find the score.

- One review pass, one arithmetic check. The goose sum error and two band-string
  staleness bugs survived authoring and survived a boundary pass because nobody
  recomputed lane-scores-to-total mechanically until the end. That check belongs in
  the acceptance gate: score files should not land unless the weighted total
  recomputes. Cheap, deterministic, catches what no reviewer catches.

- Boundary reviews should run a dedup grep only after scoring. Two reviewers were
  contaminated by reading prior finding titles during dedup checks; both fenced
  themselves and disclosed, and the norm became explicit, but the ordering should
  have been designed in.

- The delegate path needs the workflow engine's artifact gate. Verdict files and
  output-freshness checks are mandatory in workflows and absent in delegation. We got
  two hollow green completions this study because of it. This is now the sharpest
  argument in our own roadmap for the machine-checked-orchestration thesis: we watched
  a worker announce intent, stall, and exit "completed" without an artifact -- on the
  harness writing this sentence.

## What the corpus itself taught, briefly

The field converged on a floor: nearly everyone has a loop, compaction, MCP, and some
approvals (permission-policy: 61 subjects; faux-provider testing: 47; compaction
tiering: 43). The A-band is not a different species of feature set; it is enforcement
that survives its own defaults, verification that cannot stay green when behavior
regresses, and honest writing about both. The single most common way a promising
harness lost a dimension was a gap between what it documented and what its shipped
default did -- the deceptive-default family is the largest anti-pattern cluster in
1,666 findings. aider landing in C despite 199 contributors remains the exhibit: this
rubric measures the 2026 artifact, not the history. And codex alone clears 88, which
is either the rubric being hard or the field being wide and shallow; the evidence
(127 small crates, cache-key-gated reuse, sandbox bundled into the distribution) says
the former.

## Hotdog, looking back at itself

Scored last, anchors seen, no advocacy: 68.5 B. The two hardest lines in the study are
in our own report -- safety-enforcement 5 because the best-tested approval gate in the
corpus ships disabled, and the observation that our workflow engine's gates are our
best originality claim while our delegated-task path lacks them. The roadmap's
three-move program (flip approvals on, test the real binary with blocking gates, cache
discipline with property tests, ~+3.5 to 72.0, zero new dependencies) is credible
precisely because the safety-5 wound is scored by the study's own doctrine rather than
argued around. Self-review with a frozen ladder is uncomfortable and it is the only
version worth publishing.

## One-line summary

~240 agent sessions, 4 days, 103 subjects, 1,666 findings, 3 machine failures and 3
worker deaths, zero review work lost -- the design's claim was that machine-checked
gates and frozen anchors make distributed review trustworthy, and the study broke often
enough to actually test that claim. It held.
