# Verification Summary: C1

| Field | Value |
|---|---|
| Task | AAV-20261121-RUST-SKILLS |
| Baseline | BASELINE-0003 (REQ-0108@r6 via CR-0002, REQ-0209@r6 via CR-0001, other REQs @r5) |
| Risk class | L1 |
| Independence | I2: role-separated `red-team-verifier` agent contexts that did not read the builders' workspaces or rationale. This is AI verification, not I3/I4 assurance. |
| Detailed reports | `.agile-v/verification/VERIFY_ART-0001.md`, `.agile-v/verification/VERIFY_ART-0002.md` (round 3) |

## Results

| Artifact | sha256 | R1 | R2 | R3 (final) | Open findings |
|---|---|---|---|---|---|
| ART-0001.3 `build-agent-rust/SKILL.md` | `26dcc510d709e5efc6ad3f74e6c618bc582be13e7a6600c87da0689572c4b1a6` | PASS w/ findings (1 major, 6 minor) | PASS w/ findings (3 minor) | **PASS** | none (O-04 observation: proptest does not build for wasm32-wasip2; no REQ affected) |
| ART-0002.4 `build-agent-embedded-rust/SKILL.md` | `b78eca250901c50ba90bb6100190edf2c493a021c5a4bb1977179ba85670469c` | PASS w/ findings (5 minor) | PASS w/ findings (2 minor) | **PASS** | none (O-02 observation: an embedded-test macro emits a cfg warning in test builds; no evidence item affected) |

Every Verification Criteria and Done Criteria item for REQ-0001 to REQ-0011, REQ-0101 to REQ-0112, and REQ-0201 to REQ-0212 passes against BASELINE-0003.

## Key evidence (round 3)

- **REQ-0001:** pytest gives 1050 passed and 24 skipped, identical on the working tree and on HEAD 4bcc917. No tracked changes outside `.agile-v/`.
- **REQ-0010:** every non-Fragment example passes its file-type check: 24 in ART-0001.3 and 16 in ART-0002.4. `cargo deny check` passes. Negative controls confirm the configuration is in effect.
- **REQ-0011:** the routing text is byte-identical in both files, and all 12 cases route as specified.
- **REQ-0105:** stalled-transport deadline test, 5/5 pass.
- **REQ-0108@r6:** the per-crate MSRV checks pass at 1.85 and 1.94. Control runs fail as expected.
- **REQ-0112:** `no_std` thumbv7em builds pass, `wasm-pack test --node` passes, and `cargo test --target wasm32-wasip2` with wasmtime 49.0.7 passes.
- **REQ-0209@r6:** the warning gate fails on an injected warning with the unquoted dotted triple.
- **On-target test Fragment:** builds with `--no-run` using the documented setup. Adding a second panic handler fails as the skill states.

## Not run (environment limits): known residual risk

- Miri, `cargo fuzz run`, Kani: no nightly toolchain, cargo-fuzz or kani installed.
- `cargo call-stack`: no nightly installed. The text was checked against the 0.1.16 source.
- On-target tests, HIL, probe-rs stack painting, timing: no hardware attached.
- thumbv6m, thumbv7em, and riscv32imac firmware builds. Only the STM32H563 (thumbv8m.main) example was built.
- wasm-pack 0.15.1: the local 0.14.0 was used.
- pytest ran with unpinned packages on Python 3.14.
- Ecosystem drift: the examples are pinned to versions as of 2026-11-21. Advisory results depend on the advisory database date. For example, ctutils needs `cargo update --precise`, as the skill documents.
