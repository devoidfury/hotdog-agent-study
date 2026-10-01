# open-interpreter -- differential triage (T0) vs codex (anchor S 88.5)

Anchor sentence: closest to codex because it is codex -- same Rust monorepo tree, rebranded.

## Verdict: rename-clone at the identity layer; in practice a documented "distribution fork" with a provider-layer delta

- Census flag `rename-clone:codex` mechanically upheld: root `package.json` name is still `"codex-monorepo"`, 4,072 files hash-identical to current codex (census), README.md:47 "Open Interpreter is a fork of OpenAI's Codex".
- But NOT a deceptive rename: `FORK_BRANDING.md:1-20` is an explicit branding policy ("inherits internal crate, protocol, and compatibility names... Do not globally replace `codex`"), user-facing identity centralized in own crate `codex-rs/product-info/src/lib.rs`, and `codex/LICENSE`/`NOTICE` are byte-identical to upstream (`diff -q` clean) -- Apache-2.0 attribution intact.
- Baseline drift: based on upstream "rust-v0.154.0 compatibility baseline" (RELEASE_NOTES.md:3-4); vs today's codex tree 2,766 files differ, 1,415 upstream files absent, 275 fork-only files.
- Real delta (fork-only files): `codex-rs/acp-server` crate + `cli/tests/acp_protocol.rs` (ACP surface), Anthropic wire support (`codex-api/src/anthropic.rs`, `endpoint/anthropic_messages.rs`, `sse/anthropic.rs`), model compatibility catalog (`model_compatibility_catalog.json`), harness catalog (`core/src/harness`), `kimi_cron*` scheduled runs, `guardian/assessment.rs`, `ThreadRollback` protocol, `cli/tests/product_identity.rs`.
- Loop/compaction/permission spot read: core loop and sandbox crates are the codex 0.154-baseline copies (`core/src/session/turn.rs` present, sandboxing crates present); fork work is transport/provider, not loop surgery.

## Scoring

Honest code-observed scoring: arch 9, verif 8 (codex suite carried: 377k test LOC), safety 10 (codex sandbox at baseline), token 9, orch 8 (kimi_cron added, some upstream orchestration absent at baseline), interop 9, oper 9, originality 2 (nearly everything visible already exists in-corpus via codex; provider/harness work is config-plane), durability 5 (funded org behind it, but v0.0.43 single-contributor rebrand pinned to upstream), docs-dx 7 (polished multilingual README + docs site). **Weighted 78.5.**

## Band D by policy cap, recorded not silent

Dispatcher rule applied: rename-clone of an in-corpus giant -> band D + study-integrity findings, pending a possible divergent-fork re-review. My evidence says "honest distribution fork", so the cap is policy, not code-quality; honest weighted 78.5 would be A (rule (a)-compatible). Logged in `calibration-notes.md`.

## License findings

No copyright violation found (Apache attribution preserved). Residual risks recorded as license-risk/anti-pattern findings: ships OpenAI-branded prompt assets and `codex-monorepo` manifest + chatgpt.com install URLs (`product-info/src/lib.rs:12-32`) under a rival product name -- provenance/trademark exposure and clone-detector misfire, respectively. No code-adjacent portables recommended (anything "portable" here is codex, already reviewed in-corpus).
