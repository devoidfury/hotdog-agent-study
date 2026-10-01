# Wave 6 - Concept Ledger

Built from all 1668 finding records in scope (24 T3 merged runs incl. workground2 + mistral-vibe, 80 findings files, 25 boundary files) = 1634 unique records (the 34-copy gemini-cli duplicate in output/findings/ excluded; the T3 run-dir copy is counted) minus 64 records with no concept field (see normalization-rules.md rule 11). 1570 concepted records, **712 distinct concept ids** before merging.

## Proposed merge groups (DO NOT rewrite source files; apply at synthesis)

Canonical pick rule: seeded canonical id if present, else the member with most records; names in *italics* are fresh labels for groups whose members are all presence/absence variants.

| strength | canonical | members (records/subjects) | merged total | note |
|---|---|---|---|---|
| strong | prompt-cache-discipline | prompt-cache-discipline(3/3); prompt-cache-key-discipline(1/1); no-client-prompt-cache-discipline(1/1); prompt-cache-economy(1/1) | 6/6 | all describe general prefix-cache discipline; marking (breakpoint placement) stays separate as prompt-cache-marking |
| strong | prompt-cache-warming | prompt-cache-warming(6/4); no-cache-warming(1/1) | 7/5 | presence/absence pair of the same design |
| strong | crash-recovery-posture | crash-recovery-posture(1/1); crash-recovery-middleware(10/10); crash-posture(1/1) | 12/12 | crash-recovery-middleware (11 recs) is crush's instantiation; crash-recovery-posture is the general concept -- pick one name and keep the middleware nuance in one_why |
| strong | *durable-queue* | durable-queue-dispatch(1/1); no-durable-queue(4/4); no-durable-queue-or-journal(1/1); no-queue-no-journal(1/1) | 7/7 | presence/absence variants of durable in-flight work queue |
| strong | interop-surface-breadth | inherited-interop-surface(1/1); interop-breadth(2/2); interop-surface(1/1); interop-surface-breadth(1/1); surface-breadth(1/1) | 6/6 |  |
| strong | headless-contract | headless-and-sdk-contract(1/1); headless-cli-contract(1/1); headless-contract(2/2); ndjson-headless-contract(1/1); wire-compatible-headless-contract(1/1) | 6/6 | variant qualifiers (ndjson, wire-compat, sdk) belong in one_why |
| strong | rival-config-import | rival-config-import(1/1); rival-config-ingestion(1/1); rival-config-surface-import(1/1); rival-plugin-import(1/1); rival-state-import(1/1) | 5/5 |  |
| strong | foreign-session-import | foreign-session-import(1/1); foreign-session-ingest(1/1); foreign-transcript-import(1/1); rival-harness-session-import(1/1) | 4/4 |  |
| strong | dual-session-abstraction | dual-session-abstraction(2/2); dual-session-engines(1/1) | 3/3 |  |
| strong | god-object-config | god-config-object(1/1); god-object-config(1/1) | 2/2 | word-order twins |
| strong | *mcp-client-only* | mcp-client-only-autoenable(1/1); mcp-client-only-no-acp(1/1); mcp-client-only-stdio-only(1/1); mcp-client-only-unverifiable-sdk(1/1) | 4/4 | all = MCP client without server/ACP surface, nuance differs |
| strong | three-transport-mcp-client | mcp-client-three-transports(1/1); three-transport-mcp-client(1/1) | 2/2 |  |
| strong | approval-only-enforcement | approval-only-enforcement(2/2); approval-only-shell(1/1) | 3/3 |  |
| strong | fork-cache-alignment | cache-safe-fork(1/1); fork-cache-alignment(2/2) | 3/3 |  |
| strong | default-on-telemetry | default-on-telemetry(2/2); telemetry-default-on-framing(1/1) | 3/3 |  |
| strong | docs-drift-gates | docs-drift-gate(1/1); docs-drift-gates(1/1); openapi-drift-gates(1/1) | 3/3 | openapi variant nuance in one_why |
| strong | module-size-ratchet | module-boundary-ratchet(1/1); module-size-ratchet(2/2) | 3/3 |  |
| strong | recursive-subagent-spawn | recursive-subagent-spawn(2/2); subagent-spawn-machine(1/1) | 3/3 |  |
| strong | acp-bidirectional | acp-bidirectional(1/1); acp-bidirectional-host(1/1) | 2/2 |  |
| strong | classifier-model-routing | classifier-model-routing(1/1); complexity-model-routing(1/1) | 2/2 |  |
| strong | cost-accounting | cost-accounting(1/1); cost-accounting-honesty(1/1) | 2/2 |  |
| strong | crash-exit-diagnostics | crash-diagnostics-gap(1/1); crash-exit-diagnostics(1/1) | 2/2 |  |
| strong | dual-loop-migration | dual-loop-migration(1/1); dual-tui-migration(1/1) | 2/2 |  |
| strong | sandbox-fail-closed-optin | fail-closed-optin-jail(1/1); sandbox-fail-closed-optin(1/1) | 2/1 |  |
| strong | fail-open-scanner-default | fail-open-scanner-default(1/1); opt-in-fail-open-scanner(1/1) | 2/2 |  |
| strong | session-writer-lease | session-turn-lease(1/1); session-writer-lease(1/1) | 2/2 |  |
| strong | verification-ceiling | verification-8-ceiling(1/1); verification-ceiling(1/1) | 2/2 |  |
| medium | windows-sandbox-gap | windows-appcontainer-sandbox(1/1); windows-sandbox-gap(2/2) | 3/3 | appcontainer-support and windows-gap are the same axis (windows sandbox coverage) |
| medium | *compaction-hooks* | compaction-collapse-hooks(1/1); compaction-hooks-absent(1/1) | 2/2 | presence/absence pair; kind carries polarity |
| medium | sandbox-absent | no-kernel-sandbox(1/1); no-os-sandbox(1/1); no-sandbox-by-design(1/1); sandbox-absent(1/1) | 4/4 | all = no OS-level sandbox; stance-vs-omission nuance in one_why |
| weak | oauth-client-impersonation | mcp-oauth-client(1/1); oauth-client-impersonation(2/2) | 3/3 | mcp-oauth-client may be a neutral capability note rather than the impersonation threat |
| weak | *injection-defense* | injection-defense-absent(1/1); structural-injection-defense(1/1) | 2/2 | absence-vs-presence pair; do NOT merge into injection-screening (different mechanism) |
| weak | untrusted-input-framing | untrusted-input-framing(1/1); untrusted-result-framing(1/1) | 2/2 | input vs tool-result framing; merge only if synthesis treats them as one framing concept |
| weak | usage-measured-compact-trigger | usage-measured-compact-trigger(19/17); projected-context-trigger(1/1); threshold-context-trigger(1/1); projected-request-compaction-trigger(1/1) | 22/20 | trigger-measurement family; integrators already merged deepseek-reasonix dual-boundary-compaction-trigger into usage-measured-compact-trigger -- follow that precedent |
| weak | compaction-tiering | compaction-tiering(42/41); two-tier-v1-compaction(1/1) | 43/42 | two-tier-v1 is a tiering instance; workground2 lane alias tiered-compaction-ladder already resolved into compaction-tiering |
| weak | workflow-resume-journal | workflow-resume-journal(19/17); same-process-only-workflow-resume(1/1) | 20/17 | the latter is the anti-pattern half of the same resumability concept |

## Explicitly NOT merged (adjacent but distinct)

- **god-file-loop vs god-file-host-wiring** -- different loci (agent loop vs host wiring); god-file-loop is 24/24, god-file-host-wiring is 12/12; convergence must count them separately
- **layered-loop-separation vs layered-loop-breakers** -- separation is architecture, breakers are runtime guards
- **llm-compaction-judge vs llm-permission-judge** -- same pattern, different targets; keep a shared alias if synthesis wants a meta-cluster
- **permission-policy vs permission-coalescing / permission-widening-ux / subagent-permission-inheritance / server-mode-permission-deferral / dual-permission-engine-mid-migration / fail-closed-permission-skeleton / llm-permission-judge** -- true variants are distinct designs; permission-policy (80/58) stays the single big bucket; no further permission merges proposed
- **checkpoint-revert vs checkpoint-failopen-degradation** -- the latter is a failure-mode of the former; keep kind-level nuance, do not fold
- **compaction-tiering vs cache-monotonic-compaction vs overflow-degradation-ladder** -- three distinct seeded concepts; do not collapse the compaction family into one id

## Integrator-resolved structured alias merges (already canonical, harvest as aliases)

| record | lane-coined concept | merged_into (canonical) |
|---|---|---|
| workground2-s1 | `sandbox-delegation` | `default-on-os-jail` |
| workground2-s2 | `sandbox-delegation` | `windows-sandbox-gap` |
| workground2-e1 | `tiered-compaction-ladder` | `compaction-tiering` |
| workground2-e3 | `prompt-cache-observability` | `cache-miss-attribution` |
| workground2-e6 | `loop-detection-breaker` | `loop-detection` |
| workground2-e14 | `actionable-error-dx` | `actionable-errors` |
| deepseek-reasonix-s4 | `ast-not-regex-judgement` | `grammar-parsed-bash-policy` |
| deepseek-reasonix-s6 | `headless-fail-closed` | `fail-closed-default-decision` |
| deepseek-reasonix-s8 | `advisory-injection-screen` | `injection-screening` |
| deepseek-reasonix-s9 | `nil-approver-ask-allows` | `autonomous-confirm-failopen` |
| deepseek-reasonix-s11 | `enforcement-tests-run-in-ci` | `e2e-runs-against-real-sandbox` |
| deepseek-reasonix-s12 | `evals-in-ci-gated` | `evals-in-ci` |
| deepseek-reasonix-e1 | `dual-boundary-compaction-trigger` | `usage-measured-compact-trigger` |
| deepseek-reasonix-e11 | `errors-carry-the-fix` | `actionable-errors` |

(opencode-m1 additionally carries two `{id, lane, concept}` alias objects -- lane-record provenance, not new aliases.)

## Alias harvest

177 records carry alias-family fields (`alias`/`aliases`/`concept_alias`/`concept_aliases`/`lane_concept`); 157 distinct alias strings (excluding self-references and the 16 structured entries above). Notable lane-coined aliases that must NOT become convergence buckets at synthesis: `cache-aware-compaction`, `tiered-compaction-ladder`, `compaction-tier-stack`, `measured-context-accounting`, `token-ground-truth-feedback`, `passive-prompt-cache`, `compat-calendar-rot`, `startup-deadlock-watchdog`.

## Full concept table

concept | records | subjects (n) | subjects
|---|---|---|---|
| permission-policy | 84 | 60 | aeon, agentty, amazon-q-developer-cli, atomic-agent, auto-code-rover, bitfun, claude-engineer, claurst, cline, code, codewhale, codex, continue, coro-code, crab-code, crush, cursor-agent, darce-cli, deepagents, devon, dvalincode, ferrum, forge, g3, gptme, grok-cli, groq-code-cli, hermes-agent, ipsupport-code, jazz, keen-code, kilocode, kimi-cli, kimi-code, kode-cli, kolega-code, kolkrabbi, maki, mini-kode, minicode, mistral-vibe, mocode, nausicaa-harness, neovate-code, ob-1, octomind, open-codex, openhands, openharness, orca-agent, ouroboros, pi, qwen-code, roo-code, tau, trae-agent, vtcode, workground2, zap-coding-agent, zot |
| faux-provider-testing | 47 | 46 | SWE-agent, amazon-q-developer-cli, claurst, claw-code, claw-code-agent, cline, code, codewhale, codex, continue, coro-code, crab-code, deepagents, deepseek-reasonix, ferrum, forge, g3, gptme, grok-build, hax, hermes-agent, ipsupport-code, keen-code, kimi-cli, kimi-code, kode-cli, kolega-code, kolkrabbi, maki, memcode, mini-kode, mocode, nausicaa-harness, ob-1, open-codex, opencode, openhands, openharness, ouroboros, pi, qqcode, qwen-code, tau, trae-agent, tura, workground2 |
| compaction-tiering | 42 | 41 | 3code, claurst, claw-code, cline, code, codebuff, codewhale, codex, coro-code, crab-code, deepagents, forge, g3, gemini-cli, goose, gptme, grinta-coding-agent, grok-build, hermes-agent, ipsupport-code, jazz, kilocode, kode-cli, kolkrabbi, letta-code, memcode, minicode, mocode, molt, nausicaa-harness, neovate-code, ob-1, oh-my-pi, openharness, orca-agent, pi, qwen-code, tura, vtcode, waveloom, workground2 |
| loop-detection | 35 | 34 | 3code, agentty, atomic-agent, bitfun, cline, codebuff, codewhale, crab-code, crush, darce-cli, dvalincode, ferrum, forge, gemini-cli, grinta-coding-agent, jazz, kimi-cli, kimi-code, kolega-code, kolkrabbi, maki, memcode, mistral-vibe, molt, ob-1, octomind, opencode, qwen-code, ra-aid, roo-code, san, waveloom, workground2, zap-coding-agent |
| sandbox-delegation | 34 | 32 | agentty, auto-code-rover, bitfun, cline, codebuff, codel, codemachine-cli, codex, continue, crab-code, crush, gemini-cli, grinta-coding-agent, grok-build, grok-cli, hax, hermes-agent, ipsupport-code, jazz, jcode, kimi-code, mistral-vibe, nanocoder, ob-1, openharness, ouroboros, pi, prime-agent, qwen-code, trae-agent, vtcode, waveloom |
| prompt-cache-marking | 31 | 29 | SWE-agent, agentless, agentty, claurst, claw-code, code, codewhale, deepagents, dvalincode, forge-norvialabs, g3, goose, jazz, keen-code, kilocode, kode-cli, maki, memcode, mistral-vibe, neovate-code, ob-1, openharness, plandex, qqcode, smelt, tau, tura, vtcode, zot |
| checkpoint-revert | 28 | 27 | 3code, aider, amazon-q-developer-cli, bitfun, cline, code, codewhale, continue, crab-code, deepagents, deepseek-reasonix, devon, gemini-cli, grinta-coding-agent, grok-build, ipsupport-code, jcode, kilocode, kimi-cli, minicode, nanocoder, ob-1, opencode, plandex, qwen-code, vtcode, workground2 |
| evals-in-ci | 28 | 27 | atomic-agent, cline, code, codex, continue, deepagents, deepseek-reasonix, forge, gemini-cli, gptme, grinta-coding-agent, hermes-agent, jazz, jcode, kilocode, kimi-code, kolkrabbi, letta-code, mocode, octomind, openhands, orca-agent, ouroboros, pi, prime-agent, qwen-code, workground2 |
| hook-trust-scoping | 26 | 25 | agentty, amazon-q-developer-cli, bitfun, claurst, cline, code, codebuff, crush, ferrum, gptme, grok-build, grok-cli, jcode, kimi-cli, kode-cli, memcode, mini-kode, mistral-vibe, mocode, openharness, pi, tau, waveloom, zap-coding-agent, zot |
| regex-denylist | 25 | 22 | SWE-agent, binharic-cli, claw-code-agent, cursor-agent, grinta-coding-agent, hermes-agent, keen-code, kilocode, kolkrabbi, mini-kode, minicode, mocode, nanocoder, neovate-code, oh-my-pi, orca-agent, qqcode, roo-code, tura, workground2, zap-coding-agent, zot |
| security-posture-docs | 25 | 23 | bitfun, continue, deepagents, ferrum, gemini-cli, grok-build, hermes-agent, kilocode, letta-code, maki, minicode, mistral-vibe, opencode, openhands, openharness, orca-agent, pi, qwen-code, roo-code, smelt, vtcode, workground2, zap-coding-agent |
| god-file-loop | 24 | 24 | aeon, aider, amazon-q-developer-cli, binharic-cli, bitfun, claw-code, claw-code-agent, code, codewhale, crush, cursor-agent, devon, ferrum, g3, goose, grok-cli, mistral-vibe, molt, prime-agent, qwen-code, roo-code, vtcode, workground2, zeroclaw |
| cache-monotonic-compaction | 21 | 19 | SWE-agent, agentty, atomic-agent, claw-code-agent, codebuff, codex, gptme, grinta-coding-agent, hax, jazz, kilocode, memcode, mistral-vibe, mocode, molt, octomind, ouroboros, san, waveloom |
| usage-measured-compact-trigger | 19 | 17 | codebuff, deepseek-reasonix, dvalincode, ferrum, gemini-cli, goose, ipsupport-code, jazz, keen-code, kilocode, letta-code, mini-kode, mistral-vibe, mocode, qqcode, zap-coding-agent, zot |
| workflow-resume-journal | 19 | 17 | SWE-agent, agentless, cline, codewhale, dvalincode, grok-build, jazz, kolega-code, minicode, mocode, molt, orca-agent, ouroboros, prime-agent, qwen-code, tura, zot |
| test-suite-without-ci | 18 | 15 | claw-code-agent, codebuff, darce-cli, devon, ferrum, g3, grok-build, grok-cli, kimi-code, kolega-code, mini-kode, mistral-vibe, plandex, roo-code, zap-coding-agent |
| phantom-safety-control | 17 | 13 | binharic-cli, claurst, claw-code, code, codebuff, continue, forge, open-codex, openlumara, ra-aid, trae-agent, tura, vtcode |
| grammar-parsed-bash-policy | 16 | 14 | continue, deepseek-reasonix, ferrum, grok-build, kimi-code, maki, memcode, mistral-vibe, opencode, openharness, ouroboros, smelt, vtcode, waveloom |
| session-rollout-journal | 15 | 15 | auto-code-rover, code, codex, forge-norvialabs, g3, gemini-cli, hax, kimi-cli, mistral-vibe, oh-my-pi, open-codex, qwen-code, san, vtcode, zeroclaw |
| god-file-host-wiring | 13 | 13 | aider, code, continue, gptme, ipsupport-code, letta-code, nausicaa-harness, oh-my-pi, orca-agent, tau, workground2, zeroclaw, zot |
| architecture-contract-test | 12 | 11 | bitfun, deepseek-reasonix, kilocode, kode-cli, kolkrabbi, letta-code, minicode, openhands, orca-agent, san, zeroclaw |
| injection-screening | 12 | 12 | bitfun, deepseek-reasonix, gemini-cli, gptme, grok-build, mistral-vibe, openlumara, orca-agent, roo-code, san, vtcode, waveloom |
| lazy-skill-loading | 11 | 11 | 3code, forge, forge-norvialabs, kolega-code, maki, nausicaa-harness, neovate-code, qqcode, san, zap-coding-agent, zot |
| quality-gates-unwired | 11 | 10 | amazon-q-developer-cli, claw-code-agent, cursor-agent, gptme, grinta-coding-agent, groq-code-cli, openhands, openharness, qqcode, vtcode |
| workspace-path-containment | 11 | 10 | claii, darce-cli, grok-build, groq-code-cli, keen-code, kimi-cli, mini-kode, mocode, picocode, roo-code |
| crash-recovery-middleware | 10 | 10 | codewhale, continue, crush, gemini-cli, goose, grok-build, qwen-code, roo-code, workground2, zeroclaw |
| overflow-degradation-ladder | 10 | 10 | agentty, amazon-q-developer-cli, gptme, grinta-coding-agent, grok-cli, kolega-code, maki, nausicaa-harness, ra-aid, tau |
| scheduled-agent-runs | 10 | 10 | aeon, cline, goose, grok-build, hermes-agent, letta-code, openlumara, ouroboros, qwen-code, vtcode |
| actionable-errors | 8 | 8 | code, deepseek-reasonix, dexto, gemini-cli, goose, mistral-vibe, qwen-code, workground2 |
| memory-files | 8 | 8 | atomic-agent, ipsupport-code, kimi-code, mimo-code, nanocoder, open-codex, prime-agent, ra-aid |
| network-egress-approval | 8 | 8 | 3code, codex, forge-norvialabs, grok-build, mini-kode, orca-agent, vtcode, workground2 |
| validated-finish-gate | 8 | 7 | atomic-agent, forge-norvialabs, grinta-coding-agent, minicode, molt, octomind, ouroboros |
| classifier-approval-ladder | 7 | 6 | deepagents, jazz, kode-cli, kolega-code, octomind, vtcode |
| context-overflow-failopen | 7 | 7 | auto-code-rover, codel, cursor-agent, devon, groq-code-cli, open-codex, trae-agent |
| eval-harness-outside-ci | 7 | 7 | agentless, amazon-q-developer-cli, jazz, ob-1, ra-aid, vtcode, waveloom |
| auto-continuation-goals | 6 | 6 | claurst, codex-infinity, deepagents, ipsupport-code, mimo-code, smelt |
| autonomous-confirm-failopen | 6 | 5 | deepseek-reasonix, gptme, kode-cli, workground2, zot |
| prompt-cache-warming | 6 | 4 | gemini-cli, octomind, pi, qwen-code |
| acp-internal-contract | 5 | 5 | grok-build, kimi-cli, nanocoder, prime-agent, qqcode |
| confine-or-refuse-startup | 5 | 5 | agentty, forge-norvialabs, grinta-coding-agent, grok-build, orca-agent |
| entry-path-loop-duplication | 5 | 5 | continue, gemini-cli, gptme, letta-code, vtcode |
| fail-closed-default-decision | 5 | 5 | bitfun, deepseek-reasonix, gemini-cli, grok-build, mistral-vibe |
| interop-matrix | 5 | 5 | codewhale, gemini-cli, mistral-vibe, qwen-code, vtcode |
| lazy-tool-catalog | 5 | 5 | forge-norvialabs, maki, openharness, openlumara, zap-coding-agent |
| ported-proprietary-source | 5 | 4 | claurst, claw-code, claw-code-agent, crab-code |
| readme-capability-drift | 5 | 5 | claii, cursor-agent, darce-cli, devon, goose |
| rival-subscription-transport | 5 | 4 | atomic-agent, grinta-coding-agent, molt, zot |
| steer-during-run | 5 | 5 | codebuff, dexto, goose, gptme, kilocode |
| subagent-process-delegation | 5 | 5 | amazon-q-developer-cli, code, goose, hax, kimi-cli |
| turn-budget-accounting | 5 | 5 | code, codewhale, goose, neovate-code, openlumara |
| unrenamed-manifest-identity | 5 | 5 | code, forge, mimo-code, open-interpreter, orca-agent |
| code-mode-runtime | 4 | 4 | codex, kolega-code, maki, opencode |
| event-sourced-agent-state | 4 | 4 | devon, grinta-coding-agent, kimi-code, nausicaa-harness |
| guardian-reviewer-pool | 4 | 4 | codex, keen-code, san, workground2 |
| headless-rpc-protocol | 4 | 4 | codewhale, goose, maki, pi |
| layered-loop-separation | 4 | 4 | deepseek-reasonix, gemini-cli, oh-my-pi, orca-agent |
| manifest-only-license | 4 | 4 | coro-code, darce-cli, g3, zap-coding-agent |
| no-durable-queue | 4 | 4 | code, gemini-cli, letta-code, mistral-vibe |
| opt-in-hard-enforcement | 4 | 4 | codewhale, gemini-cli, letta-code, octomind |
| parallel-agent-stack-migration | 4 | 4 | amazon-q-developer-cli, goose, mistral-vibe, zeroclaw |
| side-effect-verified-guardrail-tests | 4 | 4 | bitfun, forge-norvialabs, grok-build, tura |
| ui-coupled-loop | 4 | 4 | codewhale, continue, nanocoder, vtcode |
| autopilot-default-failopen | 3 | 3 | bitfun, letta-code, ob-1 |
| deterministic-fuzz-replay | 3 | 3 | agentty, minicode, smelt |
| e2e-runs-against-real-sandbox | 3 | 3 | codewhale, deepseek-reasonix, gemini-cli |
| interactive-pty-e2e | 3 | 3 | 3code, ob-1, octomind |
| lease-fenced-run-execution | 3 | 3 | deepseek-reasonix, mistral-vibe, nausicaa-harness |
| no-fuzzing-advisory-evals | 3 | 3 | codewhale, gemini-cli, hax |
| prompt-cache-discipline | 3 | 3 | aider, bitfun, qwen-code |
| receipt-based-token-accounting | 3 | 3 | 3code, codebuff, tau |
| reviewer-retry-loop | 3 | 2 | SWE-agent, auto-code-rover |
| session-tree-branching | 3 | 3 | codewhale, pi, zeroclaw |
| subscription-continuity | 3 | 3 | 3code, goose, kolkrabbi |
| test-fixture-loc-inflation | 3 | 3 | auto-code-rover, deepagents, kolega-code |
| tool-output-spillover-files | 3 | 3 | crab-code, forge, gemini-cli |
| unauthenticated-control-plane | 3 | 3 | codel, devon, openlumara |
| zero-verification-snapshot | 3 | 3 | codemachine-cli, free-code, openlumara |
| aci-tool-design | 2 | 2 | SWE-agent, auto-code-rover |
| agent-directive-file | 2 | 2 | codemachine-cli, groq-code-cli |
| approval-only-enforcement | 2 | 2 | plandex, san |
| cache-miss-attribution | 2 | 2 | deepseek-reasonix, workground2 |
| compaction-continuity-gate | 2 | 2 | forge-norvialabs, grinta-coding-agent |
| coverage-regression-gate | 2 | 2 | nanocoder, prime-agent |
| declared-parity-lineage | 2 | 2 | kimi-code, ob-1 |
| default-on-os-jail | 2 | 2 | deepseek-reasonix, workground2 |
| default-on-telemetry | 2 | 2 | devon, keen-code |
| dns-pinned-ssrf-guard | 2 | 2 | maki, openlumara |
| documented-omissions | 2 | 2 | codewhale, hax |
| dual-copy-parity-test | 2 | 2 | 3code, codebuff |
| dual-session-abstraction | 2 | 2 | bitfun, gemini-cli |
| embedded-internal-markers | 2 | 2 | free-code, kode-cli |
| embedded-wal-datastore | 2 | 2 | kimi-code, smelt |
| extension-owned-policy | 2 | 2 | pi, smelt |
| fork-cache-alignment | 2 | 2 | grok-build, waveloom |
| harness-adapter-contract | 2 | 2 | aeon, codemachine-cli |
| headless-contract | 2 | 2 | grok-build, workground2 |
| heuristic-task-completion | 2 | 2 | cursor-agent, trae-agent |
| interop-breadth | 2 | 2 | continue, hermes-agent |
| interop-ceilings | 2 | 2 | bitfun, orca-agent |
| landlock-self-sandbox | 2 | 2 | 3code, octomind |
| master-context-recovery | 2 | 2 | ferrum, gptme |
| mcp-prompt-skills | 2 | 2 | crab-code, keen-code |
| message-bus-host-split | 2 | 2 | continue, neovate-code |
| mid-turn-user-inbox | 2 | 2 | g3, grok-build |
| model-provenance-attestation | 2 | 2 | aeon, gptme |
| module-size-ratchet | 2 | 2 | kolkrabbi, ouroboros |
| oauth-client-impersonation | 2 | 2 | claurst, qqcode |
| onboarding-error-dx | 2 | 2 | bitfun, orca-agent |
| parallel-engine-migration-tax | 2 | 2 | kimi-code, opencode |
| phantom-artifact-save | 2 | 2 | coro-code, mini-kode |
| project-settings-self-grant | 2 | 2 | claurst, mini-kode |
| provider-plane-fork | 2 | 2 | open-codex, open-interpreter |
| read-before-edit-hash-guard | 2 | 2 | darce-cli, mini-kode |
| reasoning-chain-carryover | 2 | 2 | forge, grok-cli |
| recursive-subagent-spawn | 2 | 2 | openhands, prime-agent |
| reproducible-release-builds | 2 | 1 | 3code |
| retry-blind-backoff | 2 | 2 | auto-code-rover, trae-agent |
| self-modification-tiering | 2 | 2 | ouroboros, san |
| server-mode-permission-deferral | 2 | 2 | openharness, ra-aid |
| shadow-mode-guardrail | 2 | 2 | gptme, ipsupport-code |
| side-channel-readonly-queries | 2 | 2 | keen-code, kimi-code |
| sni-pinned-egress-proxy | 2 | 1 | kilocode |
| speculative-compaction | 2 | 2 | bitfun, oh-my-pi |
| subagent-permission-inheritance | 2 | 2 | keen-code, opencode |
| tamper-evident-audit-chain | 2 | 2 | dvalincode, molt |
| tests-prove-properties | 2 | 2 | gemini-cli, vtcode |
| tool-bundle-install | 2 | 2 | SWE-agent, trae-agent |
| tool-disclosure-tiers | 2 | 2 | jazz, mocode |
| trim-only-context-window | 2 | 2 | binharic-cli, darce-cli |
| turn-provenance-tagging | 2 | 2 | crush, prime-agent |
| two-pass-background-prefire | 2 | 2 | grok-build, vtcode |
| windows-sandbox-gap | 2 | 2 | deepseek-reasonix, workground2 |
| yolo-no-floor | 2 | 2 | gemini-cli, grok-build |
| a2a-peer-door | 1 | 1 | jazz |
| a2a-plus-foreign-harness | 1 | 1 | dexto |
| ablation-pair-eval | 1 | 1 | memcode |
| acp-as-universal-surface | 1 | 1 | grok-build |
| acp-bidirectional | 1 | 1 | bitfun |
| acp-bidirectional-host | 1 | 1 | goose |
| acp-first-class | 1 | 1 | opencode |
| acp-plane | 1 | 1 | orca-agent |
| acp-server-surface | 1 | 1 | workground2 |
| acp-trust-default | 1 | 1 | vtcode |
| action-pin-consistency-test | 1 | 1 | dvalincode |
| actuals-based-context-projection | 1 | 1 | dexto |
| adr-doc-skew | 1 | 1 | mistral-vibe |
| adversarial-fixture-tests | 1 | 1 | goose |
| advisory-observer-lane | 1 | 1 | nausicaa-harness |
| agent-driven-ui-control | 1 | 1 | openhands |
| agent-flow-diagrams | 1 | 1 | kimi-cli |
| agentless-pipeline | 1 | 1 | agentless |
| agents-docs-mesh | 1 | 1 | bitfun |
| airgap-ssh-relay | 1 | 1 | agentty |
| ambient-content-injection | 1 | 1 | aider |
| anti-vacuous-test-gate | 1 | 1 | molt |
| api-implicit-yes-always | 1 | 1 | aider |
| approval-bound-in-loop | 1 | 1 | opencode |
| approval-default-off | 1 | 1 | goose |
| approval-free-kernel-fence | 1 | 1 | 3code |
| approval-grammar-tested | 1 | 1 | opencode |
| approval-key-scope-bypass | 1 | 1 | dexto |
| approval-lifecycle-spec | 1 | 1 | opencode |
| approval-only-shell | 1 | 1 | aider |
| architect-editor-pipeline | 1 | 1 | aider |
| ask-fails-closed | 1 | 1 | kilocode |
| async-remote-subagents | 1 | 1 | deepagents |
| atomic-session-store | 1 | 1 | bitfun |
| authority-revalidating-tool-cache | 1 | 1 | codewhale |
| auto-debug-repair-loop | 1 | 1 | plandex |
| autofix-edit-search | 1 | 1 | binharic-cli |
| backend-absent-failopen | 1 | 1 | 3code |
| backend-registry-multi-origin | 1 | 1 | openhands |
| background-recovery-thin | 1 | 1 | vtcode |
| background-run-stream-resume | 1 | 1 | letta-code |
| background-task-adopt | 1 | 1 | hax |
| bash-ast-rule-matching | 1 | 1 | san |
| behavior-equivalence-gate | 1 | 1 | tura |
| binary-hijack-var-block | 1 | 1 | waveloom |
| boomerang-delegation-stack | 1 | 1 | roo-code |
| branch-indifferent-security-tests | 1 | 1 | goose |
| brand-adjacent-naming | 1 | 1 | cursor-agent |
| budgeted-delegation | 1 | 1 | hermes-agent |
| budgeted-fold-recall | 1 | 1 | deepseek-reasonix |
| bundled-diagnostics-command | 1 | 1 | workground2 |
| busl-change-license | 1 | 1 | kolega-code |
| cache-discipline-as-observable-property | 1 | 1 | jcode |
| cache-evidence-release-gate | 1 | 1 | nausicaa-harness |
| cache-impact-ci-gate | 1 | 1 | workground2 |
| cache-safe-fork | 1 | 1 | qwen-code |
| cache-safe-result-dedup | 1 | 1 | smelt |
| cache-strategy-siloed-to-bedrock | 1 | 1 | roo-code |
| cache-warm-resume-replay | 1 | 1 | 3code |
| caller-allowlist-default-deny | 1 | 1 | hermes-agent |
| cancellation-race-hygiene | 1 | 1 | open-codex |
| cgroup-process-containment | 1 | 1 | ferrum |
| checked-in-run-artifacts | 1 | 1 | auto-code-rover |
| checkpoint-failopen-degradation | 1 | 1 | roo-code |
| checkpoint-log-projection | 1 | 1 | mistral-vibe |
| ci-culture | 1 | 1 | opencode |
| ci-excluded-test-files | 1 | 1 | atomic-agent |
| ci-perf-budget | 1 | 1 | forge |
| ci-security-gates | 1 | 1 | qwen-code |
| circuit-breaker-scheduler | 1 | 1 | aeon |
| classifier-fail-closed | 1 | 1 | qwen-code |
| classifier-model-routing | 1 | 1 | zap-coding-agent |
| cli-internal-boundary | 1 | 1 | tura |
| code-as-actions-loop | 1 | 1 | ra-aid |
| command-arity-attention-patterns | 1 | 1 | opencode |
| command-enum-bloat | 1 | 1 | grok-build |
| compaction-authority-ordering | 1 | 1 | ferrum |
| compaction-collapse-hooks | 1 | 1 | maki |
| compaction-engine-split | 1 | 1 | continue |
| compaction-god-pair | 1 | 1 | hermes-agent |
| compaction-hooks-absent | 1 | 1 | codewhale |
| compaction-pricing | 1 | 1 | deepseek-reasonix |
| compaction-trigger-completion-budget | 1 | 1 | coro-code |
| compaction-trigger-fixed-window | 1 | 1 | mini-kode |
| compaction-trigger-ground-truth | 1 | 1 | hermes-agent |
| compaction-trigger-measurement | 1 | 1 | workground2 |
| compile-enforced-protocol-parity | 1 | 1 | codewhale |
| compile-time-exhaustive-policy-proof | 1 | 1 | agentty |
| complexity-model-routing | 1 | 1 | waveloom |
| compression-evidence-lanes | 1 | 1 | octomind |
| computed-blast-radius-deny-tier | 1 | 1 | jcode |
| config-doc-drift | 1 | 1 | deepseek-reasonix |
| config-monotonic-tightening | 1 | 1 | kilocode |
| config-scope-monotone-narrowing | 1 | 1 | kilocode |
| confinement-intersection | 1 | 1 | kilocode |
| consequence-class-grants | 1 | 1 | memcode |
| containment-derived-grants | 1 | 1 | codebuff |
| content-addressed-summary-cache | 1 | 1 | orca-agent |
| context-dehydration-fragments | 1 | 1 | g3 |
| context-economy-guardrails | 1 | 1 | orca-agent |
| context-epoch-cache-baseline | 1 | 1 | opencode |
| context-upgrade-ladder | 1 | 1 | crab-code |
| context-utils-grab-bag | 1 | 1 | dexto |
| core-loop-naming-trap | 1 | 1 | mistral-vibe |
| core-loop-uncovered | 1 | 1 | trae-agent |
| cost-accounting | 1 | 1 | continue |
| cost-accounting-honesty | 1 | 1 | grok-build |
| cost-budget-autosubmit | 1 | 1 | SWE-agent |
| counterfactual-artifact-promotion | 1 | 1 | octomind |
| crash-classified-stop-reasons | 1 | 1 | jcode |
| crash-diagnostics-gap | 1 | 1 | mistral-vibe |
| crash-durability-gaps | 1 | 1 | opencode |
| crash-durable-swarm-coordination | 1 | 1 | jcode |
| crash-exit-diagnostics | 1 | 1 | oh-my-pi |
| crash-group-recovery-sessions | 1 | 1 | jcode |
| crash-posture | 1 | 1 | aider |
| crash-recovery-posture | 1 | 1 | hermes-agent |
| crash-safe-json-persistence | 1 | 1 | roo-code |
| crate-extraction-path-alias | 1 | 1 | codewhale |
| cross-harness-session-resume | 1 | 1 | jcode |
| cross-surface-rewind-invariant | 1 | 1 | hermes-agent |
| curl-replay-export | 1 | 1 | groq-code-cli |
| cursor-addressable-replay | 1 | 1 | grok-build |
| daemon-separated-agent-loop | 1 | 1 | jcode |
| dangerous-default-widened-surface | 1 | 1 | oh-my-pi |
| dangling-turn-repair | 1 | 1 | kilocode |
| dead-command-validation-layer | 1 | 1 | dexto |
| dead-concurrency-classification | 1 | 1 | darce-cli |
| debug-dump-eval-asserts | 1 | 1 | forge |
| debug-socket-surface | 1 | 1 | jcode |
| declarative-ci-recipes | 1 | 1 | picocode |
| default-allow-policy | 1 | 1 | opencode |
| default-write-open-read-open | 1 | 1 | kilocode |
| delegation-prefix-identity | 1 | 1 | mocode |
| denial-receipt-escalation | 1 | 1 | orca-agent |
| destructive-discard-guard | 1 | 1 | prime-agent |
| detect-only-invariant | 1 | 1 | continue |
| diagnostic-doctor-command | 1 | 1 | orca-agent |
| differential-parser-oracle | 1 | 1 | kimi-code |
| disclosed-private-mirror | 1 | 1 | codebuff |
| dispatcher-side-sandbox | 1 | 1 | aeon |
| distribution-fork-branding | 1 | 1 | open-interpreter |
| doc-tree-depth | 1 | 1 | roo-code |
| docs-and-error-quality | 1 | 1 | kilocode |
| docs-as-governed-corpus | 1 | 1 | deepseek-reasonix |
| docs-as-shipped-artifact | 1 | 1 | grok-build |
| docs-breadth-with-rot-and-undocumented-acp | 1 | 1 | jcode |
| docs-code-divergence-headless-permissions | 1 | 1 | continue |
| docs-coverage-and-drift | 1 | 1 | workground2 |
| docs-depth-vs-dual-tree | 1 | 1 | opencode |
| docs-drift-gate | 1 | 1 | qwen-code |
| docs-drift-gates | 1 | 1 | orca-agent |
| docs-dx-mass | 1 | 1 | aider |
| docs-dx-tooling | 1 | 1 | vtcode |
| docs-match-code | 1 | 1 | codewhale |
| docs-scope-gaps | 1 | 1 | grok-build |
| docs-tree-quality | 1 | 1 | continue |
| dual-durability-ledgers | 1 | 1 | orca-agent |
| dual-loop-migration | 1 | 1 | opencode |
| dual-permission-engine-mid-migration | 1 | 1 | letta-code |
| dual-session-engines | 1 | 1 | kilocode |
| dual-tool-engine | 1 | 1 | qwen-code |
| dual-tui-migration | 1 | 1 | qwen-code |
| duplicate-policy-path | 1 | 1 | gemini-cli |
| durability-loss-contract | 1 | 1 | grok-build |
| durable-approval-records | 1 | 1 | dexto |
| durable-bounded-board | 1 | 1 | kilocode |
| durable-inbox-and-restart-honest-jobs | 1 | 1 | deepseek-reasonix |
| durable-queue-dispatch | 1 | 1 | bitfun |
| durable-run-orchestration | 1 | 1 | workground2 |
| e2e-boundary-harness | 1 | 1 | hermes-agent |
| economy-machinery-density | 1 | 1 | oh-my-pi |
| edit-format-strategy-factory | 1 | 1 | aider |
| edit-ledger-injection | 1 | 1 | zap-coding-agent |
| effort-routing-dial | 1 | 1 | kolkrabbi |
| egress-authority-separation | 1 | 1 | deepseek-reasonix |
| egress-log-only | 1 | 1 | goose |
| embedding-capability-routing | 1 | 1 | octomind |
| enforcement-chokepoint | 1 | 1 | opencode |
| enforcement-untested-in-ci | 1 | 1 | orca-agent |
| ephemeral-checkpoint-rewind | 1 | 1 | mistral-vibe |
| ephemeral-orchestration-state | 1 | 1 | continue |
| ephemeral-task-registry | 1 | 1 | dexto |
| equality-guarded-background-summarizer | 1 | 1 | aider |
| escalation-human-only | 1 | 1 | kilocode |
| estimate-calibration | 1 | 1 | agentty |
| eval-approval-bypass | 1 | 1 | ra-aid |
| eval-on-model-output | 1 | 1 | agentless |
| evals-release-gated-no-fuzz | 1 | 1 | kilocode |
| event-driven-daemon | 1 | 1 | nanocoder |
| event-sourced-session-core | 1 | 1 | opencode |
| execution-budget-engine | 1 | 1 | orca-agent |
| expired-compat-shim-layer | 1 | 1 | hermes-agent |
| explicit-yes-fail-safe | 1 | 1 | aider |
| exposure-aware-tool-output-attenuation | 1 | 1 | jcode |
| external-coding-agents-as-tools | 1 | 1 | letta-code |
| external-command-rewrite | 1 | 1 | maki |
| external-docs-strong-failure-copy | 1 | 1 | letta-code |
| fail-closed-folder-trust | 1 | 1 | orca-agent |
| fail-closed-optin-jail | 1 | 1 | kilocode |
| fail-closed-permission-skeleton | 1 | 1 | goose |
| fail-open-scanner-default | 1 | 1 | hermes-agent |
| fail-safe-crash-semantics | 1 | 1 | oh-my-pi |
| failure-copy-engineering | 1 | 1 | hermes-agent |
| fanout-scope-grant | 1 | 1 | atomic-agent |
| fatal-crash-mirror | 1 | 1 | deepseek-reasonix |
| fatal-exit-semantics | 1 | 1 | letta-code |
| fault-localization-agent-tool | 1 | 1 | auto-code-rover |
| filesystem-boundary-half-strength | 1 | 1 | dexto |
| fleet-graph-restart-reissue | 1 | 1 | deepseek-reasonix |
| folded-file-context-in-summary | 1 | 1 | roo-code |
| folder-trust-off-by-default | 1 | 1 | gemini-cli |
| foreign-session-import | 1 | 1 | goose |
| foreign-session-ingest | 1 | 1 | grok-build |
| foreign-transcript-import | 1 | 1 | kilocode |
| fork-annotation-governance | 1 | 1 | kilocode |
| fork-overlay-loop | 1 | 1 | kilocode |
| fork-with-rollback | 1 | 1 | dexto |
| four-protocol-surfaces | 1 | 1 | deepseek-reasonix |
| free-tier-model-router | 1 | 1 | ob-1 |
| frozen-fact-turn-recovery | 1 | 1 | bitfun |
| frozen-kernel-seam | 1 | 1 | minicode |
| full-context-singlepass | 1 | 1 | open-codex |
| generalized-task-budget | 1 | 1 | deepseek-reasonix |
| generated-code-loc-census | 1 | 1 | amazon-q-developer-cli |
| generation-fenced-rehydration | 1 | 1 | zeroclaw |
| glob-command-grants | 1 | 1 | forge |
| global-singleton-session-state | 1 | 1 | code |
| god-config-object | 1 | 1 | qwen-code |
| god-object-config | 1 | 1 | gemini-cli |
| god-object-controller | 1 | 1 | workground2 |
| god-package-concentration | 1 | 1 | deepseek-reasonix |
| goroutine-leak-test-gate | 1 | 1 | workground2 |
| grammar-constrained-tool-calls | 1 | 1 | atomic-agent |
| grounded-guard-evolution | 1 | 1 | octomind |
| guardian-llm-approval | 1 | 1 | hermes-agent |
| guardrail-stripped-redistribution | 1 | 1 | free-code |
| hard-gates-and-rollback | 1 | 1 | aider |
| harness-addressable-loop | 1 | 1 | dexto |
| harness-constant-benchmark | 1 | 1 | kolkrabbi |
| harness-self-optimization | 1 | 1 | openharness |
| headless-and-sdk-contract | 1 | 1 | opencode |
| headless-cli-contract | 1 | 1 | bitfun |
| hermetic-multi-os-ci | 1 | 1 | codewhale |
| history-change-retry-guard | 1 | 1 | dexto |
| honest-absent-enforcement | 1 | 1 | hax |
| honest-durability-checklist | 1 | 1 | opencode |
| hook-timeout-failopen | 1 | 1 | maki |
| hooks-blind-to-default-mode | 1 | 1 | letta-code |
| host-parity-verification | 1 | 1 | qwen-code |
| host-resident-turn-policy | 1 | 1 | qwen-code |
| hot-reload-session-continuity | 1 | 1 | jcode |
| http-front-auth | 1 | 1 | workground2 |
| image-cost-blind-spot | 1 | 1 | vtcode |
| implicit-shared-loop-state | 1 | 1 | aider |
| in-flight-turn-resume | 1 | 1 | workground2 |
| in-memory-queue | 1 | 1 | kilocode |
| incident-grounded-retry-budgets | 1 | 1 | jcode |
| inherited-interop-surface | 1 | 1 | code |
| injection-defense-absent | 1 | 1 | dexto |
| internals-docs-matched-code | 1 | 1 | oh-my-pi |
| interop-mcp-server-and-ide-gap | 1 | 1 | oh-my-pi |
| interop-outbound-and-sdk-missing | 1 | 1 | vtcode |
| interop-regression-ci | 1 | 1 | dvalincode |
| interop-surface | 1 | 1 | kilocode |
| interop-surface-breadth | 1 | 1 | oh-my-pi |
| invalid-config-escalates | 1 | 1 | oh-my-pi |
| journal-derived-approval-state | 1 | 1 | goose |
| journal-driven-state-machine | 1 | 1 | goose |
| kanban-first-class-queue | 1 | 1 | hermes-agent |
| kernel-policy-string-tests-only | 1 | 1 | letta-code |
| last-message-state-inference | 1 | 1 | roo-code |
| layered-authorization-contract | 1 | 1 | codewhale |
| layered-budgets-with-actionable-denials | 1 | 1 | jcode |
| layered-loop-breakers | 1 | 1 | oh-my-pi |
| leader-follower-ipc-daemon | 1 | 1 | grok-build |
| leaderboard-fallback-model | 1 | 1 | ra-aid |
| leak-detection-production-corpus | 1 | 1 | oh-my-pi |
| leaked-proprietary-snapshot | 1 | 1 | free-code |
| learned-context-window | 1 | 1 | atomic-agent |
| least-privilege-secret-injection | 1 | 1 | aeon |
| license-gated-skill-install | 1 | 1 | openharness |
| llm-compaction-judge | 1 | 1 | code |
| llm-drift-gated-docs | 1 | 1 | gemini-cli |
| llm-generated-approval-grammar | 1 | 1 | opencode |
| llm-patch-selection | 1 | 1 | trae-agent |
| llm-permission-judge | 1 | 1 | goose |
| local-stack-supervisor | 1 | 1 | openhands |
| loss-bounded-compaction | 1 | 1 | workground2 |
| lua-scriptable-core | 1 | 1 | smelt |
| macro-command-graph | 1 | 1 | tura |
| manifest-gated-product-surface | 1 | 1 | openhands |
| markdown-transcript-resume | 1 | 1 | aider |
| mcp-client-depth | 1 | 1 | workground2 |
| mcp-client-only-autoenable | 1 | 1 | opencode |
| mcp-client-only-no-acp | 1 | 1 | letta-code |
| mcp-client-only-stdio-only | 1 | 1 | jcode |
| mcp-client-only-unverifiable-sdk | 1 | 1 | grok-build |
| mcp-client-three-transports | 1 | 1 | roo-code |
| mcp-oauth-client | 1 | 1 | amazon-q-developer-cli |
| mcp-secret-redaction | 1 | 1 | openhands |
| mcp-step-transition-gate | 1 | 1 | codemachine-cli |
| measured-cache-policy | 1 | 1 | hermes-agent |
| memory-export-server | 1 | 1 | memcode |
| memory-reflection-trees | 1 | 1 | ob-1 |
| mid-turn-steering | 1 | 1 | workground2 |
| middleware-exclusion-coverage-check | 1 | 1 | deepagents |
| migration-confusion-surface | 1 | 1 | codewhale |
| missing-license-open-source-claim | 1 | 1 | claw-code-agent |
| mit-restrictive-use-addendum | 1 | 1 | mimo-code |
| mixin-composed-state-bag | 1 | 1 | hermes-agent |
| model-exchange-tracing | 1 | 1 | bitfun |
| model-retrain-regression-gate | 1 | 1 | ipsupport-code |
| model-specific-loop-residue | 1 | 1 | oh-my-pi |
| model-visible-token-budget | 1 | 1 | openlumara |
| module-boundary-ratchet | 1 | 1 | qwen-code |
| monolithic-agent-module | 1 | 1 | ferrum |
| monolithic-turn-function | 1 | 1 | grok-build |
| multi-agent-team | 1 | 1 | cline |
| mutation-metric-not-gate | 1 | 1 | grinta-coding-agent |
| naive-json-session-store | 1 | 1 | continue |
| ndjson-headless-contract | 1 | 1 | roo-code |
| next-task-oracle | 1 | 1 | codel |
| no-cache-warming | 1 | 1 | kilocode |
| no-client-prompt-cache-discipline | 1 | 1 | letta-code |
| no-cost-token-budget | 1 | 1 | vtcode |
| no-durable-queue-or-journal | 1 | 1 | roo-code |
| no-image-budget | 1 | 1 | aider |
| no-interactive-approval-gate | 1 | 1 | jcode |
| no-kernel-sandbox | 1 | 1 | goose |
| no-llm-context-eviction | 1 | 1 | qwen-code |
| no-llm-economy-layers | 1 | 1 | grok-build |
| no-llm-emergency-and-413-payload-recovery | 1 | 1 | jcode |
| no-os-sandbox | 1 | 1 | oh-my-pi |
| no-prompt-injection-posture | 1 | 1 | jcode |
| no-protocol-interop | 1 | 1 | aider |
| no-published-sdk | 1 | 1 | hermes-agent |
| no-queue-no-journal | 1 | 1 | aider |
| no-sandbox-by-design | 1 | 1 | opencode |
| non-authoritative-denial-text | 1 | 1 | orca-agent |
| openai-compat-app-server-plus-sdk | 1 | 1 | letta-code |
| openapi-drift-gates | 1 | 1 | dexto |
| operation-journal-invariants | 1 | 1 | orca-agent |
| opt-in-fail-open-scanner | 1 | 1 | goose |
| opt-in-provider-cache | 1 | 1 | continue |
| opt-in-retention-preview | 1 | 1 | orca-agent |
| orchestration-budgets | 1 | 1 | grok-build |
| orchestration-docking-notes | 1 | 1 | bitfun |
| ordered-fit-ladder | 1 | 1 | grok-build |
| orphan-service | 1 | 1 | gemini-cli |
| orphan-tool-settlement-on-resume | 1 | 1 | opencode |
| orphaned-persistence | 1 | 1 | mini-kode |
| overflow-hard-stop | 1 | 1 | aider |
| overflow-summary-graft | 1 | 1 | plandex |
| panic-contained-turn-execution | 1 | 1 | orca-agent |
| panic-to-crashlog | 1 | 1 | code |
| parity-simulation-gap | 1 | 1 | letta-code |
| park-revive-subagent-crashrecovery | 1 | 1 | oh-my-pi |
| peer-egress-containment | 1 | 1 | jazz |
| per-provider-loop-duplication | 1 | 1 | keen-code |
| per-session-run-coordinator | 1 | 1 | opencode |
| permission-coalescing | 1 | 1 | crab-code |
| permission-widening-ux | 1 | 1 | gemini-cli |
| permissive-default-vs-docs | 1 | 1 | dexto |
| persist-partial-on-cancel | 1 | 1 | dexto |
| phantom-state-snapshot | 1 | 1 | code |
| phase-pipeline-loop | 1 | 1 | hermes-agent |
| pinned-attribution-hygiene | 1 | 1 | nausicaa-harness |
| plaintext-token-compare | 1 | 1 | codewhale |
| policy-file-self-lock | 1 | 1 | 3code |
| policy-hot-reload | 1 | 1 | 3code |
| policy-tier-hardening | 1 | 1 | gemini-cli |
| port-segregated-controller | 1 | 1 | workground2 |
| portable-agent-bundle | 1 | 1 | zot |
| pr-comment-injection-sanitization | 1 | 1 | codebuff |
| pre-mode-unbypassable-guards | 1 | 1 | letta-code |
| preregistered-placebo-eval | 1 | 1 | nausicaa-harness |
| presentation-god-region | 1 | 1 | jcode |
| preview-suffix-continuation | 1 | 1 | waveloom |
| priced-fold-economics | 1 | 1 | octomind |
| process-lifecycle-hygiene | 1 | 1 | kilocode |
| process-local-orchestration | 1 | 1 | opencode |
| projected-compaction-credit | 1 | 1 | continue |
| projected-context-trigger | 1 | 1 | orca-agent |
| projected-request-compaction-trigger | 1 | 1 | opencode |
| prompt-baked-policy | 1 | 1 | codel |
| prompt-cache-audit-only | 1 | 1 | orca-agent |
| prompt-cache-economy | 1 | 1 | oh-my-pi |
| prompt-cache-key-discipline | 1 | 1 | grok-build |
| prompt-cache-strategy-layer | 1 | 1 | roo-code |
| prompt-injection-zero-defense | 1 | 1 | opencode |
| prompt-preview-command | 1 | 1 | codewhale |
| property-asserting-test-corps-in-ci | 1 | 1 | oh-my-pi |
| prose-only-run-contract | 1 | 1 | dexto |
| protocol-level-cache-budgeting | 1 | 1 | opencode |
| provenance-anchored-compaction | 1 | 1 | octomind |
| provider-anchored-token-accounting | 1 | 1 | oh-my-pi |
| provider-retry-discipline | 1 | 1 | opencode |
| rate-limit-model-feedback | 1 | 1 | waveloom |
| re-derivable-fix-proof | 1 | 1 | dvalincode |
| reactive-only-compaction-single-knob | 1 | 1 | dexto |
| readonly-command-table | 1 | 1 | workground2 |
| real-violation-tests-in-ci | 1 | 1 | kilocode |
| rebranded-redistribution-provenance | 1 | 1 | open-interpreter |
| recoverability-tiered-permissions | 1 | 1 | san |
| redact-at-ingress | 1 | 1 | smelt |
| reflection-gate-justification | 1 | 1 | jcode |
| remote-control-surface | 1 | 1 | grok-cli |
| replica-tests | 1 | 1 | binharic-cli |
| repo-map-budget | 1 | 1 | aider |
| retired-tool-replay-surface | 1 | 1 | codewhale |
| reversible-summarization-cutoff | 1 | 1 | openlumara |
| rewind-safe-compaction | 1 | 1 | roo-code |
| risk-classifier-failopen | 1 | 1 | grinta-coding-agent |
| rival-agent-scan | 1 | 1 | gptme |
| rival-artifact-compat | 1 | 1 | orca-agent |
| rival-config-import | 1 | 1 | codewhale |
| rival-config-ingestion | 1 | 1 | deepseek-reasonix |
| rival-config-surface-import | 1 | 1 | claw-code-agent |
| rival-harness-adoption | 1 | 1 | workground2 |
| rival-harness-compat-imports | 1 | 1 | opencode |
| rival-harness-session-import | 1 | 1 | cline |
| rival-plugin-import | 1 | 1 | mistral-vibe |
| rival-state-import | 1 | 1 | atomic-agent |
| rotation-stable-cache-scope | 1 | 1 | hermes-agent |
| route-coverage-exerciser | 1 | 1 | opencode |
| run-span-inspection-cli | 1 | 1 | dexto |
| same-process-only-workflow-resume | 1 | 1 | grok-build |
| sandbox-absent | 1 | 1 | bitfun |
| sandbox-by-default | 1 | 1 | orca-agent |
| sandbox-escape-approval | 1 | 1 | deepseek-reasonix |
| sandbox-escape-proof | 1 | 1 | kolkrabbi |
| sandbox-fail-closed-optin | 1 | 1 | kilocode |
| sandbox-request-failopen | 1 | 1 | codewhale |
| sandbox-surface-exclusion | 1 | 1 | qwen-code |
| sanitizer-coverage-ceiling | 1 | 1 | agentty |
| sanitizer-gated-portable-ci | 1 | 1 | hax |
| sbfl-context-seeding | 1 | 1 | auto-code-rover |
| second-backend-parallel-plane | 1 | 1 | letta-code |
| security-table-contract-test | 1 | 1 | ferrum |
| self-authored-tools | 1 | 1 | claude-engineer |
| self-healing-session-store | 1 | 1 | workground2 |
| self-scored-docs-accuracy-risk | 1 | 1 | vtcode |
| semantic-contract-corps | 1 | 1 | orca-agent |
| serializable-turn-state | 1 | 1 | dexto |
| server-exposure | 1 | 1 | opencode |
| server-hosted-agent-loop | 1 | 1 | letta-code |
| session-actor-command-loop | 1 | 1 | grok-build |
| session-drain-graph | 1 | 1 | kilocode |
| session-fork | 1 | 1 | opencode |
| session-turn-lease | 1 | 1 | hermes-agent |
| session-writer-lease | 1 | 1 | qwen-code |
| shared-pure-recovery-policy | 1 | 1 | letta-code |
| shim-hosted-engine-reuse | 1 | 1 | roo-code |
| silent-threat-model | 1 | 1 | aider |
| single-breakpoint-cache | 1 | 1 | dexto |
| single-crate-runtime | 1 | 1 | grok-build |
| single-maintainer-inheritance | 1 | 1 | codex-infinity |
| single-mutation-entry-point | 1 | 1 | roo-code |
| single-owner-operation-host | 1 | 1 | orca-agent |
| single-persistence-layout-authority | 1 | 1 | workground2 |
| skill-integrity-lockfile | 1 | 1 | aeon |
| skill-whitelist-bypass | 1 | 1 | waveloom |
| snapcompact-bitmap-archive | 1 | 1 | oh-my-pi |
| spec-first-generation | 1 | 1 | developer |
| speculative-streamed-tool-execution | 1 | 1 | tura |
| split-fail-closed-fail-open | 1 | 1 | letta-code |
| split-history-ownership | 1 | 1 | trae-agent |
| sqlite-code-knowledge-graph | 1 | 1 | coro-code |
| sqlite-cron-compensation | 1 | 1 | dexto |
| ssrf-guarded-fetch | 1 | 1 | workground2 |
| stale-approval-denial | 1 | 1 | letta-code |
| stale-ci-exclusion | 1 | 1 | amazon-q-developer-cli |
| stale-required-status-checks | 1 | 1 | agentty |
| state-lifetime-partitioning | 1 | 1 | deepseek-reasonix |
| static-default-session-secret | 1 | 1 | openlumara |
| strict-sandbox-backends | 1 | 1 | orca-agent |
| structural-injection-defense | 1 | 1 | codewhale |
| subagent-allow-all-escalation | 1 | 1 | continue |
| subagent-as-persisted-child-session | 1 | 1 | mistral-vibe |
| subagent-budgeting | 1 | 1 | workground2 |
| subagent-coordinator | 1 | 1 | grok-build |
| subagent-governance-bundle | 1 | 1 | dexto |
| subagent-machinery | 1 | 1 | bitfun |
| subagent-protocol-parity | 1 | 1 | gemini-cli |
| subagent-queue-recovery | 1 | 1 | orca-agent |
| subagent-run-budgets | 1 | 1 | oh-my-pi |
| subagent-spawn-machine | 1 | 1 | opencode |
| subagent-startup-context-budget | 1 | 1 | letta-code |
| support-diagnostics-bundle | 1 | 1 | goose |
| surface-breadth | 1 | 1 | aider |
| survival-contract-schema | 1 | 1 | codewhale |
| symlink-aware-path-containment | 1 | 1 | oh-my-pi |
| tamagotchi-companion | 1 | 1 | openharness |
| targeted-package-ci | 1 | 1 | bitfun |
| task-status-control-plane | 1 | 1 | tura |
| telemetry-default-on-framing | 1 | 1 | jcode |
| telescoping-constructors | 1 | 1 | zeroclaw |
| test-addressed-facade-surface | 1 | 1 | ouroboros |
| test-corpus-real-shape | 1 | 1 | aider |
| test-coverage-completeness-checker | 1 | 1 | letta-code |
| test-edit-guard | 1 | 1 | ob-1 |
| test-name-overreach | 1 | 1 | deepseek-reasonix |
| test-only-agent-loop | 1 | 1 | binharic-cli |
| tested-approval-nothing-underneath | 1 | 1 | oh-my-pi |
| textual-tool-call-recovery | 1 | 1 | ob-1 |
| thin-published-plane | 1 | 1 | roo-code |
| thin-transport-security-services | 1 | 1 | oh-my-pi |
| thrash-aware-tool-eviction | 1 | 1 | memcode |
| thread-goal-budgets | 1 | 1 | bitfun |
| three-axis-run-budgets | 1 | 1 | mistral-vibe |
| three-mode-compaction-with-anti-signal-guard | 1 | 1 | jcode |
| three-transport-mcp-client | 1 | 1 | dexto |
| threshold-context-trigger | 1 | 1 | roo-code |
| timer-throw-uncaught | 1 | 1 | binharic-cli |
| token-economy-ceilings | 1 | 1 | bitfun |
| token-estimator-feedback | 1 | 1 | cline |
| tombstone-installer-redirect | 1 | 1 | kimi-cli |
| tool-arg-contract-validation | 1 | 1 | cursor-agent |
| tool-batch-scheduling | 1 | 1 | mini-kode |
| tool-output-masking-fifo | 1 | 1 | gemini-cli |
| tool-output-offload | 1 | 1 | opencode |
| tool-output-pruning-tier | 1 | 1 | dexto |
| tool-output-range-condensing | 1 | 1 | octomind |
| tool-pair-resume-invariant | 1 | 1 | oh-my-pi |
| torn-journal-salvage | 1 | 1 | jcode |
| trajectory-step-tagging | 1 | 1 | trae-agent |
| transport-abstracted-exec | 1 | 1 | kimi-cli |
| tree-sitter-anchor-edits | 1 | 1 | plandex |
| turn-class-prompt-tiering | 1 | 1 | zap-coding-agent |
| turn-exclusion-guard | 1 | 1 | mistral-vibe |
| turn-memory-projection | 1 | 1 | keen-code |
| turn-persistence-ordering | 1 | 1 | oh-my-pi |
| turn-step-decomposition | 1 | 1 | zeroclaw |
| two-tier-v1-compaction | 1 | 1 | opencode |
| typed-error-identity | 1 | 1 | deepseek-reasonix |
| ui-god-host | 1 | 1 | pi |
| ui-render-triplication | 1 | 1 | kilocode |
| unbounded-subagent-fanout | 1 | 1 | letta-code |
| undocumented-bind-all-control-plane | 1 | 1 | continue |
| undocumented-economy-knobs | 1 | 1 | dexto |
| undocumented-json-schema-plus-stubs | 1 | 1 | roo-code |
| unenforced-layering-law | 1 | 1 | opencode |
| unexpected-stop-rejudge | 1 | 1 | oh-my-pi |
| ungated-approval-gate | 1 | 1 | aider |
| untested-reflection-loop | 1 | 1 | aider |
| untrusted-input-framing | 1 | 1 | qwen-code |
| untrusted-result-framing | 1 | 1 | hermes-agent |
| unwired-crash-resume | 1 | 1 | dexto |
| unwired-security-validator | 1 | 1 | claw-code-agent |
| unwritten-event-journal | 1 | 1 | dexto |
| upstream-loop-assembly | 1 | 1 | deepagents |
| upstream-sync-cron | 1 | 1 | codex-infinity |
| upstream-workflow-allowlist | 1 | 1 | kilocode |
| usage-anchored-context-projection | 1 | 1 | bitfun |
| user-directive-ledger | 1 | 1 | mocode |
| v1-static-cache-heuristic | 1 | 1 | opencode |
| vendored-fork-intent-cards | 1 | 1 | kimi-code |
| vendored-fork-unpinned | 1 | 1 | memcode |
| verbatim-proprietary-prompt | 1 | 1 | claw-code-agent |
| verification-8-ceiling | 1 | 1 | oh-my-pi |
| verification-ceiling | 1 | 1 | opencode |
| verification-corpus | 1 | 1 | goose |
| verification-maturation | 1 | 1 | deepseek-reasonix |
| verification-scale-and-quality | 1 | 1 | opencode |
| verified-auto-resume | 1 | 1 | codewhale |
| verified-failure-escalation | 1 | 1 | ob-1 |
| versioned-harness-api-parity-gated-sdk | 1 | 1 | jcode |
| vestigial-refactor-tree | 1 | 1 | code |
| wal-ofd-lockguard | 1 | 1 | hermes-agent |
| wide-facade-delegation-holds | 1 | 1 | dexto |
| windows-appcontainer-sandbox | 1 | 1 | orca-agent |
| wire-compatible-headless-contract | 1 | 1 | letta-code |
| workflow-config-contract-test | 1 | 1 | deepagents |
| workflow-token-budget | 1 | 1 | qwen-code |
| write-is-the-boundary | 1 | 1 | deepseek-reasonix |
| x402-agent-payments | 1 | 1 | grok-cli |
| yolo-floors | 1 | 1 | deepseek-reasonix |
| yolo-unbypassable-failclosed-gates | 1 | 1 | oh-my-pi |
