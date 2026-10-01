# Boundary re-review: ferrum (T1, Rust, MIT, shallow clone)

Independent recount. Provisional 63.0 C (1.5 under B floor 65); original reviewer named
verification and safety as each a 1-pt swing. I scored all ten lanes from source; findings
ferrum-b1..b8, ids from ferrum-b1, new-only (original review artifacts not read).

**Closest anchor: crush (69.0 B)** - binding non-kernel enforcement with a real test corpus
and honest docs; ferrum sits BELOW crush because (i) zero CI means the entire test corpus
never executes automatically (crush's -race 3-OS matrix runs on every push), and (ii) the
single-file fusion is larger than crush's agent.go+coordinator.go pair. Above nanocoder
(60.5) because enforcement ships ON (nanocoder's jail is opt-in and fail-open).

Activity: shallow clone (.git/shallow present); per dispatch, NO activity claims made either
way. Rule (b) not evaluated. Rule (a) n/a (original provenance; README.md:11 "inspired by
Pi's agent-harness ideas" declared, no code lineage seen).

## 1. Verification: "zero CI" claim VERIFIED. Lane = 6 (cap).

- No CI of any kind: `find . -name '*.yml' -o -name '*.yaml'` -> 0 files repo-wide; `ls -a`
  shows no .github, .forgejo, .woodpecker, .gitlab-ci; the only workflow-adjacent artifacts
  are manual scripts (scripts/package-linux.sh, scripts/check-release-docs.sh).
- Test corpus is real and property-shaped, not decorative: ~17.3k inline `#[cfg(test)]` LOC
  (brace-counted) + 1,752 LOC integration tests. Examples: deny/allow matrices across all
  three tiers (src/tools/shell_guard.rs:695-940), fail-closed resource limits
  (syntax_resource_limits_fail_closed, :868), compaction invariants
  (compaction_keeps_one_untrusted_summary_before_immutable_policy,
  compaction_split_drops_orphan_tool_results_from_recent_context, src/agent/mod.rs:7016,7160),
  loop-guard module tests (mod loop_guard_tests, src/agent/mod.rs:2590), ACP protocol tests
  spawning the real binary over stdio (tests/acp_stdio.rs:1+), faux provider
  (src/providers/fake.rs, faux-provider-testing precedent).
- 21-task bench harness with per-task validate/score scripts (bench/tasks/001..021) comparing
  ferrum vs Pi vs OpenCode (bench/README.md:3) - an eval harness wired to NOTHING.
- Per claw-code-agent precedent: with zero CI, verification maxes at 5-6 regardless of test
  LOC. ferrum takes the TOP of the cap (6) because test shape is clearly above claw-code's
  (cross-tier matrices, fail-closed tests, doc-contract test, protocol-level integration
  tests). It cannot exceed 6: nothing blocks; there is no evidence the suite is green; and
  even with CI the ERRATA 8-ceiling applies (no fuzz targets, no in-CI evals).
- Amazon-q ledger rule checked: this is NOT broken-as-configured CI (there is no gate to be
  broken); it is absent CI, which the claw-code cap governs. No double-dock: the CI absence
  is charged here, not again in safety.

## 2. Safety: grammar policy BINDS by default; its tests never execute in CI. Lane = 6.

- Machinery: tree-sitter-bash parsed policy, not regex. Fail-closed on every parse path:
  parser-unavailable / parse-failure / syntax-error / >256KB / >20k nodes / >256 depth all
  DENY (src/tools/shell_guard.rs:66-105). Deny is enforced pre-execution at every bash/wait
  entry (src/tools/mod.rs:223,237,347,363 via validate_before_permission) and the interactive
  `!`/`!!` shortcuts go through the same gate (src/agent/mod.rs:9614). write/edit enforce
  writable roots and are fully disabled at high tier (src/tools/mod.rs:240-258).
- Ships ON: `safety` defaults to Medium (src/config.rs:583) - medium rejects command
  substitution, interpreter payloads, dynamic/indirect authority, out-of-root mutation
  (docs/security.md tiers table; each row cross-checked against guard tests). Tier-
  independent checks reject `rm -rf /`, `sudo`, privilege escalation at ALL tiers.
  Project config can only tighten (config.rs:640-644 stricter_safety + floor re-enforced at
  :707-711); a malicious repo cannot loosen authority (ferrum-b3 below).
- Interaction with enforcement-tests-in-ci: the corpus doctrine (keen-code) is that tests
  RUNNING in CI is what separates 6 from 7. ferrum's enforcement tests exist in volume but
  nothing runs them - so 7 is unreachable regardless, and the enforcement itself binds in the
  shipped default, which keeps ferrum AT 6 (the "tested approval, nothing underneath" rung -
  here "tested" is static-quality, same credit keen/amazon-q got, minus CI-execution which
  was the 7-gate anyway). honesty-credit doctrine (hax): irrelevant upward - rung 4+ is
  binding machinery; the exemplary honesty ("Ferrum is not a sandbox", docs/security.md:31;
  containment.md:3 self-labeled design note; "Low is not a hostile-input boundary") earns
  docs credit, not enforcement credit.
- Nothing underneath: no seccomp/landlock/namespaces anywhere (grep: zero hits); cgroup v2 is
  used for lifecycle cleanup of process trees (src/process_containment.rs, bash.rs:101 -
  containment_errors are REPORTED, execution continues), and the docs say so honestly (docs/security.md:31).
  ACP mode defaults to `--permissions Off` (src/cli.rs:82): editor hosts get zero prompts,
  only the grammar gate (ferrum-b7, bounded nuance - policy still binds).
- No fail-open found in the policy path itself; fail-open watch family (gptme/nanocoder/
  maki/tura) NOT triggered - the cgroup soft-failure is lifecycle, not permission.

## 3. Architecture: 10,178-LOC agent/mod.rs confirmed; lane = 6.

- src/agent/mod.rs = 10,178 total, ~3,102 inline test, ~7,076 production - fuses rustyline
  completion/picker/palette machinery (:141-940), slash-command handling (:1227+), system
  prompt template (:1531), loop guard (:2498-2585), run_turn loop (:3300-3440), tool batch
  execution (:3826+), compaction (:4588, :8631). This is a >5k-LOC product/TUI file -
  ERRATA docking applies; that is why 7 (crush) is out despite the rest of the tree being
  cleanly decomposed.
- Credits keeping it at 6 (nanocoder-5 plus): single loop shared by interactive/print/ACP
  through an AgentEventSink trait (src/agent/events.rs) - no parallel loop reimplementations
  (the nanocoder-5 failure); policy fully extracted into tools/{shell_guard,shell_policy,
  write_policy}; providers/session/mcp/acp/config all separate modules; parallel-safe
  read-only tool batching (agent/mod.rs:3836-3841, 3886).

## 4. Full ten-dimension pass

| lane | score | basis |
|---|---|---|
| architecture | 6.0 | above: sink-decoupled single loop, clean module tree; docked: 7.1k-LOC production god file (errata) |
| verification | 6.0 | top of zero-CI cap (claw-code precedent); ~19k LOC property tests + 21-task bench, nothing executes them; ERRATA 8 irrelevant |
| safety-enforcement | 6.0 | grammar policy binds by default (medium), fail-closed parse; no kernel layer; enforcement-tests-in-ci gate (6->7) unreachable with zero CI; honest posture |
| token-economy | 7.0 | projected trigger = max(provider-usage projection, len/4 estimate) incl. tools+pending (agent/mod.rs:1782-1796), pre-request phase checks + FAIL-CLOSED bail if still over budget after compaction (:4376-4383), untrusted-summary prefix (messages.rs:11), usage+cache-token accounting with /usage rollups (usage.rs:156-161); single threshold 95% (:62), no cache discipline, no tiering, documented staleness discrepancy (context-accounting.md) - cline-7 rung |
| orchestration | 5.0 | loop guard nudge+force (repeated fingerprint, error streak, hard round limit, tested :2590), resume, parallel read batches; NO subagents/queue/workflow journal; background tasks explicitly deferred design note (docs/background-tasks.md:3) |
| interop | 7.0 | official ACP v1 schema crate server (acp.rs 2,583 LOC, tests spawn real binary), MCP stdio bridge (mcp.rs 1,856), Zed doc (docs/zed.md), OpenAI-compatible registry + ChatGPT/Codex OAuth (auth/openai_codex.rs), headless print mode; no published SDK |
| operability | 6.5 | JSONL journals with archived entries + history_search/history_read over pre-compaction history (agent/mod.rs:2349, jsonl.rs:782), --resume/--continue/--session, /sessions picker incl. delete, atomic file writes (atomic_file.rs), sanitized error surfacing (main.rs:30); no checkpoints/rewind, no crash-recovery posture docs |
| originality | 7.0 | grammar-policy itself is corpus-convergent (x9 prior instances) BUT ferrum's combo is distinctive: 3-tier authority ladder with table-driven doc-contract test (shell_guard.rs:1267 pins the published docs/security.md matrix), repo-config-monotone-narrowing, archived-history search tools, bench that scores against Pi AND OpenCode head-to-head |
| durability | 4.5 | solo-maintainer personal repo (Codeberg primary + GH mirror), real release engineering (semver pin script, deb/rpm, sha256, man page), external security review credited by name (docs/security.md:17); zero CI/release automation in-tree; activity evidence-limited (shallow) |
| docs-dx | 8.0 | 24 in-repo docs incl. security/containment/context-accounting/resource-boundaries/tool-authority, man page, honest known-defect documentation (context-accounting.md documents a stale-usage bug openly), install matrix; check-release-docs.sh drift-gates versions but no CI runs even that |

**weighted_total = 62.75 -> C** (needs 65.00 for B; 2.25 short).

## Verdict on the "two 1-pt swings"

Both swings were exercised and BOTH fail to reach the floor:
- verification 5->6: granted (top of cap) = +1.5 weighted.
- safety 5->6: granted (binding shipped default = rung 6, not 5; nanocoder's opt-in fail-open
  is the 5, ferrum's medium-ships-on is not) = +1.0 weighted.
Even taking both, the recount nets 62.75 because the remaining lanes (durability 4.5,
orchestration 5, operability 6.5) sit a touch under lenient-reading values. The B floor is
NOT crossed. Ferrum is a genuinely well-engineered C: the binding grammar policy and the
property-test corpus are real; the absence of any CI is the single structural fact that
caps verification, blocks the safety 7-gate, and hollows out the durability/release story.

Doctrines applied: honesty-credit (hax, confirmed verbatim - not needed here, machinery
carries), shipped-default posture (octomind/forge - scored what ships at medium/default Off
ACP), verification-8 ceiling + legitimate breakers ledger (nothing blocks -> no break),
broken-as-configured CI (amazon-q - N/A, no gate exists), enforcement-tests-in-ci (keen-code
6-vs-7 gate - unreachable), errata >5k-LOC product file docking (pi errata).
