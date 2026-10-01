# grok-cli (T1) — grok-dev, community Grok-API coding agent

**Identity:** manifest `grok-dev` (package.json:2), repo `superagent-ai/grok-cli`. Community-built
wrapper around the public xAI Grok API — README:3 carries an explicit "not affiliated with xAI
Corp" disclaimer; provider is `@ai-sdk/xai` (package.json:70), auth is a user-supplied
`GROK_API_KEY`. NOT the xAI-owned `grok-build` (separate T3 subject; different org, language,
and scope). Author field "Vibe Kit"; distributed as npm `grok-dev` + install.sh.

**Census sanity:**
- `test_loc: 0` is wrong. 48 `*.test.ts` files co-located in `src/`, 5,379 LOC, ~253 `it(` cases
  (e.g. `src/agent/compaction.test.ts:46`, `src/tools/bash.test.ts:28`). Glob missed co-located
  tests. `non_test_loc: 27711` ≈ correct (all src ts/tsx = 31,150 incl. tests).
- Shallow clone (`.git/shallow`, 1 commit, head_date 2026-05-15). Remote check:
  `pushed_at 2026-07-06`, not archived, 3,484 stars / 422 forks. Active; contributors=1 is
  shallow-clone-unreliable.

**Anchor question (one sentence):** Closer to **nanocoder** than any other anchor — a
single-team TS terminal agent with real machinery wired to the wrong defaults (no host approvals,
sandbox opt-in and platform-limited), with tests that assert properties but no CI running them;
it sits a notch below nanocoder on verification, a notch above on orchestration/originality.

## Architecture (6)
- Modules do separate: `src/agent` loop, `src/grok` provider, `src/storage` sqlite, `src/mcp`,
  `src/lsp`, `src/hooks`, `src/verify`, `src/telegram`. Clean observer seam between loop and UI
  (`agent.ts:155-176` ProcessMessageObserver).
- Loop is the Vercel AI SDK `streamText` full-stream switch — tool execution happens inside SDK
  callbacks in `src/grok/tools.ts`, not in an explicit harness loop with step contexts
  (`agent.ts:1936-1951`). Far from codex's StepContext model; closer to crush's fused shape.
- God files: `src/ui/app.tsx` 5,862 LOC (`:597` App) and `src/agent/agent.ts` 2,830 LOC fusing
  interactive loop + batch-API loop + delegation wiring + recap (`:535` class Agent). Per the
  pi errata, a >5k product-surface file is a docking factor. Hence 6, not 7.

## Verification (5)
- Real property tests, not existence tests: cut-point never lands on tool-result
  (`compaction.test.ts:46`), split-turn prefix capture (`:61`), summary-excluded-from-recompaction
  (`:80`), shuru wrap/escaping/net-flags (`bash.test.ts:28-77`), trust answer parsing
  (`workspace-trust.test.ts`).
- **But no CI job runs tests.** `typecheck.yml` = format+lint+typecheck+build:binary only;
  `security.yml` = `bun pm untrusted` + trufflehog; `release.yml` = 3-OS build + provenance
  attest, no vitest anywhere. Husky pre-commit runs biome only. Matches the seeded
  `test-suite-without-ci` anti-pattern; caps this below the crush/nanocoder 7 rung at 5.
- No evals, no fuzzing, no -race equivalent.

## Safety-enforcement (5)
- **No approval gate for bash, edit_file, write_file, or MCP tools.** The only `needsApproval` in
  the tree is `paid_request` (`src/grok/tools.ts:901-905`), and it's the x402 payment tool.
  Host mode = autonomous execution of any command.
- Real sandbox exists: Shuru microVM wrapping (`src/tools/bash.ts:450-557`), fail-closed when
  unsupported (`:411-413` returns error, does NOT fall back to host — better than nanocoder's
  fail-open), network off by default with host allowlists (`:390-399`). But macOS-arm64 only
  (`workspace-trust.ts:23-25`) and `sandboxMode` defaults to `"off"` (`bash.ts:42`).
- Per-workspace trust prompt nudging sandbox on first run, default-Y, tested
  (`src/index.ts:192-235`, `src/utils/workspace-trust.ts:52-66`, file mode 0600 `:104`).
- PreToolUse hooks block (exit code 2) but bind only the bash tool
  (`hooks/executor.ts:11,68`; call site `grok/tools.ts:107` — no file-tool/MCP coverage).
- Sandbox-mode mutation denylist is regex-based (git-mutate, formatters, package installs) and
  tested (`bash.ts:570-610`, `bash.test.ts`) — bypassable, same class as nanocoder's denylist.
- Host desktop automation (`computer_*` tools, `src/tools/computer.ts`) with prompt-only
  restraint — a large attack surface with no per-action approval.
- No SECURITY.md. Hook scoping honesty is a bright spot (below).
Net: same rung as nanocoder (real machinery, wrong default) = 5.

## Token-economy (6)
- One standard auto-summarize tier, done well: reserve-headroom trigger
  (`compaction.ts:247-252`), keep-recent-token budget with turn-boundary cut and split-turn
  prefix summarization (`:264-297,345`), incremental summary-update pass on subsequent compactions
  (`:73-85`, previous-summary threading `:303`), tool-result truncation to 2k chars for summary
  input (`:27`). Overflow ladder: on context-limit error with no output, halve keep-recent to a
  4k floor and retry once (`agent.ts:1908,2134-2140`; `compaction.ts:342-347`).
- Estimation is naive chars/4 (`compaction.ts:211-237`), not measured-usage projection (pi's
  rung). No prompt-cache discipline at all. Cost fields tracked to sqlite usage events
  (`agent.ts:927`, `storage/usage.ts`); `--batch-api` halves cost for unattended runs
  (`index.ts:369`, `grok/batch.ts`). Sits between the 6 (crush) and 7 (cline) rungs; 6.

## Orchestration (7)
- Foreground `task` subagents (explore/general/verify/computer/custom, `agent.ts:354-483`) plus
  background `delegate` = detached child processes with persisted JSON state, stable ids, output
  files, nested-delegation prohibition (`delegations.ts:53-60`).
- Cron/one-shot schedule tools + standalone scheduler daemon with pid-file lock and detached
  headless firing (`daemon/scheduler.ts:13-31`, `tools/schedule.ts:278-335`); recoverable via
  `--background-task-file` (`index.ts:371,384`).
- No loop detection (nothing like crush's `loop_detection.go`), no spend budgets on delegation.
  crush rung = 7.

## Interop (6)
- MCP client only (stdio + http, `/mcp` modal; `src/mcp/runtime.ts:32-55`), mounted only in
  agent mode (`agent.ts:1926`). No MCP server, no ACP, no published SDK.
- LSP client with semantic tools (`src/lsp/client.ts`, `builtins.ts`) — rare and valuable.
- Headless contract: `--prompt`, `--format json`, `--session latest` (`index.ts:365-369`,
  `headless/output.ts`). Telegram remote control is a genuinely different second surface
  (pairing codes, audio STT, streamed previews — `src/telegram/`). Below crush's 7 for the
  agent-mode-only MCP and no SDK.

## Operability (6)
- bun:sqlite WAL store with tested migrations and per-message sequence transcript; compaction
  persisted as a replayable checkpoint record (`storage/transcript.ts:210`,
  `compaction.test.ts:106` builds effective transcript from latest checkpoint — good crash story
  for context state).
- Resume via `--session <id|latest>` (`index.ts:365`), session recap
  (`agent.ts:727-735`), self-update/uninstall with dry-run (`README` self-management block),
  humanized API errors (`agent.ts:1772`), SIGTERM clean exit (`index.ts:43-52`).
- No rewind/checkpoint-revert of file edits (verify/checkpoint.ts is sandbox worktree tooling,
  not user undo); no diagnostics command. nanocoder rung = 6.

## Originality (7)
- **x402 agentic payments**: local wallet, `paid_request` with mandatory approval (default),
  BRIN security scan that hard-blocks URLs scoring <25 before paying
  (`payments/service.ts:103-123`), 402-header probe feeding the approval prompt
  (`agent.ts:2043-2076`), payment audit log. No other subject in the corpus has payments.
- Telegram remote-control with pair codes + expiry (`telegram/pairing.ts:27`), voice-note STT,
  preview streaming (`telegram/preview-stream.ts`).
- Hooks deliberately loaded from user settings only, with an explicit anti-repo-committed-hook
  security rationale in comments (`hooks/config.ts:5-11`) — exemplary hook-trust-scoping.
- Encrypted-reasoning handling: PGP/AGE marker detection to hide/strip opaque reasoning from
  display and from compaction/summary inputs (`agent.ts:1977-1986`, `reasoning.ts:4-9`).
- Shuru microVM + default-Y workspace-trust prompt is a distinctive safety posture shape.
Not field-defining (no codex/pi-level mechanism), but genuinely unique features = 7.

## Durability (5)
- Active: remote `pushed_at 2026-07-06` (verified; shallow HEAD 2026-05-15 would mislead),
  3.5k stars, org-owned, npm releases with build-provenance attestation
  (`release.yml:70-71`), current CHANGELOG, weekly scheduled security scan (`security.yml:8-10`).
- Against: apparent single-author project ("Vibe Kit"; contributors=1, shallow-caveated), no
  SECURITY.md, stale AGENTS.md describing a different tree (`.eslintrc.js`,
  `settings-manager.ts`, `@vibe-kit/grok-cli` — `AGENTS.md:5-29`). Between nanocoder(4) and
  crush(8) = 5.

## Docs-dx (5)
- 496-line README with honest scope (trademark disclaimer, terminal support list, headless
  examples); `--verify` one-shot QA mode; clear install path. Stale AGENTS.md, no docs/ tree,
  no SECURITY.md; MCP/subagent/settings reference lives only inside README. nanocoder(7) has a
  generated config schema; this doesn't = 5.

## Totals
weighted = 6*15 + 5*15 + 5*10 + 6*10 + 7*10 + 6*10 + 6*10 + 7*10 + 5*5 + 5*5
= 90+75+50+60+70+60+60+70+25+25 = 585 → **58.5, band C**.
Strongest: originality (7, x402 payments + Telegram control are corpus-unique).
Weakest: verification (5, tests prove properties but nothing in CI runs them).
License MIT permissive; concepts and (with attribution) code are portable. No boundary risk
(not within 2 of 88/78/65/45).
