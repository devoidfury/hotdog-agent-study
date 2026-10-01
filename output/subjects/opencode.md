# T3 subject report: opencode (run 20260929-0502-t3-giant-review-4)

Integrator synthesis of lanes core / safety / econ. 35 merged findings (f-core 10 + f-safety 13 + f-econ 13, minus one cross-lane dedup). Read-only static review; the subject was never executed.

## Final scores

| dimension | raw (x/10) | weight | weighted | lane |
|---|---|---|---|---|
| architecture | 8 | 15 | 12.0 | core |
| verification | 8 | 15 | 12.0 | safety |
| safety-enforcement | 6 | 10 | 6.0 | safety |
| token-economy | 8 | 10 | 8.0 | econ |
| orchestration | 7 | 10 | 7.0 | econ |
| interop | 8 | 10 | 8.0 | econ |
| operability | 8 | 10 | 8.0 | core |
| originality | 8 | 10 | 8.0 | integrator |
| durability | 8 | 5 | 4.0 | integrator |
| docs-dx | 8 | 5 | 4.0 | econ |

**weighted_total: 77.0 -> band B** (65-77). One point below the A floor; the sensitivity is recorded in calibration notes.

**Closest anchor: cline.** **Strongest dimension: architecture. Weakest: safety-enforcement.**

## Strongest dimension: architecture (8/15), with evidence

The V2 session slice is the strongest single piece of engineering evidence in the entire review, and two of its mechanisms are unique across the anchor set:

- Event-sourced session core: a durable per-aggregate sequenced event journal with projector and replay-under-transactions (`packages/core/src/event.ts:134,138-142,217-318`; dated drizzle migrations incl. `20260604172448_event_sourced_session_input.ts`). No anchor (codex JSONL rollouts, crush/cline sqlite state) puts a real event store under the agent loop (merged concept `event-sourced-session-core`).
- The crash-correctness properties the journal enables are implemented and property-tested, not claimed: orphan-tool settlement before any model call (`core/src/session/runner/llm.ts:119-148`, spec at `core/test/session-runner.test.ts:2262`), idempotent durable admission (`session/input.ts`, `session.ts:368-383`), and a 105-LOC run coordinator with an explicit join/coalesce/interrupt algebra (`run-coordinator.ts:73-103`).
- Deliberate decomposition: 439-LOC runner over a 105-LOC coordinator over a 175-LOC context-epoch, plus a 3,466-LOC `it.effect` property spec for the loop.

Why 8 and not 9 (the dock is cline's): the V1→V2 migration is not a doc artifact -- both execution stacks ship side by side in one binary (V1 `opencode/src/session/prompt.ts:1088` while(true) and V2 `core/src/session/runner/llm.ts:400`; the protocol `v2.session.prompt` surface IS mounted, `server/routes/instance/httpapi/api.ts:48,80` -> `packages/server/src/handlers/session.ts:21`), the 1,631-LOC SessionPrompt monolith still serves legacy clients, and the layering law is AGENTS.md prose with no mechanical enforcement while `core` depends on 20 `@ai-sdk/*` packages (c7). God-files persist (`codemode/interpreter/runtime.ts` 3,652; `provider.ts` 2,072).

## Weakest dimension: safety-enforcement (6/10), with evidence

The tested-approval-with-nothing-underneath rung (pi/cline), and the weakest on both raw score and weighted contribution:

- Enforced, tested, real: single `ctx.ask` chokepoint wired to the Permission service with `Effect.orDie` (`opencode/src/session/tools.ts:81-89`); findLast wildcard grammar with default-ask fallback unit-tested as an algebra (`permission/index.ts:29-37`; ~60 decision specs in `test/permission/next.test.ts`); per-subcommand shell asks from a parsed AST before spawn (`tool/shell.ts:257-290,:627`); reject-cascade/always-persist/directory-isolation lifecycle specs; plan-mode subagent-bypass regression tests importing the production helper.
- Nothing underneath: no sandbox of any kind, stated verbatim in SECURITY.md ("The permission system exists as a UX feature"); shipped default ruleset is `*: allow` -- bash/edit/write/apply_patch/webfetch auto-approve inside the workspace until the user opts in (`agent/agent.ts:119-135`); zero named prompt-injection defense (grep over both src trees: review-template text only); webfetch has no SSRF guard (`tool/webfetch.ts:76-126`); MCP servers auto-stored `enabled: true` on connect (`mcp/index.ts:650,941`). The composed attack chain -- poisoned web/MCP content, unmarked in durable history, into a default-allow unsandboxed shell -- is the subject's defining risk (s2,s3,s4).
- The approval grammar itself is an audit target: the command-prefix arity dictionary defining what "allow git *" means was LLM-generated, prompt preserved verbatim in-source (`permission/arity.ts:14-30`), guarded by 33 happy-path test lines.

## Score-disagreement resolutions (explicit)

1. **Anchor closeness was split three ways -- core said cline, safety said pi, econ said codex.** Resolved to **cline**. The dominant structural fact of this subject -- a mid-flight dual-tree migration that docks the two heaviest dimensions (architecture 8 with both loops shipped; docs-dx 8 docked exactly for the cline "two realities unexplained to users" reason) -- is cline's profile. Pi is the right rung for the *safety dimension in isolation* (SECURITY.md honesty + tested approval, nothing underneath) and codex is the right *ceiling reference* for cache/interop mechanisms, but opencode is materially above pi on orchestration, interop, and docs, and materially below codex on enforced grammar, server breadth, and supply-chain enforcement. The single closest whole-profile subject is cline.
2. **V2 credit asymmetry (core credited V2 under architecture; econ docked token-economy for "principled machinery in unreleased V2").** Not contradictory; adopted as an explicit rule: V2 is credited where it verifiably *ships in the binary* (the v2.session.prompt route is mounted -- scout correction accepted -- so its architecture and tested properties count), and cost/safety *behavior* is scored on the V1 path because that is what the shipping product actually does to a user's tokens and shell. This is why architecture takes the event journal and context-epoch at face value while token-economy is docked for the shipping path being the static 2+2 cache heuristic (`provider/transform.ts:358-400,483`) and usage-counted overflow.
3. **Safety 6 vs pressure toward 5 (nanocoder rung) from the default-allow finding s3.** Kept at 6. Nanocoder was docked for a fake fail-open jail; opencode ships no false protection claim -- the docs say the wall is UX -- and pi's 6 likewise runs no default protection. The lane's own "5.5 is defensible if the study weights default-on heavily" is recorded, unaveraged, here; 6 stands because mechanism and declared threat model match.
4. **Verification kept at 8 despite the s12 stubbed-`ctx.ask` gap.** The gap (no end-to-end "rejected -> process never spawned" spec through the real service; tool tests capture-then-throw at `test/tool/shell.test.ts:164-166`) is real but the semantics are covered at service level (RejectedError cascade specs plus the `orDie` wiring), and 8 is the anchor-ceiling score (no fuzzing, no in-CI evals -- shared with codex/cline/pi). Downgrading to 7 would double-count the ceiling.
5. **Operability 8 (core) corroborated, not contested, by the safety lane's crash findings.** s13/c9 (merged as `crash-durability-gaps`) are exactly why it is 8 and not the codex/pi 9: non-durable activity status (`runner/llm.ts:51-53`), no sqlite-corruption recovery, journal version-binding TODO (`event.ts:180`).
6. **docs-dx reported by econ as "8/5-weighted" -- normalized** to raw 8/10 contributing 4.0, consistent with the shared 10-point raw scale used by all lanes.

## Integrator-owned dimensions

### Originality: 8/10 (weight 10)

Criterion applied: the idea must exist as tested code, not marketing copy. Verified in code, all three flagship claims hold:

- `event-sourced-session-core`: real journal, replay, projector, dated migration, 81-case `it.effect` property suite (`core/test/session-runner.test.ts`, 3,466 LOC). Unique among anchors (c2).
- `context-epoch-cache-baseline`: immutable provider-cache baseline with reconcile/replace semantics and property tests for reuse/rebuild ("reuses one durable baseline after the context producer changes" :741; "rebuilds the baseline directly after completed compaction" :1007) (`core/src/session/context-epoch.ts:39-80,129-152`). Beyond pi's cache warming and codex's cache-key gating (c4,e3).
- `protocol-level-cache-budgeting`: default-on auto breakpoint policy with the 1.25x/0.1x Anthropic cost math in-code, hard 4-breakpoint cap with counted drops, 5m/1h TTL buckets (`packages/llm/src/cache-policy.ts:1-36`; `protocols/anthropic-messages.ts:234-252,:534-537`). No anchor does this at the wire layer (e3).
- Additional portable originals: orphan-tool settlement with property tests (c3), managed tool-output offload with 7-day GC (`core/src/tool-output-store.ts:14-18`, e5), projected-request compaction trigger (`core/src/session/compaction.ts:232-242`, e1), doom-loop-into-permission-ask (`session/processor.ts:355-380`).

Why 8 not 9: the flagship machinery lives in the V2 tree that is mounted but not the default product path; the shipping cache behavior is the V1 static heuristic (e4). And one genuinely novel artifact -- the LLM-generated approval grammar (s6) -- is originality pointed the wrong way: novelty in a security grammar that a purpose-built DSL (codex execpolicy) exists to avoid.

### Durability: 8/10 (weight 5) -- from census metadata and in-snapshot signals

The census row is known-corrupt and was not used at face value: `contributors: 1, commits: 1` is a shallow-clone artifact (scout flagged; same failure mode as the 358k test_loc row, actual ~162.5k). Scored instead on locally checkable project-vitality metadata:

- **Commit cadence**: HEAD `b471c2b` dated 2026-09-26, three days before run date, committed by `opencode-agent[bot]` with five-digit PR number (#51538) -- high-frequency, bot-integrated mainline flow, not a stalled repo.
- **Institutional backing**: remote `anomalyco/opencode`; package versioning `@opencode-ai/core` v1.18.32 (mature release line); published VS Code extension with its own CI publish workflow, published GitHub Action, 26 GitHub workflows, 25-locale doc tree with a CI drift gate -- organizational-grade infrastructure, not a hobby repo.
- **License**: MIT ("Copyright (c) 2025 opencode"), permissive, unambiguous, no relicensing-residue signals.
- Not archived, no dead-repo signals anywhere in the snapshot.

Why 8 not 9 or 10: contributor count, org solvency, and real release cadence are unverifiable from a 1-commit shallow clone; the score rests on indirect in-snapshot evidence. The ongoing V1→V2 rewrite is both a vitality signal and a concentration risk (bus-factor on the durable-core design).

## Rule checks and calibration notes

- **Sync-fork rule**: not applicable -- provenance verdict is *original* upstream snapshot; `grep -rniE "crush|charmbracelet|kujthoxha"` zero hits, no rename residue. No demotion applied.
- **Archived/dead cap**: not applicable -- active mainline (see durability evidence).
- **No silent demotions**; every dock is attributed above.
- Census artifacts logged: test_loc 358,013 claimed vs ~162.5k observed (`find` over `*.test.ts(x)`, 683 files); contributors/commits 1/1 from shallow clone. Scores use observed values, never census rows.
- Band sensitivity: 77.0 is at the top of B; originality 9 or verification 9 would cross into A (78). Held at B deliberately -- both are anchor-ceiling or dual-tree-limited at 8 by named evidence, and rounding up on either would double-count against the docking rationale above.
- merged-findings.jsonl dedup: `opencode-c9` (core, "process-local-activity") and `opencode-s13` (safety, "crash-and-durability") describe one concept; canonical record `opencode-m1` keeps the stronger safety evidence, absorbs c9's unique citations (`event.ts:180`, `specs/v2/session.md:32`), and lists both originals in `aliases`. All other 34 ids are 1:1 with source findings; id and concept uniqueness machine-checked.
