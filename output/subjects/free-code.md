# free-code -- differential triage (T0): leaked-snapshot:claude-code CONFIRMED

Anchor sentence: closest anchor is codex (same category of mature loop/permission design), but the tree is not this project's own -- anchor comparison is moot for a leak.

## Leak evidence (upstream claude-code not in corpus; internal markers)

1. Self-declared: `package.json:1-4` -- name `"claude-code-source-snapshot"`, version `"2.1.87"` (Claude Code product versioning), description "Reconstructed Bun CLI workspace for the Claude Code source snapshot."
2. Proprietary identity strings in code: `src/constants/prompts.ts:452` and `src/constants/system.ts:9-10` -- "You are Claude Code, Anthropic's official CLI for Claude."
3. **Internal-only marker**: `src/constants/prompts.ts:245` instructs the model to post session-share links to Slack channel `#claude-code-feedback` "channel ID C07VBSHV7EV" -- an Anthropic-internal routing identifier; no public npm artifact would need it in source. Same marker family in `src/skills/bundled/stuck.ts`, `src/utils/permissions/yoloClassifier.ts` (grep `ccshare`).
4. Real internal engineering comments, not decompiled-minified output: `src/main.tsx:1-8` times import cost "~135ms" and keychain prefetch behavior -- consistent with a genuine source leak reconstructed into a buildable tree, not scraped docs.
5. No development signal: no LICENSE file (top-level `ls` confirms), test_loc 199 against 1,914 source files (`find src -name '*.ts*' | wc -l`), 1 commit, IPFS mirror advertised (README badge) -- distribution posture of a leak, not a project.

## Deliberate safety stripping (README.md:9, 60-72)

"All telemetry stripped. All guardrails removed." README.md:66-72 documents removing "injected 'cyber risk' instruction blocks" and "managed-settings security overlays" that Anthropic ships -- i.e., the fork markets itself by deleting the upstream's safety layers. Permission machinery from upstream is retained (`src/tools/BashTool/bashPermissions.ts:1469,1507` `bypassPermissions` handling; `src/query.ts:12` autoCompact carried intact), so this is Claude Code minus guardrails.

## Scoring (code observed is Claude Code's, minus its safety posture; reconstruction incomplete)

arch 6 (real CC structure; "34 broken flags" reconstruction, README.md:280), verif 1 (199 LOC tests), safety 2 (mechanisms present but deliberately stripped of policy overlays -- misleading posture, worse than absent-by-omission), token 6 (CC compaction/autoCompact present), orch 6, interop 7 (MCP + agent-sdk deps, `package.json` deps), oper 5, originality 0 (nothing here is this repo's), durability 1 (1 contributor, 1 commit, expected takedown), docs-dx 3. **Weighted 38.5 -> D.**

## Findings policy

License-risk x2 (high). **No code-adjacent portables** per protocol rule 3 -- even "ideas" notes here must be pure-concepts only (Claude Code is proprietary, no license file present at all).
