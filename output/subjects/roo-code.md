# roo-code - T3 integrated review (run 20260929-0901-t3-giant-review)

**Subject:** Roo Code (`RooCodeInc/Roo-Code`), TypeScript monorepo (pnpm+turbo), VS Code extension + headless CLI. Census: 234,539 non-test / 140,070 test LOC; shallow clone (1 commit, HEAD 2026-05-15).
**Provenance:** divergent lineage fork of Cline, not a sync-fork and not "original" as the census flagged. Evidence: extension package still named `"roo-cline"` (`src/package.json:2`, publisher RooVeterinaryInc), flagship upstream class retained (`src/core/webview/ClineProvider.ts`, 3,149 LOC), core message type is `ClineMessage` (`packages/types/src/message.ts:276`), legacy `.clinerules*` still protected (`RooProtectedController.ts:20`), README:68 acknowledges origin. Forked pre-SDK cline (the cline anchor's `sdk/packages/agents` tree is absent); divergence is real: monorepo restructure, `apps/cli` + `packages/{core,ipc,types,vscode-shim}`, 37 providers, condense/context split, skills, orchestrator mode.
**Closest anchor:** cline (76.5) - by ancestry and design shape.

## Scores

| dimension | weight | score | one-line basis |
|---|---|---|---|
| architecture | 15 | 6 | 4,619-LOC fused `Task.ts` loop+state+persistence+checkpoints+condense; free-function half mutating Task fields; webview god-pair 3,230+3,149; CLI re-infers loop state from presentation-stream heuristics (c1, c3) - credit for shim-hosted single engine (c2) |
| verification | 15 | 7 | 460 spec/test files + ubuntu+windows CI + weekly CodeQL, but the composed approval router has zero specs (s9), e2e suite unwired from all 7 workflows (s8), no evals/fuzzing/coverage gate |
| safety-enforcement | 10 | 7 | fail-closed approval router in-loop + three tested enforcement layers above the cline 6-rung (shell-substitution demotion s2, config self-protection s3, realpath .rooignore s4); nothing kernel-bound, skill-tool injection gap (s6), terminal ignore check bypassable (s5) |
| token-economy | 10 | 8 | tested MultiPointStrategy cache-placement layer where the cline anchor has none (e3), non-destructive rewind-safe condense (c4), tree-sitter folded-file context in summaries (e2); docked for Bedrock-only consumption (e4) and reported-usage trigger (e5) |
| orchestration | 10 | 7 | careful boomerang delegation with flush-guards and repair specs (e6), loop breaker + demonstrated resume/crash (s10, c8); in-memory 98-LOC queue, no journal/cron/budgets (e7) |
| interop | 10 | 7 | typed NDJSON headless contract stronger than cline's CLI (e11), MCP client 3 transports (e10); two surfaces, types-only publish, no ACP/MCP-server (e12) |
| operability | 10 | 8 | shadow-git per-task checkpoints with GC + metrics-compensated rewind, locked/backup/rollback JSON writes with self-reconciling index (c6), crash posture with flush-before-exit (c8); not 9: checkpoints fail open to permanently-disabled on any transient error - 11 disable sites (c7), no journal/post-mortem tooling |
| originality | 10 | 7 | integrator-scored; see below |
| durability | 5 | 4 | integrator-scored; see below |
| docs-dx | 5 | 8 | 510-file docs tree matching live mechanisms, 18 locales, actionable errors (e13); docked: stream-json schema undocumented (e14), 5-line AGENTS.md, three overlapping `src/core/context*` dirs |

**Weighted total: 69.5 -> band B (65-77).** S-gate n/a (band B regardless; safety-enforcement 7 and verification 7 both clear the >=5 floor).

## Integrator-scored dimensions

### Originality: 7 (weight 10)

Verified in code, not marketing. I opened each flagship mechanism in the checkout:
- `src/core/condense/foldedFileContext.ts` - real tree-sitter signature folding (`parseSourceCodeDefinitionsForFile` import at :2, `maxCharacters?: 50000` default documented :36, error-string guards for tree-sitter failures), wired into the condense summary path; no equivalent in cline/pi/crush anchors. Not vapor.
- `src/api/transform/cache-strategy/multi-point-strategy.ts` - `previousPlacements` cross-turn keep/combine logic at :73-134, `requiredPercentageIncrease = 1.2` verified at :158; 1,112-LOC spec exists. Genuine placement-stability discipline, but consumed only by bedrock (`bedrock.ts:852-863` is the sole `previousPlacements` persistence site, matching e4).
- `src/core/condense/index.ts:459` "NON-DESTRUCTIVE CONDENSE" tagging comment verified; the 567-LOC `rewind-after-condense.spec.ts` exists - the property is enforced, not advertised.
- `src/core/auto-approval/commands.ts:22-58` - the bash/zsh substitution classes (`${var@P}`, `${!v}`, `=(...)`, `*(e:...:)`, `<<<$(...)`) are individually implemented and commented, with a two-sided spec (bypass + false-positive regressions). An anti-bypass layer cline lacks.
- `packages/vscode-shim` is a real package (api/classes/storage/interfaces) hosting the extension headlessly (c2) - an unusual, honest answer to multi-surface reuse.

Docked from 8+: the loop itself, checkpoint model, persistence shape and provider layer are inherited cline architecture; the "whole dev team of AI agents" framing (repo description) maps to a sequential single-active-child stack (e6), i.e. marketing runs slightly ahead of the mechanism; and two of the four flagship novelties carry ceilings (cache strategy siloed to one provider, condense still LLM-summarize-only). Six concepts coined as new against the study corpus (shim-hosted-engine-reuse, last-message-state-inference, rewind-safe-compaction, single-mutation-entry-point, crash-safe-json-persistence, checkpoint-failopen-degradation) corroborate real divergence from the anchor set.

### Durability: 4 (weight 5)

Census metadata is clone-limited (1 commit / 1 contributor - artifact per map risk 6, not evidence), so I used in-repo release artifacts + one external remote check (read-only; never executed the subject):
- **In-repo cadence:** 76 release artifacts in `releases/`, CHANGELOG through 3.53.0, changesets + `changeset-release.yml` + `marketplace-publish.yml` + `nightly-publish.yml` + `cli-release.yml` - at snapshot time this was a weekly-or-better release machine.
- **License:** Apache-2.0 file present, matches upstream cline's license; no license risk.
- **Institutional backing:** organization-owned (`RooCodeInc`), homepage roocode.com, 24,290 stars / 3,423 forks network.
- **The kill shot:** GitHub API reports `archived: true` for RooCodeInc/Roo-Code (observed 2026-09-30); `pushed_at: 2026-05-15` equals the census HEAD commit, i.e. the snapshot captured the final commit and the repo has been dead ~4.5 months. The workflow rule "archived/dead caps at B" is triggered - see calibration notes. A 3.53.x org-backed fork with a large install base is recoverable by the community (Apache-2.0, fork network), which keeps this above 2, but project backing is now historical, not live: 4.

## Score-disagreement resolution (three reviewers -> one)

No two reviewers scored the same dimension, so there were no head-to-head numeric conflicts to split; the disagreements that existed were evidence-territory and framing conflicts, resolved explicitly:

1. **Crash-handling evidence, claimed by two lanes (c8 vs e8).** r-safety deliberately scored the CLI crash posture "not docked... operability lane owns the rest" while r-core counted it toward operability 8 and r-econ counted the same `run.ts:409-491` handlers toward orchestration 7. Ruling: single credit under operability (merged record c8); orchestration 7 keeps only the resume-substrate consequence (lock-protected JSON, `--resume`), not the handler quality itself. No inflation, no double-count.
2. **Is roo above or below the cline anchor?** All three lanes said "closer to cline" but with opposing drags: econ ("cleanly exceeds cline on token-economy"), core ("architecture below cline, toward crush"), safety ("a notch above cline's rung-6 evidence on enforcement, below it on the approval-matrix test"). Ruling: keep all three partial verdicts - they are dimension-local, not a total conflict. The total (69.5) lands below cline's 76.5 because the two 15-weight dimensions split cline-like (verification 7) and sub-cline (architecture 6), and the token-economy surplus (8 vs cline's 7-ish rung) cannot offset 2 weights' worth of architecture debt.
3. **Operability 8 despite checkpoint fail-open (c7) - is that also a safety deduction?** The 11 `enableCheckpoints=false` sites are a reliability failure of the recovery promise, not an enforcement bypass (checkpoints never guard a permission decision). Ruling: docked in operability only (keeps it at 8, not 9); safety lane correctly did not double-count, so safety-enforcement 7 stands without an additional penalty.
4. **Token-economy 8 vs the Bedrock silo (e3 vs e4).** In-lane tension inside r-econ, resolved there and adopted: the rung-7-vs-8 question is "does cache discipline exist and is it tested" (yes: 1,112-LOC spec, placement persistence); the silo plus the reported-usage trigger (e5) is exactly what prevents 9. No revision.
5. **Architecture "6 not 7" (c1/c3 vs c2's credit).** r-core conceded c2 (shim-hosted engine reuse avoids nanocoder's loop-duplication) but held 6 because the fused surface exceeds crush's 7-scoring pair and the second host plane re-infers state from render-schema heuristics (c3). I adopt 6: the anti-pattern avoided is not the same as a pattern achieved, and c3 is an architecture liability created by the hosting choice, not offset by it.

## Strongest / weakest

- **Strongest: token-economy (8)** - the only dimension on which the review evidence places roo strictly above the closest anchor: a tested prompt-cache placement layer with cross-turn persistence where the cline anchor records no cache discipline at all (e3, verified at `multi-point-strategy.ts:73-158`), plus non-destructive rewind-safe compaction with dedicated specs (`rewind-after-condense.spec.ts`, 567 LOC) that cline's delete-range compaction cannot match. Note: operability and docs-dx also scored 8; token-economy chosen because it carries the anchor-relative surplus.
- **Weakest: architecture (6)** - lowest score against the joint-largest weight (15). Evidence: `Task.ts` 4,619 LOC / ~86 members fusing loop+state+persistence+checkpoints+condense (c1), streaming/tool-dispatch as a free function mutating Task fields, `webviewMessageHandler.ts` 3,230 + `ClineProvider.ts` 3,149 god-pair, and the CLI plane's `AgentLoopState` reconstructed from incidental fields of the presentation stream (c3: "streaming" = a cost key's absence).

## Findings integration

32 lane findings (c1-c8, s1-s10, e1-e14) -> 29 merged records in `merged-findings.jsonl`. Deduped by concept, keeping the stronger-evidence record and recording the other id as a concept alias:
- `roo-code-c4` + `roo-code-e1` -> concept **rewind-safe-compaction** (alias non-destructive-condense-tagging); econ evidence kept (line-ranged impl citations + spec LOC), core's tag-hide citation merged in.
- `roo-code-c8` + `roo-code-e8` -> concept **crash-recovery-middleware** (alias resume-and-crash-discipline); core evidence kept (tighter crash-handler line ranges), econ's resume substrate citations merged in.
- `roo-code-s10` + `roo-code-e9` -> concept **loop-detection** (alias loop-breaker-plus-skills); safety evidence kept (wiring + spec size), econ's SkillsManager citations merged in.
Kept distinct despite sharing the concept term `permission-policy` (s1, s3, s5, s9): same-lane findings about different facts (fail-closed defaults / self-poisoning guard / terminal-ignore hole / untested router) with different kinds - collapsing them would destroy the safety ledger.

## Calibration notes

- **Archive demotion applied, not silent:** upstream remote is archived (`archived: true` on GitHub API, observed 2026-09-30; pushed_at 2026-05-15 == census HEAD). Rule "archived/dead caps at B" is in force. The raw weighted total 69.5 was already B, so the cap did not bind numerically - but it materially changed two dimensions: durability scored 4 (dead-project posture) instead of the 7-8 an active org-backed weekly-release project with this release/cadence profile would earn, and that scoring is what pulled the total to the low end of B. Had the raw total been A-range, this record would be capped at B with the pre-cap number noted here.
- **Upstream-outrank rule checked:** roo-code is a divergent fork, not a sync-fork derivative, so it *may* outrank cline (76.5) on merit; it does not (69.5), so the ordering rule is satisfied either way. No override needed.
- **Shallow-clone trap honored:** durability was not inferred from census (1 commit / 1 contributor is a clone artifact); external GitHub metadata + in-repo release artifacts used instead, per map rule 2.
- **S-gate:** band B; safety-enforcement 7 and verification 7 both >= 5, noted for completeness - the floor rule is moot here.
- **Anchor consensus:** 3/3 lanes + scout answer "closest anchor: cline"; adopted.
- **Tie-break transparency:** three dimensions scored 8 (token-economy, operability, docs-dx); strongest_dimension reports token-economy per the anchor-surplus rationale above, weakest is unambiguous (architecture 6, heaviest weight).
- **Carried caveats from lanes:** shallow clone = file-content evidence only; webview-ui approval-UI paths inferred, not exercised; skills auto-approve finding (s6) confidence med; e5 trigger-drift confidence medium. None change a band.
