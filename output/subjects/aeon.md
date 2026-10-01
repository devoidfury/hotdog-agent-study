# aeon -- T1 review

Subject: /data/samples/agents/aeon (aeonfun/aeon, MIT, manifest `aeon-root`).
Census: 78,720 non-test / 5,340 test LOC, 71 contributors, 1,330 commits, head 2026-09-24, not shallow, provenance "original". Census commit/contributor counts verified exactly (`git rev-list --count HEAD` = 1330; `git shortlog -sn` = 71).

## Shape (read this before the scores)

Aeon is not an interactive coding-agent harness. It is a **scheduled agent framework that dispatches every run through one bash contract to nine rival coding-agent CLIs** (Claude Code, Codex, Grok, Kimi, Pi, Vibe, fx, Cursor, Hermes) on GitHub Actions, keeping state in git-committed files (`llms.txt:3`). There is no in-process agent loop, no interactive session, and deliberately no compaction: each skill is a one-shot prompt. The "core loop" for rubric purposes is `.github/workflows/aeon.yml` (2,231 lines: the skill runner), `scheduler.yml` (522), `chain-runner.yml` (716), plus `harness-adapter/` (~1,700 LOC bash) and ~90 helper scripts. The `skills/` tree (85 skills, 20.8k MD) is prompt payload, not engine. Per dispatcher instruction I scoped the harness core; peripheral apps (dashboard Next.js operator console, mcp-server, webhook Cloudflare Worker) are noted in provenance and only spot-read.

## Anchor question

Closest anchor: **crush** -- shared defensive-ops culture with a tested helper corpus and real enforcement machinery, but aeon's queue/schedule-driven shape has no true anchor; it shares codel's "no agent loop in this codebase" gene, and the entire gap between 23 and 69-plus is what aeon builds around that shape (enforcement, contracts, recovery) that codel never did.

## Loop read (workflow `aeon.yml`)

- Entry: `workflow_dispatch`/`workflow_call`/`issues:labeled`; `permissions:` block scoped (contents/id-token/attestations/prs/issues:write, actions:read) `aeon.yml:99-105`.
- Concurrency group keyed by skill+`inputs.var` with a postmortem comment: the old skill-only key silently cancelled 8 of 10 same-skill/different-target dispatches `aeon.yml:118-130`.
- Prompt built from `skills/<name>/SKILL.md` + optional chain-context file, path-contained to `output/.chains/` with explicit `..` rejection (fixes **GHSA-cqvj-gr78-24mc**, cited inline) `aeon.yml:987-1003,1008-1012`.
- Execution: one `run-harness` call per non-claude harness `aeon.yml:1126-1147`; claude runs behind a 9-provider gateway cascade with per-attempt failover and a lesson-hardened stderr surfacing (`tail -c 4000`, not 160) `aeon.yml:1046-1092`.
- Post-run: envelope usage parsed (incl. cache read/creation) `aeon.yml:1149-1166`; egress audit artifact `aeon.yml:1168-1186`; read-only capability guard `aeon.yml:1879-1921`; attestation gate/manifest/attest `aeon.yml:1364-1470`; cron-state ledger update `aeon.yml:2012+`.

## Dispatch contract (harness-adapter)

- `run-harness` exposes a Claude-Code-shaped headless contract (flags mirror claude: `--allowed-tools`, `--mcp-config`, `--json-schema`, `--max-turns`) with a strict stdout envelope `{result, usage{...}, session_id?, total_cost_usd?}` and exit-3-on-unparseable `run-harness:1-30,160-177`. Envelope validated, never partial-as-success `envelope.sh:1-9,29-40`; raw output rejected via `wrap_raw_output` `envelope.sh:42-54`.
- Structured output on schema-less harnesses: prompt+pragmatic-validate+one retry, with a JSON-recovery ladder (fences anywhere, widest brace span) that self-documents the failure modes it fixes `schema-retry.sh:1-47`.
- Per-harness capability manifest auto-generated from `rh-meta` blocks in each adapter into `harnesses.json`, explicitly framed as "the local analog of a UHP GET /v1/harnesses discovery response" `adapters/claude.sh:5-16`, `harnesses.json:1-10` (parity gated in CI: `ci-harnesses-json.yml`).
- MCP translation is the most field-notes-heavy interop code I've seen: `.mcp.json` `${VAR}` expansion that leaves unset vars literal and reports them `mcp-translate.sh:11-19`; codex `-c` TOML overrides with the JSON-object-vs-inline-table trap measured on codex-cli 0.144.6 `mcp-translate.sh:21-45`; a sandbox-layer overlay that fixes kimi auto-discovering the un-expanded project `.mcp.json` and sending `Bearer ${MCP_GLIM_TOKEN}` verbatim `sandbox.sh:16-29`.

## Compaction / context

No compaction, by design: one-shot runs. Context management = chain-context file injection (bounded, `aeon.yml:987-1012`) + git-committed memory (`memory/logs/`, `cron-state.json`, topics) -- the strongest memory-files usage in the corpus so far, but that is application state, not context-window engineering. Cost visibility is genuinely strong: per-run cache_read/cache_creation parsing `aeon.yml:1152-1155`, token CSV per model with a comment correcting a bug where adapter harnesses filed tokens under a claude id that never ran `aeon.yml:1117-1124`, gateway provider selection priced on cached-read economics `scripts/llm-gateway.sh:312-355`.

## Permission / sandbox code

Three-layer model, single-sourced: (1) tool-grammar allowlist per capability tier `scripts/skill_mode.sh:1-21,37-70` -- read-only base deliberately excludes `gh` as a write vector with documented degraders `skill_mode.sh:31-36`; (2) dispatcher-side OS sandbox: bwrap ro-binds the workspace (memory/ + output/ re-bound rw) around *every* harness because "no harness enforces read-only usefully on its own" -- with per-harness measurements (grok's `--sandbox read-only` silently ignored on 0.2.101; kimi/vibe/pi ship none; claude's `--allowedTools` sidestepped by shell redirection; nesting codex's landlock inside bwrap broke file access, so codex's self-sandbox is deliberately disabled under the wrapper) `lib/sandbox.sh:1-15,47-67`, `run-harness:133-157`; (3) post-run revert guard: `git checkout --` + `git clean -fd` over a CODE_PATHS list, with a comment admitting the earlier list missed paths so an untracked file elsewhere survived the guard and got committed `aeon.yml:1913-1921`. CI installs bwrap AND repairs the AppArmor userns restriction it observed breaking it, self-testing afterwards `aeon.yml:303-322`. Fail-open edge: if no OS sandbox exists, `run-harness` warns "read-only is advisory" and continues `run-harness:150`.

Secrets hygiene is best-in-corpus: an explicit allowlist blob `ALL_SECRETS` (deliberately not `toJSON(secrets)` -- GitHub's scanner holds runs that serialize the whole store) `aeon.yml:679-692`; per-skill `requires:` resolution injects only declared keys `aeon.yml:966-980`; blob `unset` before the agent starts `aeon.yml:982`; notify channel tokens removed from the run env entirely -- `./notify` is a queue writer, delivery is post-run `aeon.yml:748-753`; `./secretcurl` keeps secrets off the agent's command line `aeon.yml:880-884`. Metadata/link-local egress blocked with a self-verification curl `aeon.yml:195-204`. Two patched GHSA-referenced holes show the loop works. Counterweight: default tier is `write` (`skill_mode.sh:12`), network stays open by design ("read-only is about the repo, not egress" `sandbox.sh:10-12`), and `GITHUB_TOKEN`/`GH_GLOBAL` stay in-run -- their own comment calls this "deferred, broad gh surface" `aeon.yml:752-753`.

## Scheduler / orchestration

- `*/5` cron with a **debt-ledger catch-up model**: GitHub delivers only ~10% of schedule ticks, so `cron-due.sh` treats last-dispatch as a ledger and fires any slot missed within `CATCHUP_HOURS=12`, which doubles as max-lateness so a daily slot can never double-fire `scheduler.yml:5-16,63-72`, `scripts/cron-due.sh:10`.
- Auto-recovering circuit breaker (closed/open/half-open probe) as the single source of the decision, called by the scheduler, tested `scripts/breaker.sh:1-49`, `scripts/tests/test_breaker.sh`.
- Chain runner: correlation-ID dispatch (`chain-<32hex>`), run discovery by expected title, conditional step routing via a deliberately tiny `when:` grammar that fails loudly on non-integer ordering `scripts/chain_when.sh:1-21`, `chain-runner.yml:89-125`.
- Self-healing loop: skill-health -> issues -> skill-repair with 24h per-skill cooldown and 3 PRs/day cap `docs/CORE.md` (skill-repair section); aeon-doctor as a static config linter for the silent-misconfig class `aeon.yml (aeon-doctor entry), docs/CORE.md`.

## Verification

64 files in `scripts/tests/` + 12 dashboard `*.test.ts` + webhook tests; ci-tests runs 63 test invocations, path-scoped triggers, concurrency cancel `ci-tests.yml:5-38,46+`. Tests assert behavior (breaker transitions, chain_when grammar, envelope contract, readonly-guard commit shape, adapter outputs). Drift gates everywhere: capabilities-parity, skills-json vs README catalog, agents-md generation, harnesses-json, shellcheck for all bash `ci-*.yml`. Gaps: the 2,231-line workflow's inline bash is itself untested (the tested corpus is precisely what was extracted from it); no mock-provider end-to-end of a real skill run; no evals-in-CI; dashboard/webhook TS tests thin (~1.5k LOC).

## Verification of census / provenance

- LOC correction: cloc total code 84,030 but that includes 28,205 Markdown (skill prompts + docs) and 20,870 JSON (catalog, eyebrowlock). Executable source (TS+JS+shell+YAML+Python) is ~34k; test corpus by `wc -l` is ~8,971 lines vs census `test_loc: 5,340` (~40% undercount, same family as the anchors' glob misses). Tier unchanged (T1 either way).
- Provenance: original per census, consistent (self-contained manifest, remote aeonfun/aeon, no fork markers). aeon is itself the canonical upstream for its fork-fleet (aeon.yml header comments "Ported from derivative instances (PR #219)") -- a *distributed* derivative model, not a derivative itself. Rule (a) N/A. Identity hygiene: no similarly named subject in the corpus.
- Peripheral apps noted per dispatcher instruction: `apps/dashboard` (Next.js operator console: secrets vault, strategy/soul builders, MCP OAuth capture -- lib/ TS with its own tests), `apps/mcp-server` (exposes every skill as an `aeon-<slug>` MCP tool running the identical SKILL.md prompt -- `apps/mcp-server/README.md:1-14`), `apps/webhook` (Cloudflare Worker instant-Telegram mode, replay-guard test), `apps/cli` (operator CLI, 35-line launcher + TS commands). Not scored beyond their evidence above.

## Scores

| dim | score | best evidence |
|---|---|---|
| architecture | 6 | disciplined single-source extraction (skill_mode.sh, breaker.sh, cron-due.sh "no inline copy" `breaker.sh:17-19`) but the loop lives in 2,231-line YAML with a ~500-line inline Run step `aeon.yml:675-1166` -- no unit-verifiable core module |
| verification | 6 | 64 test files + 63 CI invocations, behavior-asserting `ci-tests.yml:46-173`, `test_breaker.sh`; nothing tests the workflow's own inline logic; no evals/fuzzing (8-rung ceiling not even reached) |
| safety-enforcement | 7 | measured dispatcher-side bwrap over all nine harnesses `sandbox.sh:1-15` + `aeon.yml:303-322`; three-layer read-only incl. post-run revert `aeon.yml:1879-1921`; least-privilege secrets `aeon.yml:679-692,966-982`; metadata egress block `aeon.yml:195-204`; docked for write-by-default `skill_mode.sh:12`, open network, broad GH token in-run `aeon.yml:752-753`, advisory fail-open without bwrap `run-harness:150` |
| token-economy | 5 | no compaction (one-shot by design); strong cost/cache visibility `aeon.yml:1152-1155`, priced provider cascade `llm-gateway.sh:312-355`; no context-window engineering at all |
| orchestration | 8 | debt-ledger cron for GH's ~10% tick delivery `scheduler.yml:63-72`; auto-recovering breaker, tested `breaker.sh`; chain routing + correlation IDs `chain_when.sh:1-21`; self-heal loop with cooldown/caps |
| interop | 8 | nine-CLI adapter with generated capability manifest `harnesses.json:1-10`; MCP translation per harness `mcp-translate.sh`; own MCP server `apps/mcp-server/README.md`; Claude Code/Codex plugins `llms.txt:70-74`; Telegram command surface, owner-gated fail-closed `messages.yml:85-123`; no ACP, no published SDK |
| operability | 8 | operator CLI + dashboard + aeon-doctor linter; attestation-verifiable runs `aeon.yml:1420-1470`; audit JSONL outside the sandbox `aeon.yml:886-897`; Langfuse/OTEL tiers `aeon.yml:723-740`; run-scoped sidecar logs `aeon.yml:1816` |
| originality | 8 | dispatcher-side sandbox uniformity with per-harness measurements; eyebrow content+egress lockfile anti-rug-pull `docs/skill-integrity.md:18-28`; artifact-attested skill runs + proof re-runs on PR head `aeon.yml:1458-1470`; debt-model cron. Not 9-10: ideas are ops-shaped, and the agentic core is delegated, so "field-defining for harnesses" can't quite carry |
| durability | 6 | active (head 2026-09-24, commits through the last day), 71 contributors, 17 CI workflows, MIT, company-backed; but <18 months public, no SECURITY.md, commit cadence partly agent-generated so true bus factor unclear |
| docs-dx | 8 | 22 in-repo docs incl. CORE/CAPABILITIES/attestation/skill-integrity; llms.txt; setup guide; capability taxonomy locked with rationale `docs/CAPABILITIES.md:9-14`; code comments are a masterclass in incident postmortems |

**Weighted total: 69.0 -> band B.** (6*1.5 + 6*1.5 + 7 + 5 + 8 + 8 + 8 + 8 + 6*.5 + 8*.5)
Strongest dimension: interop (with orchestration tied at 8; interop named because no other subject in the corpus has a heterogeneous-harness portability layer that doubles as the enforcement point). Weakest: token-economy (5).

## Calibration notes

- Not within 2 pts of any band boundary (69 vs 65/78). No calibration rule fires.
- Score is crush-adjacent numerically but via a different route: aeon beats crush on safety/interop/originality, loses on verification depth and token-economy. B, upper half.
- Sanity against distribution: this rubric is calibrated for interactive harnesses; a scheduled-dispatch framework scores structurally low on compaction/loop dimensions. Scoring the honest shape rather than inflating for effort; 69 reflects "excellent ops engineering around a deliberately absent agent loop."
