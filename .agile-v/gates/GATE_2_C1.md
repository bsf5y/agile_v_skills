# Human Gate 2 Approval Record: C1

| Field | Value |
|---|---|
| Gate | Human Gate 2 (release) |
| Task | AAV-20261121-RUST-SKILLS |
| Approver | Wes Cravens (stakeholder) |
| Date | 2026-11-21 |
| Decision | **APPROVED**: the stakeholder explicitly selected "Approve release" in a structured Gate 2 prompt |
| Baseline | BASELINE-0003 |
| Risk class | L1 |

## Released artifacts

| ART | Path | sha256 | Verification |
|---|---|---|---|
| ART-0001.3 | `.agile-v/staging/build-agent-rust/SKILL.md` | `26dcc510d709e5efc6ad3f74e6c618bc582be13e7a6600c87da0689572c4b1a6` | PASS (round 3), `.agile-v/verification/VERIFY_ART-0001.md` |
| ART-0002.4 | `.agile-v/staging/build-agent-embedded-rust/SKILL.md` | `b78eca250901c50ba90bb6100190edf2c493a021c5a4bb1977179ba85670469c` | PASS (round 3), `.agile-v/verification/VERIFY_ART-0002.md` |

## Accepted residual risks

- Miri, cargo-fuzz, Kani, cargo-call-stack, on-target/HIL, stack painting and timing guidance were not executed here; the text was checked against documentation and source only.
- Only one firmware target was built: STM32H563, `thumbv8m.main-none-eabihf`.
- Ecosystem versions are pinned as of 2026-11-21. Version tables need periodic refresh, and advisory results depend on the advisory database date.
- Verification independence is I2: AI agents in separate, role-separated contexts. No I3/I4 human or external assurance.

## Release and rollback

- **Release:** the stakeholder moves both directories to the destination project. No automated deployment was performed by the agent.
- **Rollback:** nothing is integrated into this repository, so rollback means not using, or deleting, the staged directories. To check integrity after the move, verify the files against the hashes above.
