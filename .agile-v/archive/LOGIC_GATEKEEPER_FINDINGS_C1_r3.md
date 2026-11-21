# Logic Gatekeeper Independent Findings
<!-- Target: .agile-v/REQUIREMENTS.md | Revision set: r3 (architect_revisions) | Cycle: C1 | Reviewer: logic-gatekeeper (independent context) | Date: 2026-11-21 -->
<!-- Prior: r1 and r2 findings archived under .agile-v/archive/ -->

Reviewed:
- REQ-0001 to REQ-0011 @r3
- REQ-0101 to REQ-0112 @r3
- REQ-0201 to REQ-0212 @r3
- A-07, A-08, and the SD evidence note

Skipped: none | Requirements edited: no | Baseline edited: no

## Verdict

**PASS WITH FINDINGS.** No blocking findings remain. There is 1 major finding (LG-201) and 5 minor ones.

- **r2 findings:** all 16 (LG-101 to LG-116) are resolved in the r3 text.
- **r1 resolutions:** no regressions found.
- **LG-201 (major):** an ambiguous wording in the REQ-0011 routing rule. It can misroute an in-scope case that SD-08 explicitly assigns. The fix is one line.

Gate 1 readiness:
- **Preferred:** the Requirement Architect applies the LG-201 one-line fix, as r3.1 or as a recorded Gate 1 amendment.
- **Alternative:** the human records an explicit disposition of LG-201 at Gate 1.
- **Either way:** r3 is acceptable for Gate 1, with LG-202 to LG-206 carried as known minor items.

None of the minors blocks synthesis.

---

## Status of r2 findings (checked against the r3 text)

| r2 ID | Status | Verification note |
|---|---|---|
| LG-101 | Resolved | A-07 sets the MSRV floor at 1.85 and edition 2024. REQ-0103 now uses only `#[expect(..., reason)]`, which is valid at that floor, and the fallback is gone. REQ-0102 and REQ-0109 #9 halt on a lower MSRV. |
| LG-102 | Resolved | REQ-0011 drops `#![no_main]` as an indicator and adds the cargo-fuzz case. REQ-0212 states it is not a trigger. |
| LG-103 | Resolved | Routing is decided per workspace. The host-testable crate routes to embedded, and a baselined REQ can designate a host-product exception. A residual edge case is in LG-202. |
| LG-104 | Resolved | REQ-0203 sets `test = false` / `bench = false`. REQ-0209 L1 runs firmware clippy without `--all-targets` and host-crate clippy with `--all-targets`. |
| LG-105 | Resolved | REQ-0203 emits the link args from `build.rs` (`cargo:rustc-link-arg`), so `RUSTFLAGS` cannot drop the linker scripts. A residual is in LG-203. |
| LG-106 | Resolved | REQ-0010 adds a check table per file type. A residual inconsistency in the verification text is in LG-204. |
| LG-107 | Resolved | New REQ-0112 covers no_std libraries and WASM. The commands are technically sound: `wasm32-wasip2`, `wasm-pack test --node`, and the no_std cross build. Residuals are in LG-205. |
| LG-108 | Resolved | A-07 is in place. REQ-0102 uses `[lints] workspace = true` for members and `[lints]` for single packages. |
| LG-109 | Resolved | REQ-0108 L3 requires I2 `red-team-verifier` (halt if absent) and an I3 human sign-off in the Gate 2 approval record. It also states that AI verification alone is not independent assurance. |
| LG-110 | Resolved | REQ-0204 places fault handlers in the BSP crate's authorized startup module. This is viable because cortex-m-rt exception symbols resolve from any linked dependency crate. |
| LG-111 | Resolved | A-08 makes ESP-IDF std and UEFI halt. aarch64-none matches the `*-none` indicator and then hits the REQ-0210 #9 HAL halt. |
| LG-112 | Resolved | A-08 puts Xtensa out of scope, and REQ-0201 is limited to stable-Rust targets. A stale mention remains (LG-206). |
| LG-113 | Resolved | REQ-0005 defines the label as the "last non-blank line before the opening fence", with exact syntax, at most one label, and stated exemptions. |
| LG-114 | Resolved | REQ-0011 makes every cross-skill route conditional ("if installed; otherwise halt and ask"). |
| LG-115 | Resolved | The evidence note records the source of each SD and requires re-confirmation at Gate 1. This reviewer cannot independently inspect the session prompts, so the Gate 1 re-confirmation is the control. |
| LG-116 | Resolved | REQ-0108 uses `--lib --bins`. REQ-0209 states `cargo +<rust-version> check --release --target <triple>`. |

## r1 regression spot-check

| r1 ID | Still resolved? | Note |
|---|---|---|
| LG-001 | Yes | Staging path unchanged. REQ-0001 still requires the pytest result to be unchanged. |
| LG-002 / LG-007 | Yes | REQ-0104 is unchanged, and REQ-0204 still matches REQ-0104(a). |
| LG-005 | Yes | Every evidence item in REQ-0108, REQ-0209, and REQ-0112 has a command or method plus a threshold. |
| LG-009 | Yes | A-04 is in place, and the L0 and out-of-scope statements are intact. |
| LG-010 | Yes | Typed lineage is preserved for every REQ, and the new REQs carry `derives_from`. |
| LG-012 | Yes | Strengthened by LG-113. |
| LG-016 | Yes | Strengthened by LG-114. |
| LG-017 | Yes | Superseded by A-07: the 1.85 floor covers resolver 3, `[lints]`, and lint reasons. |

---

## New findings (r3)

### LG-201: Firmware indicators over-match: `rust-toolchain.toml` target lists and `wasm32v1-none`
- **Affected:** REQ-0011@r3 (vs REQ-0112@r3, SD-08), REQ-0212@r3
- **Severity:** major (ambiguity in the deterministic routing rule; misroutes an in-scope case)
- **Issue:** Two parts of the rule over-match:
  - **"`rust-toolchain.toml` sets a bare-metal target":** `rust-toolchain.toml` never sets a build target. Its `targets = [...]` key only lists targets for rustup to install. REQ-0112 requires standalone no_std libraries to run `cargo build --target thumbv7em-none-eabihf`, and such crates commonly list `thumbv7em-none-eabihf` in `rust-toolchain.toml` so that the target is installed. A literal reading then classifies a standalone no_std driver library as firmware. That contradicts SD-08 and the REQ-0011 case "standalone `no_std` driver lib → rust".
  - **The `*-none` glob:** it also matches `wasm32v1-none`, a stable tier-2 no_std WASM target since Rust 1.84. Under SD-08, WASM routes to `build-agent-rust`.
- **Recommendation:**
  - Restrict the target indicator to `.cargo/config.toml` `[build] target` (or `CARGO_BUILD_TARGET`) set to a bare-metal triple.
  - State that `rust-toolchain.toml` `targets` alone is **not** an indicator.
  - Exclude `wasm32*` from the `*-none` glob.
  - Add these routing cases:
    - a no_std driver library whose `rust-toolchain.toml` lists `thumbv7em-none-eabihf` → rust
    - a `wasm32v1-none` library → rust

### LG-202: The firmware-project boundary misses host-testable crates kept outside the firmware workspace
- **Affected:** REQ-0011@r3 vs REQ-0203@r3
- **Severity:** minor
- **Issue:** REQ-0203 prefers a host-testable crate "outside the target-configured directory". A common layout is a separate workspace or sibling directory that the firmware crate pulls in as a path dependency, such as `core/` next to `firmware/`. That crate has no firmware indicators of its own, so REQ-0011 routes it to `build-agent-rust`. That defeats the LG-103 resolution in exactly the layout REQ-0203 recommends. Same-workspace layouts are handled correctly.
- **Recommendation:** Define a firmware project as the workspace containing a firmware crate **plus any crates the firmware crate depends on by `path`**. Add the case "path-dependency core crate in a sibling directory → embedded".

### LG-203: `RUSTFLAGS` still drops non-link `target.<triple>.rustflags`; commands are missing for the explicit-host-target alternative
- **Affected:** REQ-0209@r3 (L1), REQ-0203@r3
- **Severity:** minor
- **Issue:**
  - REQ-0203 moves only the link args to `build.rs`. Any other `target.<triple>.rustflags`, such as `-C target-cpu=...`, `--cfg ...`, or `-C force-frame-pointers`, is still silently replaced by `RUSTFLAGS="-D warnings"`. The evidence build can then differ from the product build while still exiting 0.
  - REQ-0203 also allows testing the host crate "with an explicit host target", but the REQ-0209 host clippy, test, and coverage commands don't show the `--target <host-triple>` form for that option.
- **Recommendation:**
  - Either prohibit `target.<triple>.rustflags` entirely in firmware projects (all flags go in `build.rs` or the profile settings), or use `--config 'target.<triple>.rustflags=["-D","warnings"]'`. Config arrays merge rather than replace.
  - State the `--target <host-triple>` variants of the host-crate commands.

### LG-204: REQ-0010's verification text contradicts its own check table
- **Affected:** REQ-0010@r3
- **Severity:** minor
- **Issue:** The new table defines a check per file type: clippy, `cargo deny`, link, and TOML parse. The Verification Criteria still say "Each must pass `cargo check` (exit 0)", and the Done Criteria say "All non-fragment examples type-check". The table also has no row for WASM examples from REQ-0112, which need `cargo check --target <wasm-triple>`.
- **Recommendation:**
  - Change the Verification Criteria to "each passes the table check for its file type".
  - Change the Done Criteria to "All non-fragment examples pass their file-type check".
  - Add `--target <wasm-triple>` to the `.rs` row for WASM examples.

### LG-205: REQ-0112 evidence placement and workspace invocation
- **Affected:** REQ-0112@r3 vs REQ-0108@r3, REQ-0006@r3
- **Severity:** minor
- **Issue:**
  - REQ-0112 labels its evidence "L1+", but the REQ-0108 Done Criteria attach "the REQ-0112 items" only to L2.
  - REQ-0006 requires evidence items to live in the per-class Evidence Requirements subsections, and REQ-0112 does not say they must appear there.
  - In a workspace that mixes std and no_std crates, `cargo build --no-default-features --target thumbv7em-none-eabihf` without `-p <crate>` tries to build every member for the bare-metal target and fails.
- **Recommendation:**
  - Make the REQ-0108 L1 list include the REQ-0112 items when applicable, and require them to be listed in the Evidence Requirements L1 subsection.
  - Use `cargo build -p <no_std-crate> --no-default-features --target <triple>` and `cargo build -p <wasm-crate> --target <wasm-triple>`.

### LG-206: Editorial inconsistencies
- **Affected:** REQ-0206@r3, the Lineage Key, and the Assumptions table
- **Severity:** minor
- **Issue:**
  - REQ-0206 still lists stack guards for "RISC-V/Xtensa", although A-08 removed Xtensa from scope.
  - The Lineage Key defines `LG-xxx` as "resolution of a Logic Gatekeeper r1 finding", but r3 lineage also cites r2 findings (LG-1xx).
  - The Assumptions table lists A-07 and A-08 before A-04.
- **Recommendation:**
  - Change REQ-0206 to "RISC-V".
  - Change the Lineage Key to "Logic Gatekeeper finding (any review round)".
  - Reorder the assumptions.

---

## Summary

| ID | REQ(s)@r3 | Severity | One-line issue |
|---|---|---|---|
| LG-201 | 0011 (vs 0112, SD-08), 0212 | major | `rust-toolchain.toml` `targets` and `wasm32v1-none` trip the firmware indicator, so a standalone no_std library or a WASM crate is misrouted |
| LG-202 | 0011 vs 0203 | minor | A host-testable crate in a sibling directory or separate workspace (path dependency) routes to rust |
| LG-203 | 0209, 0203 | minor | `RUSTFLAGS` still drops non-link target rustflags; the explicit-host-target command variants are not stated |
| LG-204 | 0010 | minor | Verification and Done criteria still say `cargo check`/"type-check", contradicting the per-type table; no WASM row |
| LG-205 | 0112 vs 0108, 0006 | minor | Evidence is "L1+" in REQ-0112 but L2 in REQ-0108; workspace builds need `-p` |
| LG-206 | 0206, Lineage Key, Assumptions | minor | Stale Xtensa mention; LG-xxx key says r1 only; assumption ordering |

Totals: 0 blocking, 1 major, 5 minor. All r2 findings (LG-101 to LG-116) are resolved, and no r1 resolution regressed. The proposed L1 risk class remains appropriate.

**Gate 1 recommendation:** r3 is acceptable for Gate 1 once LG-201 is fixed (one line in REQ-0011 plus two routing cases) or explicitly dispositioned by the human. Carry LG-202 to LG-206 as known minor items; the Requirement Architect may fold them into the same edit at negligible cost.
