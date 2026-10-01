# Anchor review: nanocoder (Nano-Collective/nanocoder, TypeScript) — C (60.5, upper-C)

Tier: T2 by census (234k "non-test" LOC) but the census split is wrong (see calibration). Shallow clone (1 commit) - activity evidence from CI/release workflows, not git history. MIT.

**Closest anchor: n/a — is anchor.** Upper-C reference: a broad feature surface with genuine tests behind surprising corners (an actual OS jail), undermined by a UI-coupled loop and one-person direction.

## Dimensions

### architecture — 5
- The agent loop lives in React hooks: `source/hooks/chat-handler/conversation/conversation-loop.tsx` (1414 LOC) with `components/user-input.tsx` at 1405 LOC; agent state and Ink UI state are entangled, so non-UI consumers (ACP, daemon) must go through parallel paths (`acp-conversation.ts`, `message-handler.ts`).
- Services layer does exist and is coherent: `source/services/` (bash-executor, checkpoint-manager, file-snapshot, timeline-manager), `source/session/`, `source/daemon/` with clean module specs beside each.
- Known gaps: 40+ top-level `source/` dirs with no enforced layering (utils importing handlers, `app/utils/handlers/compact-handler.ts` owning command logic).

### verification — 7
- ~161k LOC of colocated ava specs (census counted 1,057 - wrong pattern); specs assert behavior, not existence: `bash-sandbox.spec.ts:15-40` asserts exact spawn plans per platform including detached flag, `command-injection.spec.ts:14-25` runs shell-metacharacter attacks through the real `execFileSync` path.
- CI: org-shared PR workflow with coverage-drop failure gate (`.github/workflows/pr-checks.yml:17-21`), a dedicated Linux jail job that installs bubblewrap and disables AppArmor userns restrictions to run `bash-executor-sandbox.spec.ts` for real (`pr-checks.yml:63-95`), config-schema drift check (`:124`), `pnpm audit` script (`package.json:49`).
- Deduction: no evals, no fuzzing, TUI component coverage is thin.

### safety-enforcement — 5
- Bash requires approval unless explicitly always-allowed (`source/tools/execute-bash.tsx:217`), ACP has a permission bridge (`source/acp/acp-permission.ts`, spec'd).
- The jail is real but **off by default and fail-open**: `const sandbox = getAppConfig().sandbox === true` (`source/services/bash-executor.ts:83`), with a planned-spawn union that silently degrades to plain `sh -c` (`bash-sandbox.ts:6-11`); macOS seatbelt + bwrap profile generation at `bash-sandbox.ts:53-60`.
- Dangerous-command detection is a regex deny-list (`execute-bash.tsx:189` `/rm\s+-rf\s+\/(?!\w)/i`...) - trivially bypassable by rewriting; it gates prompts, not execution.

### token-economy — 6
- Auto-compaction with tunable threshold + session override, wired into both UI and ACP paths (`source/utils/auto-compact` via `compact-handler.ts:50-51,126`, `acp-conversation.ts:46 maybeAutoCompact`).
- Cost visibility: `source/usage/calculator.spec.ts` (priced usage), `source/stats/`, repo-map for codebase context (`source/repo-map/`).
- Deduction: no prompt-cache discipline, no compaction tiers.

### orchestration — 6
- Ambitious daemon: per-project event loop owning cron + file-watcher triggered skill runs, IPC server, lockfile, and a backpressure dispatcher (`source/daemon/daemon.ts:1-12`, `ipc.ts`, `lockfile.ts` specs).
- Subagents with built-ins and failure messaging (`source/subagents/built-in/`, `failure-message.spec.ts`), checkpoints + timeline (`services/checkpoint-manager.ts`, `timeline-manager.ts`).
- Gaps: no budgets, resumability is session-file level (`session/resolve-session.ts`), daemon is new enough that crash-recovery semantics are unproven in-repo.

### interop — 7
- The VS Code plugin talks to nanocoder over its **own ACP server** (`plugins/vscode/src/acp-client.ts`, `acp-process-manager.ts`; core in `source/acp/` 14+ modules with specs) - the ACP contract is load-bearing, not decorative.
- MCP client (`source/mcp/mcp-client.ts` 1150 LOC), LSP (`source/lsp/`), custom tools + hooks + skills install pipeline (`source/skills/install.ts`).
- Deduction: no published embeddable SDK; rival-harness import limited to AGENTS.md-family context files.

### operability — 6
- Sessions with title generation (`session/title-generator.ts`), lockfile-guarded daemon, settings wizards (`app/components/settings-*`), history renderer for resuming, `docs/getting-started` + `docs/battlemap.md`.
- Gaps: no crash-recovery middleware visible (no process-level recover strategy in-repo), diagnostics limited to `source/utils` logging.

### originality — 7
- Event-driven backpressured skill daemon (`daemon.ts` comment block:1-12) - unique among anchors so far; memory proposals as reviewable artifacts (`source/memory/proposal-store.ts`); text/XML tool-call parsing for weak providers (`tool-calling/xml-parser.ts`); battlemap docs; lazy command registry (`commands/lazy-registry.ts`).
- Deduction: most headline features are variants of seeded concepts rather than new mechanisms.

### durability — 4
- Effectively solo-maintained collective; shallow clone caps history evidence, but signals agree: 1 contributor, bus factor 1, young rewrite churning fast (README.zh-CN/TW, badges, homebrew/nix publish workflows show real distribution effort: `update-homebrew.yml`, `update-nix.yml`).
- No institutional backing, no SECURITY.md, CLA absent.

### docs-dx — 7
- Structured docs tree (`docs/configuration`, `docs/features`, `docs/getting-started`), three-language README, generated JSON schema for config (`schemas/agents.config.schema.json` drift-gated in CI).
- Deduction: error-message quality unverifiable statically; onboarding still assumes the TUI.

## Verdict
**Strongest: verification (7).** Weakest: durability (4).
Weighted 60.5 -> **C** (upper). C-anchor lesson: feature breadth + genuine tests can coexist with a structurally weak core - the UI-coupled loop caps architecture at 5 and drags everything wired through it (ACP path, headless). Also the fail-open jail shows why "has a sandbox" must always be checked against its default: real jail + off-by-default + regex deny-list = 5, not 7.
