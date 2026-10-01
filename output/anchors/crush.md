# Anchor review: crush (charmbracelet/crush, Go) — B (69.0, mid-band)

Tier: T2 boundary (census 139k non-test LOC; test_loc=0 is a **measurement artifact** — 278 `*_test.go` files, 61,738 LOC, 1,731 Test funcs). Full history: 171 contributors, 4225 commits. MIT.

**Closest anchor: n/a — is anchor.** Mid-band reference: competent standard design everywhere, defended nowhere; the B-band "solid but not enforced" centroid (codex is B+19 above it, nanocoder C+24 below).

## Dimensions

### architecture — 7
- Client/server split with a proto boundary (`internal/server`, `internal/client`, `internal/proto`), all state in sqlite via sqlc + migrations (`internal/db/`, `sqlc.yaml`), pubsub core (`internal/pubsub`), coordinator pattern (`internal/agent/coordinator.go`).
- Deduction: `agent.go` 2393 LOC and `coordinator.go` 1877 LOC are god files holding loop + policy + tool wiring together; `internal/` is a 40-package flat grab bag.

### verification — 7
- 61,738 LOC across 278 test files (census said zero — wrong). Property-shaped suites: `dispatch_race_test.go`, `coordinator_mcp_gate_test.go`, `loop_detection_test.go`, `permission_test.go`.
- CI: `build.yml:18` 3-OS matrix, :29-30 `go build -race` + `go test -race -failfast`; `security.yml:59-73` govulncheck. Race testing across agent dispatch is real property verification.
- Deduction: no evals, no fuzzing, TUI layer (the largest product surface) essentially untested.

### safety-enforcement — 6
- Permission service prompts per tool call and binds execution (`internal/permission/permission.go`, `internal/agent/hooked_tool.go`), with its own tests (`permission_test.go`).
- Best-in-corner detail: hook pre-approvals are scoped to a single tool-call id via an unexported context key so an approval cannot be replayed across calls (`permission.go:15-29`).
- Shell-command hooks on Claude-compatible events (`internal/hooks/hooks.go:1-15` `EventPreToolUse`).
- The gap: no OS-level sandbox anywhere (`grep sandbox` finds no module); approved bash runs as the user. Standard design, tested — a 6, same class as pi/cline.

### token-economy — 6
- Auto-summarize compaction wired into the loop with an unknown-context-window guard (`agent.go:181` Summarize, :832, :1093 "if context window is unknown (0), skip auto-summarize") and a disable setting (:216); channel-turn goal explicitly survives a summarize continuation (`agent.go:136-138`).
- Usage accounting with fallbacks (`usage_fallback_test.go`).
- Deduction: no cache discipline, no tiers, no cost visibility beyond raw token counts.

### orchestration — 7
- Subagents via `agent_tool.go`; loop detection breaks stuck tool-call repetition (window 10 / >5 repeats, `loop_detection.go:11-40`, tested) — the only anchor with a first-class anti-loop guard.
- Turn cancellation and dispatch races are tested (`dispatch_cancel_test.go`, `dispatch_race_test.go`); sessions persist in sqlite so resume survives process death.
- Deduction: no budgets, no queues, no scheduling.

### interop — 7
- MCP as tool directory with gating tested (`internal/agent/tools/mcp/init.go`, `coordinator_mcp_gate_test.go`); LSP client integration (`internal/lsp/`) is rarer and more useful than most; client/server proto enables external drivers.
- Deduction: no ACP, no IDE extension surfaces, no published SDK.

### operability — 8
- Crash posture: recover middleware on the server (`internal/server/recover.go:18`), shell runner (`internal/shell/run.go:63`), and logger (`internal/log/log.go:67`); single-instance data-dir lock (`internal/db/datadirlock.go`).
- Sessions/resume in sqlite, broad auth incl. AWS SSO refresh (`internal/agent/aws_sso_refresh.go`), OAuth/login/logout packages, self-update (`internal/update`), JSON-schema-validated config (`schema.json`).

### originality — 7
- Loop detection enforced in core (seeded concept, first anchor); hook pre-approval tool-call scoping; skills manager with a tracker (`internal/skills/tracker.go`); `agentic_fetch_tool.go`; herdr external assistant service (`internal/herdr/`).
- Deduction: much of the design is the opencode lineage (see calibration), with Charm's polish rather than novel mechanism.

### durability — 8
- Funded institution (Charm) with CLA, release/nightly/snapshot CI, govulncheck cadence; 171 contributors, full history.
- Risk: the original author's line continues as opencode — this is Charm's divergent continuation, so longevity is Charm's, not shared.

### docs-dx — 6
- In-repo docs cover config and hooks only (`docs/`); README is a product page; onboarding relies on `crush.json` schema discovery. No SECURITY.md; install one-liner fine.

## Verdict
**Strongest: operability (8).** Weakest: docs-dx (6).
Weighted 69.0 -> **B**. B-mid anchor: everything present, one standard design per area, race-tested CI, but no dimension reaches "defended" (8) and nothing here is field-defining. Census corrections: test_loc 0 -> 61,738; provenance "original" understates opencode lineage (`internal/agent/opencode_routing_test.go`).
