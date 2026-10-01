# openhands — T2 deep review

## Identity (the census question)

Manifest `@openhands/agent-canvas` v1.24.0 (package.json:2) is honest. **This is not the classic
OpenHands Python agent, and it is not the old monorepo's `frontend/` directory either.** The
remote `All-Hands-AI/OpenHands` redirects (GitHub API, verified 2026-09-29) to
`OpenHands/OpenHands` — the flagship 89k-star repo — which *itself is now* Agent Canvas: a pure
TypeScript React Router + Electron control-center. The classic Python app's runtime moved to
`OpenHands/software-agent-sdk`; the agent runtime this UI drives is `openhands-agent-server`
1.49.6, **out-of-tree**, launched via `uvx` at a pinned version (config/defaults.json:3-6,
bin/agent-canvas.mjs:3-11).

Implication for the T2 mandate: the agent core loop, compaction, and action-level permission
enforcement **do not exist in this tree**. docs/architecture.md:7-18 states this explicitly as
non-responsibilities ("not responsible for: executing agent actions... providing the sandbox").
I line-read what the tree actually contains instead: the event-stream transport (the shell's
loop substitute), the confirmation-policy assembly, and the security/auth plumbing, and scored
only in-tree evidence. A separate review of software-agent-sdk would be needed for the harness
core itself; synthesis must not read these scores as OpenHands-agent scores.

## Census sanity

- Shallow clone (1 commit, HEAD 47a1080 "feat(settings)... (#16426)", 2026-09-25). The census's
  `contributors: 1, commits: 1` is a measurement artifact. Remote-verified active:
  `pushed_at 2026-09-29`, not archived, MIT.
- **test_loc wrong (undercount).** Actual: `__tests__/` 695 files / 168,497 LOC (cloc-inflated wc:
  wc -l total), `tests/e2e` 46 files / 14,671, colocated `src/**/*.test.*` 64 files / 12,142
  → **~195.3k test LOC, not 149,921**. Census test glob missed one or more roots.
- non_test_loc 205,856 overcounts: repo total code ≈ 355k; tests ≈ 195k → non-test ≈ 160k, of
  which **42,502 LOC is a single generated blob, src/i18n/translation.json** (i18n build
  artifact; i18next-http-backend loads real locales at runtime).
- Provenance: `original` stands. Not a fork; no rule (a) exposure. Org rename only.

## Anchor question

Closer to **cline** than to codex/pi: like cline's 8-rung "SDK-first core consumed by hosts",
this is a product surface that consumes an out-of-process core — but more radically, the core is
entirely another repo, so this tree is a pure shell; verification discipline (faux-provider +
real-provider E2E in CI) resembles cline's test-heavy culture more than pi's in-tree loop.

## Mandatory-read notes (what substitutes for loop/compaction/permission)

### Event-stream core ("the loop" of the shell)
`src/contexts/conversation-websocket-context.tsx` (1,278 LOC) is the closest analogue. Real
engineering, with named failure modes in comments:
- REST-first history preload, then WS subscribe with `resend_mode='since'` anchored to the
  latest REST timestamp (:395-417); the gate is deliberately on `isPending` not `isFetching`
  because a refetch→teardown→reconnect loop stuck conversations at "Connecting" (:461-483).
- Streaming deltas coalesced to ≤1 store commit per frame via `createStreamingDeltaBatcher`
  (:165-182; src/utils/streaming-delta-batcher.ts); main vs planning streams can never merge.
- Conversation-switch store reset as a single atomic `set` in a layout effect, with a comment
  explaining why a parent's passive effect would wipe freshly seeded history (:300-320).
- `sendMessage` falls back to REST queue when the socket is closed (durable send, :1122-1180)
  and **refuses to fall back to the parent conversation in plan mode** — "exactly the boundary
  plan mode exists to enforce" (:1136-1147).
- Metrics: aggregates `usage_to_metrics` across arbitrary usage ids including the `condenser`
  key, with cache_read/cache_write tokens and `max_budget_per_task` (:213-276) → rendered in
  src/components/features/conversation/usage-panel/usage-panel.tsx:79.
Behavior spec `__tests__/contexts/conversation-websocket-provider.behavior.test.tsx` (2,216 LOC)
asserts properties, not existence: "retains a newer failed identical prompt when old history
reloads" (:366), "waits for the first history load before opening the main socket" (:458),
"clears conversation-owned stores once per conversation identity" (:549).

### Compaction
Absent in-tree; delegated. The only trace is UI accounting of the backend condenser's spend
(ws-context :217-219 comment) and `LLMContextWindowExceedError` handling in the mock LLM
(tests/e2e/mock-llm/scripts/mock-llm-server.py:25). Scored as absent-for-this-tree.

### Permissions / safety plumbing (in-tree parts)
- Confirmation-policy composition: agent-server-adapter.ts:701-714 — `confirmation_mode !==
  true` → `NeverConfirm` (**the default is no approval**), `security_analyzer: "llm"` →
  `ConfirmRisky{threshold:"HIGH", confirm_unknown:true}`, else `AlwaysConfirm`; analyzer
  selection (LLM/pattern/policy_rail) at :717-729. Tested in
  __tests__/api/agent-server-adapter.test.ts. Enforcement itself is out-of-tree.
- Auth: `X-Session-API-Key` local / bearer cloud (src/api/backend-registry/auth.ts:9-21);
  `--public` mode keeps the key out of the bundle and gates the UI behind key entry
  (bin/agent-canvas.mjs:66-74, 105). Persisted dev keys written `{ mode: 0o600 }`
  (scripts/dev-safe.mjs:167). Electron: `nodeIntegration: false, contextIsolation: true`
  (electron/main.mjs:354-355, 394-395).
- MCP secret hygiene: tool-result and error text scrubbed of configured secrets before render
  (src/api/mcp-service/mcp-service.api.ts:76-111), 747-line spec (mcp-redacted-credentials.test.ts).
- Honest boundary docs: docs/SELF_HOSTING.md:4-8 "Anyone who can talk to the agent server can
  [read/write FS, exec, network]. Treat the VM as you would any machine that holds production
  credentials"; layered-defense enumeration :47-56. specs/canvas-extensions.md decision 2:
  "Extension code is trusted, same-realm code... never a security boundary"; decision 3:
  install≠enable consent invariant, explicitly *not* proof of human presence against an agent
  holding the same APIs.

## Verification

- CI (ci.yml, PR + main): lint (typecheck+eslint+prettier), vitest, build app+lib, npm pack
  check, ubuntu+windows matrix (:61-81).
- **Live E2E against a real agent-server + real LLM on PRs** (:106-295): credential-gated
  (:153-163), skipped for fork PRs so secrets never reach untrusted code (:149-151), posts PR
  comment + video artifacts (:217-299). Asserts expected reply/bash tokens with a live model
  (tests/e2e/live/real-agent-server-conversation.spec.ts:1-18).
- **Faux-provider full-stack E2E on every push to main**: scripted OpenAI-compatible server
  powered by SDK TestLLM, including injected auth/rate-limit/context-window faults
  (mock-llm-server.py:1-30,60+), driving the real agent-server via uvx against the built
  frontend across conversations/settings/files/mcp/skills/automations/backends/regressions
  (.github/workflows/mock-llm-e2e.yml; ~5k LOC of specs), plus a Docker-image variant gated on
  main builds (mock-llm-docker-e2e.yml:11-16) and 850-LOC backend cross-connect spec.
- Mutation testing (stryker.config.mjs, `test:mutation:diff`) and an affected-E2E router
  (tests/e2e/mock-llm/test-mapping.json + resolve-affected-tests.mjs) exist but **no workflow
  invokes either** — quality tooling off the critical path.
- Per the anchor errata, 9+ requires evals-in-CI or fuzzing evidence: mock-llm-e2e.yml:90-134
  (scripted-trajectory runs with token assertions) and ci.yml:265-276 (live-model runs) are
  workflow file:line evidence of eval-shaped CI, though unscored/smoke-grade. This is the only
  subject in the corpus so far to clear that bar in both scripted and real-provider form.

## Orchestration / interop / operability highlights

- Child conversations launched by agent events: `LAUNCH_CHILD_CONVERSATION_TOOL_NAME`, isolation
  levels, cloud sandbox poll (3s cadence, 180s ceiling), and a localStorage launch ledger that
  survives corruption ("a corrupt ledger must not block the launch")
  (src/services/child-conversation-launch.ts:22-40, 206-215).
- Planner runs as a *separate conversation with its own WebSocket* (:636-656 + dual sockets).
- Automations: cron/webhook scheduling is the out-of-tree `openhands-automation`; in-tree it is
  a manifest-driven surface — nav, copy, endpoints, import/export envelope all come from the
  pinned `@openhands/extensions` manifest, and with no admitted manifest "there is nothing to
  serve: nav entries do not render and the routes 404" (src/manifests/automation-interface.ts:1-14).
- ACP: drives Claude Code / Codex / Gemini CLI via agent-server-spawned ACP subprocesses
  (docs/ACP_AGENTS.md:1-30); provider list mirrored from the SDK registry; examples/acp-docker
  + live-acp docker harness for containerized ACP agents.
- Multi-backend registry: local/remote/cloud backends, per-backend agent choice, health store,
  version-compat gating with `MINIMUM_COMPATIBLE_AGENT_SERVER_VERSION` and 401 sentinel
  (src/api/agent-server-compatibility.ts:19-27).
- Embeddable: `build:lib` with subpath exports (browser/conversation/files/settings/sidebar/
  terminal/i18n) — the UI is itself a component library (CHANGELOG.md:16-24).
- Local stack supervisor (scripts/dev-safe.mjs, 1,261-LOC own spec __tests__/scripts/dev-safe.test.ts):
  shutdown-hook registry (:606-631), SIGINT/SIGTERM/SIGHUP handling (:1050-1057), port leases so
  parallel stacks coexist (:1166), stable persisted keys, file logging (scripts/logger.mjs).
- Self-update detection via npm dist-tag (src/api/agent-canvas-updates.ts:1-14); helm chart,
  Docker, Electron (3-OS desktop workflows), telemetry with consent banner + DO_NOT_TRACK.

## Scores (against anchor ladder; shell-boundary caveats in calibration notes)

| dimension | score | best evidence |
|---|---|---|
| architecture | 8 | api/hooks/stores layering + enforced typed-client boundary (src/api/no-direct-agent-server-calls.test.ts:32-40 bans ad-hoc axios/fetch outside allowlist); architecture.md:7-18 stated non-responsibilities; largest file 1,701 (agent-server-adapter.ts), WS provider 1,278 with behavior spec |
| verification | 9 | live-LLM E2E on PRs (ci.yml:265-276, fork-gated :149-151) + faux-provider full-stack E2E on main (mock-llm-e2e.yml:90-134, mock-llm-server.py:1-11); ~195k test LOC incl. property-level behavior specs (:366, :458, :549) |
| safety-enforcement | 5 | policy assembled in-tree, enforced out-of-tree; default `NeverConfirm` (agent-server-adapter.ts:704-706); real hygiene that does bind: 0600 keys (dev-safe.mjs:167), electron isolation (main.mjs:354-355), MCP secret scrubbing (mcp-service.api.ts:76-111); exemplary honesty (SELF_HOSTING.md:4-8) |
| token-economy | 4 | no in-tree compaction or cache discipline; strong cost visibility only: usage/cost/cache-token/budget aggregation incl. condenser spend (ws-context:213-276, usage-panel.tsx:79), llm-balance-service; frame-batched deltas = render economy |
| orchestration | 6 | child-conversation launches w/ isolation + ledger (child-conversation-launch.ts:206-215); durable send queue fallback (ws-context:1122-1180); automations UI is manifest-driven while the scheduler is out-of-tree |
| interop | 8 | ACP-brokered Claude Code/Codex/Gemini (docs/ACP_AGENTS.md:13-30 + live-acp), MCP config/health/redaction, multi-backend registry w/ cross-connect spec (850 LOC), lib build + docker/helm/electron surfaces; no MCP server, no headless CLI contract in the shell |
| operability | 7 | backend health/switching, version-compat gating, draft persistence, self-update detection, supervisor leases/shutdown hooks, file logs; no checkpoint/rewind in tree (server-side); service-death = teardown, no restart |
| originality | 8 | agent-driven UI tool (`canvas_ui_control`: server-side executor is a no-op, frontend dispatches — tools/canvas_ui_tool.py:14-18); manifest-gated product surfaces; consent-split extension model (specs/canvas-extensions.md:24-40); DefenseClaw guardrail integration doc; agent-broker control-center shape |
| durability | 9 | funded org, 89k stars, pushed 2026-09-29 (remote-verified), MIT, release-please + OIDC npm publishing + 3-OS desktop + docker + helm (ci.yml, npm-publish.yml, desktop-*.yml, docker.yml) |
| docs-dx | 8 | architecture.md boundary statement, SELF_HOSTING.md threat model, ACP_AGENTS.md, TESTING_MATRIX.md install×OS×agent grid, annotated .env.sample, `--info` CLI (bin/agent-canvas.mjs:31-52), specs/ dir; docked for posthog key in defaults.json noise and partial external-docs split |

Weighted: 0.15·8 + 0.15·9 + 0.10·5 + 0.10·4 + 0.10·6 + 0.10·8 + 0.10·7 + 0.10·8 + 0.05·9 + 0.05·8 = **72.0 → B**

Strongest dimension: verification (9). Weakest dimension: token-economy (4).

## Provenance

original; flagship-repo pivot verified (All-Hands-AI → OpenHands org rename; TS primary language,
89,472 stars, pushed 2026-09-29 via GitHub API). Not a fork; calibration rule (a) n/a. Shallow
clone honored: no activity claims from HEAD alone.

## Calibration notes

- **Shell-subject caveat (important for synthesis):** scores describe a control-center shell.
  Loop/compaction/action-policy quality of the actual OpenHands agent lives in
  software-agent-sdk (out-of-tree, pinned 1.49.6) and earned zero credit; conversely the
  tree is not punished for its core's flaws. Comparing this 72.0 against full-harness subjects
  (codex 88.5, cline 76.5) is apples-to-oranges; recommend synthesis treat "agent-in-tree
  presence" as a covariate.
- Census corrections: test_loc 149,921 → ~195.3k (missed ~45k across `__tests__/`+colocated
  `src/**/*.test.*`); non_test_loc 205,856 → ~160k incl. a 42.5k-line generated i18n blob.
  contributors/commits = 1 are shallow-clone artifacts.
- verification 9 rests on workflow-file evidence for both scripted (mock-llm-e2e.yml) and
  live-provider (ci.yml:265) model-in-CI runs; they are acceptance smokes with token
  assertions, not scored evals — if synthesis deems that insufficient for the errata bar,
  demote to 8 (total 70.5, same band).
- No band-boundary risk (72.0 vs 78/65 cut).
