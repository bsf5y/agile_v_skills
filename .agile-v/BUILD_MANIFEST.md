# Build Manifest: C1

Task: AAV-20261121-RUST-SKILLS | Risk: L1 | Baseline: BASELINE-0001 | Toolchain: rustc/cargo 1.96.0 | AI_RUN_MANIFEST: .agile-v/aibom/AAV-20261121-RUST-SKILLS/AI_RUN_MANIFEST.yaml

ART-0001 | REQ-0001..0011@r5, REQ-0101..0112@r5 | BASELINE-0001 | implements | .agile-v/staging/build-agent-rust/SKILL.md | 876 lines, 39,133 B; 24 artifact examples (1 Fragment); build-time compile checks exit 0 (cargo-deny etc. not run)
ART-0002 | REQ-0001..0011@r5, REQ-0201..0212@r5 | BASELINE-0001 | implements | .agile-v/staging/build-agent-embedded-rust/SKILL.md | 794 lines, 38,360 B; 17 artifact examples (2 Fragments); STM32H563ZI / thumbv8m.main-none-eabihf; HALT: CR-0001 (REQ-0209 quoting defect)

## CR-0001 update (BASELINE-0002)

ART-0001 | REQ-0001..0011@r5, REQ-0101..0112@r5 | BASELINE-0002 | implements | .agile-v/staging/build-agent-rust/SKILL.md | C1 | — | carried forward unchanged; not affected by CR-0001
ART-0002.2 | REQ-0001..0011@r5, REQ-0201..0212@r5, REQ-0209@r6 | BASELINE-0002 | implements | .agile-v/staging/build-agent-embedded-rust/SKILL.md | C1 | CR-0001 | one-line correction to the dotted-triple rule; traceability header updated

## Final (BASELINE-0003, post-verification)

ART-0001.3 | REQ-0108@r6; REQ-0001..0011@r5, REQ-0101..0107@r5, REQ-0109..0112@r5 | BASELINE-0003 | implements | .agile-v/staging/build-agent-rust/SKILL.md | C1 | CR-0002 | 898 lines, 42,522 B; sha256 26dcc510d709e5efc6ad3f74e6c618bc582be13e7a6600c87da0689572c4b1a6; verified PASS (R3)
ART-0002.4 | REQ-0001..0011@r5, REQ-0201..0212@r5, REQ-0209@r6 | BASELINE-0003 | implements | .agile-v/staging/build-agent-embedded-rust/SKILL.md | C1 | CR-0001 | 830 lines, 41,904 B; sha256 b78eca250901c50ba90bb6100190edf2c493a021c5a4bb1977179ba85670469c; verified PASS (R3)
