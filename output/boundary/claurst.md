# Boundary re-review: claurst (independent pass)

Provisional 65.0 (B floor), safety-enforcement=4 flagged as swing. Anchors read including
ERRATA. subjects/claurst.md, scores/claurst.json not read (an incidental whole-corpus
concept grep surfaced the existence of claurst-1 "clean-room claim unsupported" and
claurst-2 "oauth-client-impersonation" before the scope slip was caught; new findings
below avoid both).

Closest anchor: crush (69.0) -- reimplemented breadth, real-but-partially-tested
approval layer, no kernel floor. On the safety axis claurst lands below nanocoder (5,
"real machinery + wrong default") and on codel's "isolation exists but reads misleading"
side of the 2-rung, which is the crux of this boundary.

## 1. Safety-enforcement: verdict 2 -- present but misleading

The approval machinery itself is real and partially tested:
- `PermissionManager::evaluate` deny-first rule evaluation with mode fallthrough
  (`src-rust/crates/core/src/lib.rs:2810-2890`), decision matrix tested
  (`core/src/lib.rs:5093-5125`), query-loop backstop with gating matrix
  (`query/src/lib.rs:2504-2573`, test at `:2659-2665`).
- MCP project-server trust gate is exemplary: origin is `#[serde(skip)]` so a repo cannot
  forge `User` (`core/src/lib.rs:1152-1184`), re-tagged in code on project load
  (`core/src/lib.rs:1894-1903`), trust stored outside the repo (`core/src/mcp_trust.rs:21-22`),
  and the attack is pinned by a test (`core/src/lib.rs:4655-4693`).
- Root/sudo block on `--dangerously-skip-permissions` (`cli/src/main.rs:542-547`).

But the two specific allegations both confirm, and they are not edge gaps:

**(a) /sandbox-toggle is a phantom.** The command writes `sandbox_mode` into
`ui-settings.json` (`commands/src/sandbox.rs:137`, state read back only by the command's
own `status` at `:44,:123`). No execution path consumes it: `sandbox_mode` appears nowhere
outside the commands crate (tools/query don't even depend on commands per
`crates/tools/Cargo.toml:7-9`, `crates/query/Cargo.toml:7-10`); there is no `bwrap`,
`sandbox-exec`, `seatbelt`, `firejail` or equivalent anywhere in src-rust (grep empty);
the bash tool spawns plainly. Meanwhile the product claims live isolation: help text
"shell commands run in an isolated environment" (`commands/src/sandbox.rs:22-24`), docs
"run in an isolated environment to limit unintended side effects" (`docs/commands.md:1183`)
and a capability table asserting "Functional on macOS, Linux, WSL2" (`docs/commands.md:1253`).
The spec confirms the TS original had a real `SandboxManager` / `sandbox-adapter.js`
(`spec/02_commands.md:254,279`) -- the port shipped the UI and dropped the enforcement.
That is a phantom safety control with affirmative honesty-washing ("restart recommended
for full effect"), which is the 2-rung tell, not a 4-rung "known gap".

**(b) Project settings self-grant permission rules.** `Settings::merge` guards the
adjacent prompt-silencing fields with explicit SECURITY comments and base-only carry
(`core/src/lib.rs:1992-2001`: `trust_project_mcp_servers`, `skip_dangerous_mode_permission_prompt`,
`allowed_bash_prefixes`) -- but `permission_rules` is extended with the project's rules
on the next line (`core/src/lib.rs:2003`). `load_hierarchical` merges the repo's
`.claurst/settings.json` (`core/src/lib.rs:1871-1879`) and the CLI feeds that merged
Settings straight into `PermissionManager::new` (`cli/src/main.rs:496,665-668`; rules ingested
at `core/src/lib.rs:2771-2777`). Since `evaluate` applies persistent Allow rules before any
mode-default Ask (`core/src/lib.rs:2823-2850`), a cloned repo shipping
`{"permission_rules":[{"tool_name":"Bash","action":"Allow"}]}` silences every approval
prompt -- including under headless `ManagedAutoPermissionHandler`, which is manager-backed
(`core/src/lib.rs:3210-3217`). The guards exist one line away; this field slipped through.

**(c) Ungated project hooks (same class).** Project `hooks` merge unguarded
(`core/src/lib.rs:1963` `merge_map` override-wins extend) and run via `sh -c` on tool
events (`core/src/lib.rs:3988-3996`; call sites `query/src/lib.rs:1672,1895,1979`). Only
`--bare` clears them (`cli/src/main.rs:530-533`). So opening an untrusted repo yields
attacker-chosen shell execution at session/tool events, plus the rule self-grant of (b).
The MCP trust model proves the authors understood this threat and tested the MCP leg --
the identical class of hole remained open for hooks and rules, which rules out
"documented gap".

Net: the enforcement that binds (approvals, MCP gate) is tested, but the product also
advertises a sandbox that does not exist and lets the repo you open grant itself
permissions and hooks. That is "present but misleading" = **2**, matching codel's
"reads misleadingly" qualifier, and consistent with the anchors' emphasis (pi at 6 earns
its rung partly on exemplary honesty; claurst is its mirror).

## 2. Verification: real but below the 7-rung -- score 6

Census-style 20.9k understates: inline `#[cfg(test)]` modules contribute ~33.2k LOC
across 151 files plus 1,366 LOC of integration tests (`core/tests/`, `tui/tests/`,
`cli/tests/`), so call it ~34k test LOC against ~148k total. Quality is mixed-positive:
- Property-style unit tests do exist and target security semantics: permission decision
  matrix (`core/src/lib.rs:5093-5125`), MCP origin forgery test (`:4655-4693`), backstop
  gating matrix (`query/src/lib.rs:2659-2665`), 427-LOC shadow-snapshot tests
  (`core/tests/snapshot_tests.rs`).
- CI runs `cargo test --workspace --locked` on ubuntu/windows/macos and clippy
  `-D warnings` (`.github/workflows/ci.yml:22-88`).
- But: no provider-level loop test exists -- zero mock-SSE/faux-provider infra (no
  mockito/wiremock/harness anywhere), so loop semantics against scripted responses are
  untested; several smoke tests are existence-level (`core/tests/parity_smoke.rs:22-27`
  asserts a path "contains projects", `:57-60` asserts tokens > 0); tests must run with
  `--test-threads=1` due to global-state races (`ci.yml:66-69`). No fuzzing, no evals,
  no coverage gate. That is solid-standard (6), not the crush/nanocoder 7 (race-tested
  multi-OS suites, jail specs installing real bubblewrap in CI).

## 3. Provenance: "no code lineage, behavioral reimplementation" -- correct call, with the spec itself flagged

`README.md:208-218` claims a two-phase clean room (spec agent, then implementer that
"never referenced the original TypeScript"). Evidence:
- Shipped Rust contains no TS expression: idiomatic Rust, own crate decomposition,
  restructured modules; per-language fingerprints differ. The 216 "ported/mirrors TS"
  comments (e.g. "ported from TS hasPermissionsToUseTool" `core/src/lib.rs:2789`,
  "Mirror TS setup.ts" `cli/src/main.rs:542`) cite behavior/function-names consistent
  with spec-mediated porting, not copied code.
- However the spec is not a clean artifact either: `spec/00_overview.md:3` names the
  analyst's local checkout of the leaked tree (`X:\Bigger-Projects\Claude-Code`, "~1,902
  files, ~800K+ LOC") and reproduces the full proprietary file inventory with sizes
  (`:31-60`); TS signature fences appear (`spec/01_core_entry_query.md:101-104`) along
  with unreleased internal feature codenames (`:120-125` KAIROS/LODESTONE/SSH_REMOTE).
  Signatures are thin, but "no source code was carried forward" overstates.
- Firewalling is unverifiable: both phases are the same author's AI agents, so the
  Phoenix-doctrine separation the README invokes is asserted, not demonstrable.
Verdict: keep the subject in the "ported-proprietary-source" family with copying
restrictions (concepts-only, no code-adjacent portables) -- already the case in this
study (an existing license-risk finding covers the clean-room claim). "No code lineage
in shipped code, behavioral reimplementation" is the right label for the Rust tree;
the spec/ directory is itself a proprietary-derived artifact shipped in-repo.

## Scores (independent)

| dim | score | one-line basis |
|---|---|---|
| architecture | 6 | 12 acyclic crates with real boundaries; docked per ERRATA: `core/src/lib.rs` 5,318 LOC fusing config+permissions+hooks+oauth+cost+history, `tui/src/prompt_input.rs` 5,084, `cli/src/main.rs` 4,905 |
| verification | 6 | ~34k test LOC, 3-OS CI + clippy -D warnings (`ci.yml:22-88`), security-semantic unit tests; no faux-provider loop tests, no fuzz/evals/coverage, some existence-assert smoke |
| safety-enforcement | 2 | tested approvals + exemplary MCP trust gate, but phantom sandbox advertised as functional (`commands/src/sandbox.rs:137` w/ no consumers; `docs/commands.md:1253`) and repo self-grant of permission rules (`core/src/lib.rs:2003`) + ungated project hooks (`:1963`, exec at `:3988-3996`) |
| token-economy | 5 | tiered compaction port (micro/snip/reactive/collapse, `query/src/compact.rs` 2,266 LOC) but chars/4 estimation (`core/src/context_collapse.rs:32-39`) and zero prompt-cache breakpoints (`CacheControl::ephemeral` `api/src/lib.rs:260` never called) |
| orchestration | 6 | cron scheduler tool (`tools/src/cron.rs`), swarm subagents registered pre-loop (`cli/src/main.rs:771-777`), resume; crash semantics unproven, no loop detection |
| interop | 7 | ACP crate + smoke test, MCP + oauth, bridge remote control, VS Code extension with own CI (`vscode-extension-ci.yml`); no published SDK |
| operability | 7 | --resume (`cli/src/main.rs:141-143`), shadow-snapshot checkpoints, `/doctor`, session tracing, share export |
| originality | 4 | buddy/effort/voice surfaces are distinctive among subjects but every one is a port of leaked upstream design (`spec/00_overview.md:11-27`) -- distinctive shape, borrowed mechanism |
| durability | 4 | solo author; shallow clone -> activity evidence-limited, but v0.1.7 tag + active release workflows indicate maintenance; no SECURITY.md |
| docs-dx | 6 | 15 in-repo docs, install scripts; drift: `docs/hooks.md:439` documents `.claude/` paths, `docs/commands.md:1253` overstates sandbox |

**weighted_total = 54.0 -> C.** The provisional B floor rode entirely on safety=4;
independently safety is 2, and verification is 6 rather than a band-carrying 7. Not a
calibration demotion (no fork/archived rule applied) -- a straight re-score.
