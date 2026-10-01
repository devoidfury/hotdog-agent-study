# wave6/roadmap-ranking.md -- candidate arithmetic (mechanical, from findings.jsonl)

Counts recomputed 2026-10-01 from findings.jsonl (1,666 records, 104 subjects). Candidates =
kind in {portable, safety-hole}: 754 records. score = conv_subjects (all kinds, matches
convergence.md headers) x thesis_fit_sum (T1 zero-dep, T2 solo-sustainable, T3 user-paid-token
benefit, T4 machine-checked-orchestration). Effort from records; "(est)" where the field is null.

| # | concept | conv | T | fit | score | effort | citations | build note (landing module) |
|---|---|---|---|---|---|---|---|---|
| 1 | faux-provider-testing | 47 | 1111 | 4 | 188 | M | ob-1-4, SWE-agent-3, cline-2, hax-4 | drive the real bin/hotdog binary behind a scripted wire, not just in-process mocks (tests/) |
| 2 | compaction-tiering | 43 | 1110 | 3 | 129 | M | crab-code-1, minicode-3, jazz-b8 | order the existing 5 strategies cheapest-first with a breaker (compaction extension) |
| 3 | permission-policy | 61 | 1100 | 2 | 122 | M (S for the flip) | crab-code-3, dvalincode-1, atomic-agent-4 | ship the existing gate ON by default (user-gate/extension.json + approvals) |
| 4 | evals-in-ci | 29 | 1111 | 4 | 116 | M/L | ouroboros-5, kilocode-b5, jcode-s7 | one blocking cost-capped live-model gate, no continue-on-error (CI) |
| 5 | loop-detection | 36 | 1110 | 3 | 108 | S-M | 3code-2, crush-1, kimi-cli-2 | add result-byte + near-dup signals to the canonical-hash detector (loop-detect) |
| 6 | prompt-cache-marking | 29 | 1110 | 3 | 87 | S | memcode-5, ob-1-8, keen-code-2 | stable/volatile split + breakpoint placement (llm-client serialize) |
| 7 | workflow-resume-journal | 19 | 1111 | 4 | 76 | L | kolega-code-2, qwen-code-e5, hotdog-4 | content-addressed node keys + journal the plain-subagent path (workflows engine) |
| 8 | hook-trust-scoping | 25 | 1101 | 3 | 75 | S | bitfun-s4, claurst-5, pi-4 | trust gate for workspace-sourced extensions/hooks/config, content-hashed (extensions) |
| 9 | usage-measured-compact-trigger | 23 | 1110 | 3 | 69 | S | opencode-e1, kilocode-e1, gemini-cli-e2 | provider-usage feedback loop over our wire-size projection (compaction utils) |
| 10 | cache-monotonic-compaction | 19 | 1110 | 3 | 57 | M | atomic-agent-3, ouroboros-4, mocode-3 | byte-stable prefix invariant + counted cache breaks (context/compaction) |
| 11 | grammar-parsed-bash-policy | 16 | 0111 | 3 (T1=0) | 48 | L | ferrum-1, smelt-2, memcode-1 | concept-only: extend the hand tokenizer + fuzz it; tree-sitter exemplars FAIL T1 (approvals/bash.ts) |
| 12 | security-posture-docs | 24 | 1100 | 2 | 48 | S | ferrum-9, openhands-8, minicode-10 | mode-by-mode table naming what does NOT bind (docs) |
| 13 | phantom-safety-control | 15 | 1101 | 3 | 45 | S | binharic-cli-2, claurst-b1, dexto-s2 | CI contract test: advertised enforcement knob has a live call site + matching default |
| 14 | architecture-contract-test | 11 | 1101 | 3 | 33 | S | kode-cli-8, kolkrabbi-7, minicode-7 | import/boundary ratchet + docs-map drift gate in CI |
| 15 | headless-contract | 7 | 1101 | 3 | 21 | M (est) | opencode-e10, grok-build-e14, roo-code-e11 | document -p/--json/--json-schema as a versioned machine contract (ui-one-shot) |
| 16 | turn-budget-accounting | 5 | 1111 | 4 | 20 | S | codewhale-e6, code-e4 | per-run token/spend cap that hard-stops and never reports success (TaskManager) |

Cut above the line (score, reason): checkpoint-revert 27x3=81 (operability theme, not one of the
three sub-sections), crash-recovery-posture 13x4=52, lazy-skill-loading 11x3=33 (low conv, no theme owner), network-egress-approval 8x3=24 (effort L, needs proxy),
fail-closed-default-decision 6x3=18 (our gate matrix already corpus-class on this point).

Non-moves (verified counts): acp-* concepts 13 subj/14 findings; mcp-client-only 5 +
interop-ceilings 4; no-published-sdk 1 (hermes-agent-e10) + interop-outbound-and-sdk-missing 1
(vtcode-e9); god-file-loop 25 + god-file-host-wiring 13 + ui-coupled-loop 4 + code-c2 44,921 LOC
chatwidget.rs; eval-harness-outside-ci 7 + private kilo-bench flag (calibration-notes:1040);
sandbox-delegation 32 + sandbox-absent 5 + windows-sandbox-gap 3 + phantom-safety-control 15;
provider-plane-fork 2 + rival-subscription-transport 4 + oauth-client-impersonation 3 (continue-e7
67 provider adapter files); injection-screening 14 subj but 1 portable.

Three-move projection (weights from scores/hotdog.json: safety 10, verification 15, token 10):
68.5 -> safety 5->6 = +1.0 -> 69.5 -> verification 7->8 = +1.5 -> 71.0 -> token-economy 7->8
= +1.0 -> 72.0.
