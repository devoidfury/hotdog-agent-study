# open-codex (T1)

TypeScript-era Codex CLI (pre-codex-rs), forked from openai/codex 2025-04-16 by ymichael to run
on any OpenAI-compatible endpoint. ~12k LOC src (ts/tsx, excl. vendored ink components 2.0k),
~5k LOC vitest tests. Apache-2.0, NOTICE preserved.

## Anchor question

Closest anchor: **nanocoder (60.5)** -- same ink/TS shape where the loop is bound to the TUI
surface and sandboxing is real machinery behind an opt-in default -- but open-codex lacks
nanocoder's test corpus (5k vs 161k), has no compaction at all, and is verifiably dead, pulling
it far below; the dead-then-abandoned character echoes codel (23) without codel's zero tests.

## Census sanity

- head_date 2025-05-03 **remote-verified**: GitHub API `pushed_at 2025-05-03T23:35:44Z`, 17mo
  stale. `.git/shallow` present (1 commit "bump version"), so HEAD alone would not have been
  sufficient -- remote confirms. Repo `archived:false` but effectively dead. Rule (b) applies.
- Provenance `divergent-fork:codex` confirmed via API: `fork:true`, `parent/source = openai/codex`.
- Census `test_loc: 3527` understates: measured **5,002** lines across 51 test files
  (4 tsx + 46 ts + 1 snap; `find codex-cli/tests -type f | xargs wc -l` = 5,014 incl. snap).
  Not material to tier (19,972 non-test plausible counting root md/yaml/js + vendored blobs).
- Identity hygiene: distinct from `codex` (the S anchor, Rust line), `codex-infinity`
  (sync-fork), `code` (rename). This is the deprecated **TS** lineage only; no facts imported
  from siblings. package.json `repository` still says `ymichael/codex` (pre-rename).

## Provenance / fork delta (file:line, per appendix)

Delta vs upstream TS codex is confined to the provider plane:

- Provider table (openai, gemini, openrouter, ollama, xai) with per-provider key/baseURL/model:
  `codex-cli/src/utils/config.ts:39-125`; env-var provider fallback `config.ts:40-48`.
- Transport swapped Responses API -> Chat Completions: `this.oai.chat.completions.create`
  `codex-cli/src/utils/agent/agent-loop.ts:508`; delta/tool_call accumulation `:745-790`.
- Dual-shape normalization for chat vs responses tool items (`function.name` vs `name`,
  `call_id` vs `id`): `agent-loop.ts:296-321` -- the portability seam.
- Everything else (loop, approvals, sandbox, singlepass) matches upstream TS codex of that date.
- Delta test coverage: `tests/config.test.tsx` asserts only the **openai** provider (lines
  64-77); the 4 added providers have zero assertions. Fork-delta work is untested.

## Core loop (read: agent-loop.ts, all 1155 lines)

Single-tool (`shell`) while-loop; `apply_patch` rides through shell. The standout is
cancellation engineering: generation counter to drop stray stream events
(`agent-loop.ts:88-92,127,432-436`), `pendingAborts` set replayed as synthetic `tool` outputs
so the API contract survives an ESC-ESC mid-tool-call (`:100-107,371-385`), separate
stream/exec/hardAbort controllers (`:80-95,246-258`), retry taxonomy for timeouts/5xx/429 with
retry-after parsing (`:540-620`). Genuinely race-tested (`tests/agent-cancel-race.test.ts:8-36`
FakeStream yields-before-cancel). Rough edges: dead `processEventsWithoutStreaming` with
`@ts-expect-error` (`:1116-1140`), commented-out staging/flush machinery, one ~700-line `run()`.

## Compaction / context

None. Full history is replayed every request (`agent-loop.ts:505-512`), no usage accounting, no
cache discipline. Context overflow is a single terminal message: "shorten the conversation, run
/clear" (`:577-596`). Token-economy at the codel-1 rung; the only budget engineering is the
legacy `--full-context` singlepass mode's per-path cumulative size maps for zip truncation
(`src/utils/singlepass/context_limit.ts:17-40`) -- relic of an experimental upstream mode.

## Permission / sandbox

Default policy `suggest` (`config.ts:21`). `canAutoApprove` classifies safe-read-only commands,
auto-approves writes under cwd, else asks (`approvals.ts:74-140`). Sandbox: seatbelt is a real
`(deny default)` policy with scoped writes, read-all, **no network allow** (`macos-seatbelt.ts:81-97`)
-- network is blocked by default-deny. But `SandboxType.LINUX_LANDLOCK` is declared
(`sandbox/interface.ts:5`) and never implemented: `exec.ts:40` ternary silently routes every
non-seatbelt platform to `rawExec`. full-auto on Linux = unsandboxed, silently at runtime
(README:151 does disclose "Linux -- there is no sandboxing by default", so docs are honest even
if the runtime is not). `ask-user`-approved commands run unsandboxed by design (`handle-exec-command.ts:118-121`).
Coarse always-approve cache: `deriveCommandKey` collapses `bash -lc "<script>"` to the script's
first word (`handle-exec-command.ts:36-63`), so one "always approve" on `git status` mutes
prompting for all `git` invocations that session -- unpinned by tests.
Positive: process-group kill with SIGTERM->SIGKILL escalation (`sandbox/raw-exec.ts:83-131`).

## Historical snapshot value (brief's ask)

Preserves the deleted TS-era codex architecture intact: (1) the full-context singlepass mode
(`cli.tsx:78,251`, `singlepass/*` 4 modules) removed upstream; (2) the pre-AGENTS.md project-doc
convention `codex.md/.codex.md/CODEX.md` with 32 kB cap + truncation warning
(`config.ts:185-260`, `.git`-rooted discovery tested at `tests/project-doc.test.ts:26-30`);
(3) the earliest rollout format: auto-saved per-conversation JSON (`terminal-chat.tsx:112`) with
view-only `codex -v` replay, no resume (`cli.tsx:233-245`) -- ancestor of codex-rs rollouts;
(4) dual-model split agentic/fullContext per provider (`config.ts:110-125`).

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 5 | AgentLoop UI-agnostic via callbacks (`agent-loop.ts:41-53`) beats nanocoder's hook-bound loop, but 700-line `run()`, dead code (`:1116-1140`), history state fused into TUI (`terminal-chat.tsx`), parallel singlepass loop (`cli.tsx:251`) |
| verification | 5 | race/property-style specs vs fake streams: `agent-cancel-race.test.ts:8-36`, `agent-function-call-id.test.ts`, `agent-rate-limit-error.test.ts`; single-OS CI test+lint+typecheck (`.github/workflows/ci.yml:11-45`); no coverage gate, no matrix, delta providers untested (`tests/config.test.tsx:64-77`) |
| safety-enforcement | 4 | honest suggest-default + tested approvals (`approvals.test.ts:1-144`); seatbelt deny-default real (`macos-seatbelt.ts:81`) but Linux full-auto silently raw (`exec.ts:40`) -- below the 6 "tested approval, nothing underneath" rung because of the silent-degradation on the dominant server platform; mitigated by README:151 disclosure |
| token-economy | 2 | zero compaction, no usage/cost fields, full replay (`agent-loop.ts:505-512`); overflow = tell the user (`:577-596`); singlepass size-maps (`context_limit.ts:17-40`) is the only budgeting |
| orchestration | 1 | nothing: no subagents, queues, workflows, journals. Cancellation is operability, not orchestration |
| interop | 3 | `-q` headless (`cli.tsx:64`), rollout JSON readable (`cli.tsx:242`), shell completions (`cli.tsx:53`); no MCP/ACP/SDK; 5-provider compat table is model breadth, not interop surface |
| operability | 4 | rollout autosave + `-v` viewer (`terminal-chat.tsx:112`, `cli.tsx:235-245`), config.json/yaml + instructions.md (`config.ts:23-30`), DEBUG log tail (`cli.tsx:40-42`); no resume, no rewind, no crash posture |
| originality | 4 | the fork's chat-vs-responses normalization seam (`agent-loop.ts:296-321`) and env-key provider fallback (`config.ts:40-48`) are real, modest mechanisms; core ideas are upstream's relics (singlepass, codex.md) |
| durability | 1 | dead 17mo remote-verified; 1 maintainer; 2,371 stars + npm-published but no signal of life |
| docs-dx | 5 | honest security limits (README:151), per-provider missing-key onboarding with key-creation URLs (`model-utils.ts:110-144`), rich `--help` (`cli.tsx:50-90`); docked: inherited upstream README describing the upstream product, docs/ dir holds only CLA.md |

**weighted_total = 36.0 -> band D.**

Strongest dimension: **verification** (the cancellation-race specs genuinely prove loop
semantics under adversarial timing, not existence checks; the weakest of the "5" trio is
arguably architecture). Weakest: **orchestration** (1; durability ties numerically but is weight-5).

## Calibration

- Rule (b): dead >12mo, remote-verified -- capped at B; does not bind (36.0 is D on merit).
- Rule (a): divergent fork, scored on merits; nowhere near upstream codex's 88.5, so the
  sync-fork cap question is moot.
- Boundary check: 36.0 is 9 pts from the 45 boundary; no boundary-risk note.
- License: Apache-2.0 with NOTICE intact -- concept and code-adjacent portables both allowed,
  though dead TS code makes only concepts worth carrying.
