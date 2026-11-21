# Decision Log

| Date | ID | Decision | Rationale | Ref |
|---|---|---|---|---|
| 2026-11-21 | DEC-0001 | Gate 1 approved for requirements r5; frozen as BASELINE-0001 | Logic Gatekeeper rated r4 ready (3 minor findings); stakeholder elected to fix the minors in r5 and approve without another review round | `.agile-v/gates/GATE_1_C1.md` |
| 2026-11-21 | DEC-0002 | CR-0001 approved: REQ-0209 r6 now requires unquoted dotted triples; re-baselined as BASELINE-0002 | Empirical cargo 1.96.0 test showed quoted dotted-triple keys are silently ignored, which made the L1 `-D warnings` evidence vacuous | `.agile-v/gates/GATE_1_C1_CR-0001.md`, `.agile-v/CHANGE_LOG.md` |
| 2026-11-21 | DEC-0003 | CR-0002 approved: REQ-0108 r6 per-crate MSRV check; re-baselined as BASELINE-0003; remaining minor findings to be fixed and re-verified before Gate 2 | A single workspace MSRV command fails for workspaces with mixed rust-versions and for binary-only crates; the per-crate form was verified by red-team | `.agile-v/gates/GATE_1_C1_CR-0002.md` |
| 2026-11-21 | DEC-0004 | Gate 2 approved: release ART-0001.3 and ART-0002.4 from staging; residual risks accepted | All REQs in BASELINE-0003 verified PASS (I2, 3 rounds); stakeholder accepts untested nightly/hardware tooling, single-MCU build, version drift, and AI-only verification | `.agile-v/gates/GATE_2_C1.md` |
