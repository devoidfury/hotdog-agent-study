# kimi-cli (T2 review)

**Anchor question.** Closer to **cline** than to any other anchor: both are docs-heavy,
multi-surface products whose safety is "tested approval, nothing underneath," whose sessions
are journal-backed, and whose cores are separated from their UI hosts -- kimi-cli exceeds cline
on ACP and flow orchestration but lacks cline's test mass; it sits just below cline's 76.5
overall, in the same B band.

## Provenance & identity

- Census row verified: `original`, not a fork. Apache-2.0 (`LICENSE`); `NOTICE:1-12` discloses
  one Codex-derived file (`src/kimi_cli/skills/skill-creator/SKILL.md`, Apache-2.0 compatible).
- **Relationship to kimi-code (code evidence, not guessing):** kimi-cli is the *predecessor*,
  kimi-code (TS monorepo) the *successor rewrite by the same team*; they are NOT forks of each
  other. Evidence:
  - `klips/klip-11-kimi-code-rename.md` (in-repo KLIP, Status: Implemented): rebrand decision
    inside this repo; `packages/kimi-code/` here is only a name-holding tombstone package
    (contains just `src/kimi_code/__init__.py`; tested by `tests/core/test_kimi_code_tombstone.py`).
  - `pyproject.toml:4` description: "[Archived] Legacy Python Kimi CLI... Use Kimi Code CLI
    instead"; `SECURITY.md:1` no security updates for kimi-cli/kosong/pykaos/kimi-sdk;
    remote GitHub API: `archived: true`, `pushed_at 2026-09-22`.
  - `CHANGELOG.md` 1.51.0/1.52.0: final releases turn the CLI into a redirector; HEAD commit
    "short-circuit entry points to a Kimi Code installer (#2666)".
  - Python code retrofitted to the TS product's telemetry vocabulary: `soul/approval.py:39-43`
    ("Maps DisplayBlock.type to the TS approval_surface vocabulary... policy_name is always
    None -- Python has no policy system"), `soul/kimisoul.py:827` ("TS parity").
  - Migration is data-level (`~/.kimi/`), not source-level; no shared code tree.
- Shallow clone (`.git/shallow`): census `contributors: 1, commits: 1` is a clone artifact.
  Remote: 11,429 stars, 1,340 forks, 137 changelog versions. No dead/low-activity inference
  from HEAD was made; archived status comes from manifest + remote `archived:true`.

## Census sanity

Census `test_loc: 51,134` is sane (tests 45,172 + tests_e2e 4,621 + tests_ai/sdk). Census
`non_test_loc: 130,707` overstates the *agent* code: Python `src` = 42,052 LOC and workspace
packages (kosong 11,356, kaos, kimi-code tombstone) = ~11.4k, so core agent code ≈ 53.4k. The
balance is the web SPA TS (22.4k), generated `web/package-lock.json` (16.3k counted as JSON
code), docs Markdown (~14.5k), etc. Tier stays T2 (repo total), but core is T1-sized.

## Core loop (line-level read)

`src/kimi_cli/soul/kimisoul.py` (1,963 LOC -- the residual god file):
- `run()` 660-825: turn wrapper. ContextVar-scoped `ApprovalSource` per foreground turn with
  `cancel_by_source` in `finally` (812-817); UserPromptSubmit hook can block a turn before any
  LLM call (696-712); Stop hook with a one-retrigger cap (744-757); interrupt telemetry with
  `interrupt_reason` (max_steps / user_cancelled / error) and unconditional TurnEnd (793-824).
- `_agent_loop()` 937-1109: documented lifecycle. Step guard `max_steps_per_turn` (1004),
  auto-compaction *before* each step on `token_count_with_pending` (1023-1035), per-step
  checkpoint (1036-1037), `BackToTheFuture` exception unwinds context to a checkpoint and
  injects the D-Mail message (1060-1105), steers drained at step boundaries force continuation
  (1075-1079).
- `_step()` 1111-1347: notification delivery (root only) -> dynamic injections -> history
  normalization -> tenacity-wrapped LLM call with `_run_with_connection_recovery` (401 OAuth
  refresh-once, connection-recovery-once, re-entrant so 401-on-retry still refreshes,
  1657-1730) -> usage/status -> tool results -> `asyncio.shield`ed context growth (1297) ->
  outcome resolution: rejection-stop (root-only, feedback-aware), D-Mail, repeat force-stop,
  continue/stop (1300-1347).
- Toolset `soul/toolset.py` (1,104 LOC): PreToolUse hook gate before dispatch (459-478);
  same-step identical-call dedup that awaits the original task and copies its result
  (370-375); canonical-JSON arg keys (176-199).
Module split is genuinely good elsewhere: providers are an out-of-tree package (`kosong`,
openai_legacy/openai_responses/anthropic/gemini/kimi + echo + mock + chaos), fs/exec is
`kaos`, UIs are adapters (`ui/shell` 8.6k, `ui/print`, `ui/acp`, `wire/server.py`, `web/`),
so the loop is UI-decoupled -- but KimiSoul itself fuses loop control + compaction + flow
runner + retry policy + telemetry, which is the crush-style fusion the errata dock.

## Compaction / context (line-level read)

- `soul/compaction.py` (198 LOC): single LLM-summary strategy (`SimpleCompaction`), preserves
  the last 2 user/assistant messages verbatim (150-166), text-only summarization input (image
  parts dropped from the summary payload, 170-176), custom-instruction override (181-189),
  no-LLM early return when history too short (146-148). Trigger `should_auto_compact` 58-74:
  ratio OR reserved-context floor, whichever fires first.
- `soul/kimisoul.py:1412-1642` compact_context: PreCompact hook with trigger reason
  (auto/manual/manual-with-prompt), background active-task snapshot re-injected post-compaction
  for root (1581-1595), conservative post-compaction token estimate from `usage.output` +
  text heuristic (`compaction.py:22-56`), retry with provider recovery, compaction_failed/
  finished telemetry, PostCompact hook (1631-1642).
- **D-Mail (`tools/dmail/dmail.md` + `soul/denwarenji.py` + `context.py:137-190`):** the model
  can revert its own context to a prior `CHECKPOINT n` and carry a model-authored payload,
  with *no extra LLM call* -- a second, model-driven compaction tier. `dmail.md` is explicitly
  honest that filesystem/external state is NOT reverted. Revert rotates the journal file and
  rewrites it up to the checkpoint (`context.py:150-190`); malformed lines are skipped on
  replay (context.py:246-340), crash-tolerant by construction.
- Cache discipline: Anthropic provider stamps `cache_control` ephemeral markers on system/
  tools/tail (`packages/kosong/src/kosong/contrib/chat_provider/anthropic.py:292,339,354`),
  Kimi/OpenAI providers carry `prompt_cache_key` and subtract `cached_tokens` into
  `input_cache_read` usage (`chat_provider/kimi.py:98,425-441`); per-request adaptive
  completion budget with safety margin (`kimisoul.py:1348-1387`, CHANGELOG 1.49.0).

## Permissions / sandbox (line-level read)

- `soul/approval.py` (413 LOC): approval is tool-call-bound (`RuntimeError` outside a tool
  call, 234-236); yolo/afk/auto-approve precedence (246-286); `approve_for_session` adds the
  *action name* to the session set and retroactively resolves all pending requests with the
  same action (352-366); rejection carries user feedback into the tool result, with distinct
  anti-bypass instruction text for subagents (76-100). Full telemetry parity with the TS gate.
- `approval_runtime/runtime.py` (243 LOC): request/resolve lifecycle, multi-waiter futures
  with refcount cleanup, timeout cancellation, ContextVar `ApprovalSource` + `cancel_by_source`
  (174-194) so orphaned approvals die with their turn.
- Workspace containment: Read/Glob reject paths outside work_dir + persisted `additional_dirs`
  (canonicalized, `tools/file/read.py:86-94`, `glob.py:87-101`); writes outside require the
  distinct `edit file outside of working directory` approval action (`tools/file/__init__.py:13`);
  plan mode restricts all edits to the single canonical plan file (`tools/file/plan_mode.py:31-46`).
- Hooks: Claude-Code-shaped event vocabulary incl. `permissionDecision: deny`
  (`hooks/config.py:5-19`, `hooks/runner.py:78-81`); hooks come ONLY from the global
  `~/.kimi/config.toml` (`config.py:276-278`), so a cloned repo cannot plant hook commands --
  an accidental but real trust-scoping property.
- **No OS sandbox exists anywhere**: grep for bwrap/seatbelt/sandbox across `src` returns zero
  enforcement code; the Shell tool is approval -> `kaos.exec` of `$SHELL -c`
  (`tools/shell/__init__.py:92-104, 229-240`), foreground timeout cap 5 min / background 24 h
  (19-20). This is exactly the "tested approval, nothing underneath" 6 rung, shared with
  cline/pi/crush.

## Tests / CI (do they prove properties?)

- 211 `test_*.py` files, ~45k + 4.6k e2e LOC. Loop semantics are driven through real `KimiSoul`
  with scripted providers: repeat ladder (`tests/core/test_kimisoul_repeat.py`), steer
  (test_kimisoul_steer.py), retry recovery (test_kimisoul_retry_recovery.py), turn balance,
  think-only, completion budget incl. ChaosChatProvider fault injection
  (test_kimisoul_completion_budget.py:260,305), approval runtime/afk/telemetry, background
  agent kill (test_background_agent_kill.py), resume protocol, session fork.
- `tests_e2e/` drives the real CLI as a subprocess over the wire protocol (`wire_helpers.py`,
  12 protocol suites incl. approvals, steer, questions, errors, MCP/skills; a `--wire` real-LLM
  suite exists gated by `KIMI_E2E_WIRE_CMD`), plus PTY tests of the interactive shell
  (`tests/e2e/test_shell_pty_e2e.py`).
- CI (`ci-kimi-cli.yml`): check (ruff+pyright, Makefile:68-72), test matrix Python 3.12/3.13/
  3.14 (68), PyInstaller binary build matrix + smoke test + macOS code signing (116-176),
  version-bump + kimi-code alignment check (179-253), nix build; separate CIs for kosong/kaos/
  sdk/docs; kosong tests include `--doctest-modules` (Makefile:107).
- Gaps: no coverage gate (pytest.ini bare; `make test` has no --cov), `tests_ai/` benchmark
  smoke (Terminal-Bench-2, `tests_ai/accuracy_smoke/README.md`) is path-filtered in CI but no
  workflow runs it, no fuzzing. Per errata, verification ceiling without in-CI evals = 8.

## Orchestration / interop / operability highlights

- Subagents: `LaborMarket` type registry with per-type tool allowlists (agent.py:389-404),
  foreground runner + background agent runner share `prepare_soul` (subagents/core.py:36-46),
  resume-by-id (`subagents/runner.py:358-359`). Runtime clone for subagents shares approval
  state but gets a fresh DenwaRenji (agent.py:330-360).
- Background tasks: persistent `BackgroundTaskStore`, dedicated worker processes with
  heartbeat (5 s) and control-file polling, stale detection + `reconcile()` before notifications
  (`background/worker.py:31-40`, `manager.py:422-483`), Windows process-tree kill (21-29),
  notifications injected into the root loop (kimisoul.py:1131-1146).
- Flows: agent flows declared as D2/Mermaid fenced blocks in SKILL.md, parsed to task/decision
  graphs (`skill/__init__.py:645-660`, `skill/flow/d2.py`, `mermaid.py`), executed by
  `FlowRunner` with `<choice>` edge matching + re-prompt (kimisoul.py:1862-1963); the ralph
  loop is implemented as a synthesized 2-node self-loop graph (1800-1845). KLIP-10 specs it.
- Interop: ACP server both single-session (`--acp`) and multi-session (`kimi acp`) with
  session/load, session/list, MCP passthrough (`acp/AGENTS.md:13-49`); ACP sessions set a
  kaos backend that routes fs/exec through the IDE host's ACP fs/terminal methods
  (`acp/kaos.py:165-186,259`; `acp/session.py:159`); pykaos also ships an SSH backend
  (`packages/kaos/src/kaos/ssh.py:61,276`). Wire JSON-RPC protocol with versioned journal
  (`wire/file.py:19-30`), host-provided external tools (KLIP-12), hooks over the wire
  (`wire/server.py:26-33`); headless `--print --output-format stream-json`; published
  `kimi-sdk` (KLIP-7, release workflow); FastAPI web server + TS SPA + `web/openapi.json`;
  Toad TUI (`kimi term`) dogfoods its own ACP server (`docs/en/reference/kimi-term.md`).
- Operability: `-r` resume + picker, `/undo` `/fork` via turn-aware truncation of wire+context
  journals (`session_fork.py:1-6,24+`), resume hint printed on fatal errors
  (`cli/__init__.py:730-736`), `kimi info`, crash telemetry (`telemetry/crash.py`), keyring
  secrets, session import/export, share. No filesystem rewind -- context checkpoints are
  narrative-only, and the tool text says so.

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 7 | kosong/kaos/wire/ui separation + journal-backed Context (context.py:19-31); docked for kimisoul.py 1963 fusing loop+compaction+flows+recovery, toolset.py 1104, wire/server.py 1060, DI TODO debt (agent.py:449-457) |
| verification | 8 | scripted-provider tests of real loop semantics (tests/core/test_kimisoul_repeat.py:63+, test_kimisoul_retry_recovery.py); protocol e2e vs real subprocesses (tests_e2e/wire_helpers.py); 3-py x OS matrix + signed binaries (ci-kimi-cli.yml:68,116-176); no coverage gate, no in-CI evals, no fuzzing (errata cap) |
| safety-enforcement | 6 | tested tool-call-bound approval (approval.py:234-236, test_approval_runtime.py), session action cache (352-366), plan-mode + workspace gates; zero OS sandbox (grep empty), "Python has no policy system" (approval.py:43) |
| token-economy | 7 | dual-trigger auto-compact (compaction.py:58-74), pending-token accounting (context.py:70-74), D-Mail no-LLM fold (dmail.md + kimisoul.py:1091-1105), adaptive completion budget (1348-1387), cache_control/prompt_cache_key (anthropic.py:292-354, kimi.py:98,425-441); single summary tier, 2-msg preservation |
| orchestration | 8 | LaborMarket allowlists (agent.py:389-404), persistent bg tasks w/ heartbeat+reconcile (worker.py:31-40, manager.py:483), subagent resume (runner.py:358), flows + ralph (kimisoul.py:1787-1845), budgets at 3 levels |
| interop | 8 | ACP single+multi-session (acp/AGENTS.md:13-49), kaos-over-ACP exec routing (acp/kaos.py:165-259, ssh.py:276), versioned wire RPC + external host tools, stream-json headless, kimi-sdk + web OpenAPI; no MCP server mode |
| operability | 8 | -r/picker, /undo //fork journal truncation (session_fork.py), fatal-error resume hints (cli/__init__.py:733), crash telemetry, replay, keyring; no FS-level rewind (honestly disclosed) |
| originality | 8 | D-Mail self-revert fold; diagram-defined agent flows (D2/Mermaid -> graph) with ralph as synthesized flow; ChaosChatProvider; kaos transport abstraction incl. SSH; KLIP governance trail (klips/) |
| durability | 3 | archived:true (remote), SECURITY.md:1 no security updates, CHANGELOG 1.51-1.52 tombstone releases; Moonshot backing + 137 releases + successor exist, but this product is deliberately dead |
| docs-dx | 8 | bilingual docs site (docs/en + docs/zh, 64+ md), per-command reference incl. kimi-acp/kimi-web/wire mode, sessions/IDEs/integrations guides, setup wizard (ui/shell/setup.py), AGENTS.md, migration guide |

**Weighted total: 73.0 -> band B.** Strongest dimension: orchestration (8 on the richest
distinct mechanisms: typed subagent market + crash-aware background plane + flow graphs).
Weakest: durability (3, archived by decision, remote-verified).

## Calibration notes

- Rule (b) applies: archived/dead caps at B. Not binding (73.0 is already inside B); recorded,
  not silent.
- Rule (a): not a sync-fork; kimi-cli is the *predecessor* of kimi-code. If synthesis ranks
  kimi-code (T2, same team, active) this subject must not outrank it via nostalgia -- currently
  73.0 vs the sibling review; no conflict.
- No boundary risk: 73.0 is >2 pts from 78 and 65.
- Census corrections: non_test_loc 130,707 includes web SPA TS + generated package-lock JSON +
  docs Markdown; Python agent core is ~53.4k. contributors/commits=1 are shallow-clone
  artifacts. License/name/census tier otherwise fine.
