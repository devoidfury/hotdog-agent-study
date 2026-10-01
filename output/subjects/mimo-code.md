# mimo-code -- differential triage (T0) vs opencode (upstream in corpus, not yet scored)

Anchor sentence: closest anchor is crush (B 69.0) because crush shares opencode's ancestry; the actual lineage tree is opencode itself.

## Verdict: census "rename-clone:opencode" is a detector misfire -- this is a divergent fork with a large delta

- Why it tripped the detector: root `package.json:3` name still `"opencode"`, `packages/opencode/` directory never renamed (census: 2,511 files identical). But README.md:553 states plainly: "MiMoCode is built as a fork of OpenCode... keeps all core OpenCode capabilities... and adds persistent memory, intelligent context management, subagent orchestration, goal-driven autonomous loops, compose workflows, and self-improvement via dream/distill."
- Delta is structural, not branding: file-level diff vs opencode -- 1,146 files differ, 777 fork-only, 860 upstream-only (fast-moving upstream). Fork-only core additions under `packages/opencode/src/`: `memory/` (FTS-backed: `memory/service.ts:1-25` reconcile+search over sqlite FTS, `fts-query.ts`, `write-gate.ts`), `actor/` + `inbox/` (actor registry / multi-actor lifecycle), `workflow/` (runtime.ts, sandbox.ts, persistence.ts), `cron/`. Backed by 40+ own drizzle migrations (`packages/opencode/migration/20260515010000_memory_fts`, `20260603000000_workflow_run`, `20260614000000_permission_grant`, `20260608000000_claude_import`...). Session loop itself heavily rewritten (`session/prompt.ts` diff: compose-protocol markers, attachment classification/shrinking, token logging).
- Own packages added: `packages/extensions`, `packages/shared`, enterprise/desktop surfaces; own CI: `.github/workflows/{lint,test,typecheck}.yml`.

## License

MIT compliant: `LICENSE` retains `Copyright (c) 2025 opencode` and adds `Copyright (c) 2026 MiMo Code, Xiaomi Corporation`. Nuance recorded as finding: `USE_RESTRICTIONS.md:5-9` imposes use restrictions on "MiMoCode or any derivatives" -- unenforceable add-on conditions over MIT-licensed portions (GPL-style inconpatibility trap for downstream reusers).

## Scoring (provisional -- opencode has no corpus score to anchor against; T0 depth)

arch 7 (opencode module graph + actor/workflow/memory layers, some package sprawl), verif 5 (200k census test LOC + test.yml, depth unverified), safety 5 (permission_grant persistence present, binding unverified), token 7 (memory FTS + context-inheritance migration; 99%/95% cache-hit claims are marketing, unverified), orch 7 (workflow_run journal, cron, actor lifecycle), interop 7 (ACP dir differs from upstream, MCP kept), oper 7, originality 3 (delta real but rides corpus-known opencode design; memory-FTS + compose are the novel bits), durability 6 (Xiaomi institution; single-contributor shallow artifact), docs-dx 6. **Weighted 60.0 -> C (provisional).** No outrank check possible until opencode is scored; expected to move toward crush's B on full review.
