# kilocode verification-9 re-audit (synthesis-mandated, 2026-09-30)

Scope: single dimension. Read-only against /data/samples/agents/kilocode @ snapshot.
Mandate per calibration-notes.md:1005/1024 ("the one outstanding verification-9 in the corpus
below the A-anchors and it decides a band"; blocking-only rule, grinta precedent).

## (a) What the "release-blocking smoke gate" mechanically is

- Workflow: `.github/workflows/smoke-test.yml` (`name: smoke-test`, job `smoke-test` :40).
  Callable two ways: `workflow_dispatch` (:18-24) and `workflow_call` (:25-31).
- Release wiring: `.github/workflows/publish.yml:585-596` -- job
  `smoke-test: Smoke Test (pre-publish gate)` with
  `uses: ./.github/workflows/smoke-test.yml`, `secrets: inherit`,
  `if: github.repository == 'Kilo-Org/kilocode'` (true on the shipping repo),
  `cli_version` fed from `needs.version.outputs.version` -- i.e. the gate tests the
  draft-release assets of the very release being published (`gh release download` at
  smoke-test.yml:84-101).
- Blocking edge: `publish.yml:599-607` -- `publish` job `needs: [..., smoke-test]`
  (:606), no `if:`, no `continue-on-error` on either job. The ONLY `continue-on-error` in
  publish.yml is :170 (SBOM scan, itself conditional on `vars.SBOM_ENFORCE != 'true'`).
  Inside smoke-test.yml there is NO `continue-on-error` anywhere; every step, including the
  two eval steps and the results-validation step, fails the job on non-zero exit.
  A failed smoke-test job => `publish` (npm/OCI/VSIX/GitHub release finalize) never runs.
  Failing closed on missing secrets is explicit (`Validate API key` :75-79 `exit 1`).
- This is NOT the other, deterministic smokes in the release: `publish.yml:246`
  (`--pure __pty-smoke`, no model) are host-run binary-startup checks; they exist and are
  correctly NOT what is credited here.

Answer (a): job `smoke-test` (smoke-test.yml) consumed by publish.yml:585-596 and required
by publish.yml:606; the release workflow FAILS (skips publish) on it. Blocking, in-file,
the exact "workflow line that blocks" required by calibration-notes.md:148.

## (b) Real model behavior?

- smoke-test.yml:112-131 runs Harbor eval tasks against the built CLI with a LIVE model:
  `./scripts/run_eval.sh -m kilo/anthropic/claude-sonnet-4.6 -d hello-world` (:116-120) and
  `--include-task-name "log-summary-date-ranges"` (:126-131), via Kilo Gateway
  (`KILO_API_KEY`/`KILO_ORG_ID` :46-47), bench checkout of private Kilo-Org/kilo-bench
  (:54-58), cost header "hello-world (~$0.01), log-summary-date-ranges (~$0.13), expected
  total < $0.50" (:5-6) -- live token spend, per-task trajectories and `result.json`
  captured (:143-148). Then `Validate results` (:133-134,
  `python3 scripts/validate_smoke_test.py jobs/smoke-test*/`), a normal (non-tolerant) step.
- Not zeroclaw-class replay: no stubbed provider, no canned traces; real agent binary, real
  provider, scored tasks. Not the prime-agent/openhands "deterministic smoke" misread --
  those subjects' precedents, verified from their own score files:
  - prime-agent: scores/prime-agent.json:22 "in-CI behavioral eval gate at
    .github/workflows/behavioral-evals.yml:430 (fail-closed verdict on 28-task base-vs-head
    SWE comparison)". Verified in sample: behavioral-evals.yml:428-430
    `test -f results/verdict && test "$(cat results/verdict)" = pass` -- per-task steps are
    continue-on-error (:229-343) but the verdict step hard-fails. Real-model, blocking, clean 9.
  - openhands: scores/openhands.json:22 "scripted-provider E2E on main (mock-llm-e2e.yml:90-134)
    and live-LLM E2E on PRs (ci.yml:265-276); ... demote to 8 => 70.5, band unchanged". So the
    openhands 9 is PARTLY scripted-provider and was honestly carried flagged; the boundary cited
    it as "9 flagged" -- accurate, not misremembered, but it is the weak half of the precedent.
  - gemini-cli: scores/gemini-cli.json:22 + calibration-notes.md:282 "evals-in-CI but softest
    blocker of the four: eval-gate is a human click, not a build failure, and
    USUALLY_PASSES/USUALLY_FAILS cases skip unless RUN_EVALS". Under the gptme rule's manual-
    trigger clause this 9 is the corpus's most contestable -- but it is band-safe either way
    (84.5 -> 83.0, A) and it is SOFTER than kilocode's gate (human approval to run; kilocode's
    runs unattended inside the release graph and no click can bypass the needs-edge).
- gptme/deepagents telemetry rule (calibration-notes.md:130-132): "evals in CI count toward
  verification 9 only if the job BLOCKS on failure; continue-on-error evals = existence of
  harness, not enforcement." kilocode: no continue-on-error, real needs-edge -- passes.
- grinta precedent (calibration-notes.md:1010, final 74.0 B): fell because "trigger pipeline
  exists but is not a blocking eval gate". Distinguishing fact confirmed: grinta's run-eval.yml
  had no job consuming its verdict; kilocode's publish consumes smoke-test's via needs at
  publish.yml:606. grinta's fall does NOT reach kilocode; it is the counter-example, not the
  controlling one.
- Main-review withhold check: subjects/kilocode.md:17 withheld 9 because "the only model eval
  in CI is release-gated... no fuzz targets". That rationale is inconsistent with the corpus's
  own accepted entries: prime-agent's 9 IS release-gated (label-gated) and ouroboros's nightly
  ($30 cap, ci.yml:9,631; ledger rank 3) is nightly-only -- neither is PR-gated. Fuzzing is the
  9->10 gate, never required for 9 (anchors errata + prime-agent "no fuzzing, so not 10").

## (c) Verdict

Kilocode's gate satisfies every element of the confirmed rule as literally stated -- model
evals, in CI, blocking -- with stronger in-file blocking mechanics than two ledger entries the
corpus kept at 9 (gemini-cli human-click, openhands flagged smoke). It is weaker than
prime-agent on breadth (2 tasks vs 28, one model) and carries one genuine evidence caveat the
9 must stay flagged for: the pass-threshold semantics (`run_eval.sh`,
`validate_smoke_test.py`) live in the private kilo-bench repo and are not snapshot-verifiable,
same trust class as openhands' flagged acceptance assertions. Ledger placement:
smelt fuzz > prime-agent gate > ouroboros nightly > kilocode release smoke (2-task, private
threshold) >= gemini-cli human-click > openhands smoke -- bottom tier of 9s, acceptance-grade,
flag retained. The main review's 8 rests on a PR-gate requirement the corpus never adopted;
the boundary's 9 rests on the correct mechanism, correctly flagged. Neither prior review
otherwise moves: the boundary-vs-main dimension deltas (arch 8/7, orig 7/8, token 7.5/8,
orch 7.5/8) net to exactly 0.0, so verification was -- and remains -- the entire 78.5/77.5 gap;
no other dimension requires re-examination.

Band consequence: 9 stands -> boundary 78.5 A stands (A floor held by 0.5; sensitivity in
boundary/kilocode.md:221-223 already disclosed). Distribution note for synthesis: A count
unchanged; kilocode remains the ledger's weakest non-flagged-class mechanism claim with a
flagged evidence tail; recommend the synthesis sweep also re-look at gemini-cli's human-click
9 (band-safe) so the ledger's softest entry is dispositioned alongside this one.

VERIFICATION FINAL: 9 | BAND: A
