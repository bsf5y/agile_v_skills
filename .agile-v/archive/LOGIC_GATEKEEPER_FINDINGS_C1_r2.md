# Logic Gatekeeper Independent Findings
<!-- Target: .agile-v/REQUIREMENTS.md | Revision set: r2 (architect_revisions) | Cycle: C1 | Reviewer: logic-gatekeeper (independent context) | Date: 2026-11-21 -->
<!-- Prior: r1 findings archived at .agile-v/archive/LOGIC_GATEKEEPER_FINDINGS_C1_r1.md -->

Reviewed: REQ-0001 to REQ-0011, REQ-0101 to REQ-0111, REQ-0201 to REQ-0212 (all @r2); SD-06 to SD-09; A-04 to A-06 | Skipped: none | Requirements edited: no | Baseline edited: no

## Verdict

**FAIL**: 3 blocking, 6 major, and 7 minor new findings.

r2 is a substantial improvement:
- 23 of the 27 r1 findings are fully resolved.
- 4 are partially resolved; each residual is tracked as a new LG-1xx finding.

The remaining blockers are contradictions between REQs (Procedure 4). All are narrow and quick to fix, but r2 is not yet fit for Gate 1.

Halt records:

```
REQ-0102 vs REQ-0103 | `reason =` in lint attributes needs Rust >=1.81, but REQ-0103 prescribes #[allow(..., reason)] for MSRV <1.81 | LG-101 | HALTED
REQ-0011 vs REQ-0108 | cargo-fuzz targets carry #![no_main], so the routing rule sends host fuzz targets to the firmware skill | LG-102 | HALTED
REQ-0011 vs REQ-0203/REQ-0208 (SD-02) | the host-testable crate of a firmware project routes to build-agent-rust, yet the embedded skill must govern it | LG-103 | HALTED
```

---

## Status of r1 findings

Each "resolved" status was checked against r2 text, not taken from the architect's resolution table. LG-001 was additionally checked against the repository tests.

| r1 ID | Status | Verification note |
|---|---|---|
| LG-001 | Resolved | SD-06 puts output under `.agile-v/staging/`. Four checks confirm this path is safe: (1) `test_skill_catalog`, `test_skill_frontmatter`, `test_skill_graduation_policy`, `test_independence_classes`, `test_evidence_admission`, and `test_compliance_language` skip dot-directories; (2) the `test_distribution_manifests` globs (`*/SKILL.md`, `domains/*/SKILL.md`, `skills/*/SKILL.md`) do not match a depth-3 path; (3) `test_behavioral_contracts` scans dot-directories but only acts on `metadata.status: draft`, which A-01 omits; (4) the REQ-0001 verification criterion "pytest result unchanged" is objective. |
| LG-002 | Resolved | REQ-0104 (b)/(c) separates forbid from deny+allow, and `undocumented_unsafe_blocks` enforces the SAFETY comment. |
| LG-003 | Partially resolved | The `#[expect(reason)]` mechanism, `indexing_slicing`, and the test exemption are correct. The MSRV < 1.81 fallback is technically wrong; see **LG-101**. |
| LG-004 | Resolved | SD-07 allows the pinned nightly. Miri, fuzz, and Kani have thresholds; the Kani success string `VERIFICATION:- SUCCESSFUL` is accurate. |
| LG-005 | Resolved | Every item has a command or method plus a threshold. Defects in two of the new commands are tracked separately (**LG-104**, **LG-105**). |
| LG-006 | Resolved | REQ-0203 places the host-testable crate outside the target-configured directory or uses an explicit host target, and requires the command to run without cross-compiling. This is consistent with Cargo's cwd-based config discovery. |
| LG-007 | Resolved | REQ-0204 matches REQ-0104(a), adds BSP, makes the app crate deny unsafe, and separates dependencies from artifacts. A residual clarification is in **LG-110**. |
| LG-008 | Partially resolved | SD-08 and REQ-0011 add deterministic routing, and the five cases route correctly. Two new contradictions follow: **LG-102** and **LG-103**. |
| LG-009 | Resolved | A-04, REQ-0101, REQ-0108 L4, REQ-0006 L0 statement. |
| LG-010 | Resolved | Typed `derives_from` lineage is in place, and `ECO` is marked as not a stakeholder directive. The provenance of SD-06 to SD-09 needs confirmation; see **LG-115**. |
| LG-011 | Partially resolved | REQ-0010 adds a dated version table and a compile check. The check cannot apply to non-Rust artifact examples; see **LG-106**. |
| LG-012 | Resolved | Definition, `ld` header, ≤30 lines. A residual label-adjacency ambiguity is in **LG-113**. |
| LG-013 | Resolved | Exact headings, and REQ-0003 now references the released skills. |
| LG-014 | Resolved | REQ-0002 values table; A-05, A-06. |
| LG-015 | Resolved | 13 named items; runtime manifest path. |
| LG-016 | Resolved | REQ-0007 conditional phrasing. A residual inconsistency in the REQ-0011 text is in **LG-114**. |
| LG-017 | Partially resolved | The lockfile default, resolver 3, and the 1.74/1.81/1.85 floors are present. The resolver-3 floor (1.84) and a floor for the lint tables are missing; see **LG-108**. |
| LG-018 | Resolved | REQ-0105. |
| LG-019 | Resolved | REQ-0106 definitions and the advisory-ignore rule. |
| LG-020 | Resolved | REQ-0205 criterion is accurate (SRP vs `InterruptExecutor`). |
| LG-021 | Resolved | REQ-0206 per-architecture protection; REQ-0207 defmt default. |
| LG-022 | Resolved | Default 20 % margin, timing at any risk class, HAL definition. |
| LG-023 | Resolved | REQ-0202 trust-boundary examples and wider escalation. |
| LG-024 | Resolved | REQ-0009 byte cap (`wc -c`). |
| LG-025 | Resolved | REQ-0008 C2+ format, evidence-summary fields, AI_RUN_MANIFEST path. |
| LG-026 | Resolved | REQ-0212 narrowed the build.rs trigger. |
| LG-027 | Resolved | REQ-0209 adds coverage, SBOM, MSRV. |

---

## New findings (r2)

### LG-101: `reason =` fallback for MSRV < 1.81 does not compile
- **Affected:** REQ-0103@r2 vs REQ-0102@r2
- **Severity:** blocking (contradiction between REQs)
- **Issue:** REQ-0103 prescribes `#[allow(..., reason = ...)]` when MSRV < 1.81. The `reason = "..."` field in lint attributes was stabilized together with `#[expect]` in Rust 1.81 (lint_reasons, RFC 2383), so on older compilers this is a compile error. REQ-0102 correctly states that "`#[expect]`/lint reasons need ≥1.81", which contradicts REQ-0103.
- **Recommendation:** For MSRV < 1.81, use `#[allow(clippy::<lint>)]` immediately followed by a `// INVARIANT: ...` comment. Alternatively, set a skill-wide MSRV floor of ≥ 1.81 (see LG-108), which removes the fallback entirely.

### LG-102: The routing rule sends cargo-fuzz targets (and other host `no_main` programs) to the firmware skill
- **Affected:** REQ-0011@r2 vs REQ-0108@r2 (L3 fuzzing), REQ-0212@r2
- **Severity:** blocking (contradiction between REQs)
- **Issue:** REQ-0011 classifies a crate as firmware if *any* indicator holds, and the first indicator is "contains `#![no_main]`". `cargo fuzz` targets (`fuzz/fuzz_targets/*.rs`, using `libfuzzer_sys::fuzz_target!`) always contain `#![no_main]`, so the host skill's own L3 fuzz harnesses would route to `build-agent-embedded-rust`. `x86_64-unknown-uefi` programs and some WASI programs also use `#![no_main]` without being MCU firmware. The REQ-0212 auto-trigger "sources containing `#![no_main]`" has the same flaw.
- **Recommendation:** Do not treat `#![no_main]` alone as an indicator. Define firmware as a bare-metal target (`*-none*`, `avr-*`), **or** the presence of `memory.x`/`link.x`, **or** `#![no_main]` together with one of those. Explicitly exclude the `fuzz/` crates of cargo-fuzz. Re-run the REQ-0011 case table with a "host crate with cargo-fuzz targets → rust" case added.

### LG-103: Ownership of a firmware project's host-testable crate is contradictory
- **Affected:** REQ-0011@r2 vs REQ-0203@r2, REQ-0208@r2, REQ-0209@r2 (L1 host tests, L2 host coverage); SD-02
- **Severity:** blocking (contradiction between REQs)
- **Issue:** REQ-0203 requires the hardware-independent crate to sit outside the target-configured directory and to have no `no_main`, `memory.x`, or bare-metal target. Under the REQ-0011 per-crate routing it is therefore not firmware and routes to `build-agent-rust`. But REQ-0203, REQ-0208, and REQ-0209 make the embedded skill responsible for that crate's structure, tests, and evidence. SD-02 says the embedded skill must not depend on `build-agent-rust`. A single firmware task would split across two skills.
- **Recommendation:** Extend REQ-0011: crates that belong to a firmware project (same workspace or repository, and depended on by a firmware crate) route to `build-agent-embedded-rust`, which supplies host-crate guidance itself. Add that case to the routing case table, for example "host-testable core crate of a Cortex-M project → embedded".

### LG-104: Embedded `clippy --all-targets` commonly fails on no_std/no_main crates
- **Affected:** REQ-0209@r2 (L1), REQ-0203@r2
- **Severity:** major
- **Issue:** `cargo clippy --target <triple> --all-targets` also compiles the test and bench targets of `no_std`/`no_main` bins and libs for the bare-metal target. Those targets need the `test` crate, which is not available for `*-none-*`, so the command fails ("can't find crate for `test`") unless the manifests set `test = false` / `bench = false` (as knurling's app-template does) or the on-target tests use `harness = false`. As written, the L1 command can fail on a correctly built project.
- **Recommendation:** Do one of the following. Option 1: require REQ-0203's structure to set `test = false, bench = false` on firmware `[[bin]]`/`[lib]` targets and `harness = false` on `defmt-test`/`embedded-test` targets. Option 2: change the command to `cargo clippy --target <triple> --bins --lib -- -D warnings` (plus the on-target test targets if they are configured with `harness = false`).

### LG-105: `RUSTFLAGS="-D warnings"` silently replaces the firmware's linker flags
- **Affected:** REQ-0209@r2 (L1 build), REQ-0203@r2 (linker configuration)
- **Severity:** major
- **Issue:** Cargo's rustflags sources are mutually exclusive. The `RUSTFLAGS` environment variable *replaces* `target.<triple>.rustflags` from `.cargo/config.toml`. That is where embedded templates normally put `-C link-arg=-Tlink.x` / `-Tdefmt.x` (and `flip-link` relies on it). The result is a link failure or a binary built without the intended linker script. The alternative `--config 'build.rustflags=[...]'` is ignored whenever `target.<triple>.rustflags` exists. Either form can invalidate the evidence.
- **Recommendation:** Specify `--config 'target.<triple>.rustflags=["-D","warnings"]'`, since array values are merged across config sources. Alternatively, require that link arguments live in `build.rs` (`cargo:rustc-link-arg=...`) so `RUSTFLAGS` is safe. Keep the "exit 0" threshold.

### LG-106: REQ-0010's compile check cannot apply to non-Rust artifact examples
- **Affected:** REQ-0010@r2, REQ-0005@r2, REQ-0102@r2, REQ-0103@r2, REQ-0106@r2, REQ-0203@r2
- **Severity:** major
- **Issue:** REQ-0005 makes every `toml` and `ld` block an artifact example, and REQ-0010 requires *every* artifact example to pass `cargo check`. `deny.toml`, `clippy.toml`, `rust-toolchain.toml`, `Embed.toml`, `.cargo/config.toml`, and `memory.x` cannot be checked by `cargo check` on their own. Labelling them all `Fragment` would likely exceed the 25 % cap, given that roughly 6 to 8 config examples are mandated across REQ-0102, REQ-0103, REQ-0106, and REQ-0203. The criterion is either unsatisfiable or forces unwanted fragment labelling.
- **Recommendation:** Define a check per artifact type:
  - `rust`: `cargo check` (or `cargo check --target <triple>`).
  - `Cargo.toml` and `.cargo/config.toml`: placed in the scratch crate and used by that check.
  - `deny.toml`: `cargo deny check` parses it and exits 0 on the scratch crate.
  - `clippy.toml`: used by a `cargo clippy` run.
  - `memory.x`: linked in the embedded scratch build.
  - `rust-toolchain.toml` and `Embed.toml`: parse-only (TOML parse plus known keys).
  
  Compute the 25 % fragment cap over `rust` examples only.

### LG-107: The host skill's expanded scope (no_std libraries, WASM) has no guidance or evidence REQs
- **Affected:** REQ-0101@r2, REQ-0102 to REQ-0110@r2, REQ-0108@r2 (SD-08)
- **Severity:** major
- **Issue:** SD-08 and REQ-0101 move `no_std`/`no_std + alloc` libraries and WASM into `build-agent-rust`, but no REQ covers them:
  - **no_std libraries:** the `std` feature-gating pattern (`#![cfg_attr(not(feature = "std"), no_std)]`, `extern crate alloc`) is not covered, and nothing requires a no_std build check. Running `cargo test` on the host does not prove no_std compatibility; that needs, for example, `cargo build --target thumbv7em-none-eabihf --no-default-features`.
  - **WASM:** there is no target or test-runner guidance (`wasm32-unknown-unknown` / `wasm32-wasip1`, `wasm-bindgen-test`). L1's `cargo test --workspace` may not exercise wasm-specific code.
  
  The evidence for these in-scope project types is undefined.
- **Recommendation:** Add a REQ, or extend REQ-0102 and REQ-0108, with: (a) the no_std library pattern plus an L1 evidence item `cargo build --no-default-features --target <no_std triple>` (exit 0) for crates that declare no_std support; (b) a WASM subsection naming the target triple and test runner, with its evidence command. If that is too much, narrow SD-08 and REQ-0101 so WASM is out of scope (halt and ask).

### LG-108: The MSRV floor table is incomplete for mandated features; single-package projects are not covered
- **Affected:** REQ-0102@r2, REQ-0103@r2, A-03
- **Severity:** major
- **Issue:**
  - `resolver = "3"`, which REQ-0102 mandates for virtual workspaces, requires Cargo ≥ 1.84. An MSRV check with `cargo +<msrv>` on an older toolchain fails to parse the manifest. That floor is not listed.
  - REQ-0103 mandates `[workspace.lints]` (≥ 1.74) but gives no fallback for MSRV < 1.74, which A-03 permits via edition 2021.
  - Single-package projects (no workspace) need `[lints]`, not `[workspace.lints]`.
  
  Taken together, the REQs allow MSRV values that the mandated configuration cannot support.
- **Recommendation:** Pick one of two options:
  - Declare a skill-wide minimum MSRV, for example ≥ 1.84 (or ≥ 1.85, aligned with edition 2024). This also resolves LG-101.
  - List every floor (1.74 lint tables, 1.81 lint reasons/`#[expect]`, 1.84 resolver 3, 1.85 edition 2024), each with its fallback.
  
  In either case, state `[lints]` for single-package crates.

### LG-109: The L3 independence wording understates the required independence class
- **Affected:** REQ-0108@r2 (L3)
- **Severity:** major
- **Issue:** "Independent verification by a separate agent context" can be met by a fresh context. Under the Agile V risk model, a fresh context alone is at most I1. L3 verification requires I2: a separately instantiated, role-separated verifier with protected, independent inputs (Red Team Protocol). Human sign-off at L3 is I3, and it belongs in the Gate 2 / approval record, not only in `VERIFICATION_SUMMARY.md`, which the verifier owns. As written, the item weakens a base rule, which REQ-0004 forbids.
- **Recommendation:** Reword to: "independent verification by `red-team-verifier` (if installed) or another role-separated verifier with independent inputs (I2), results in `.agile-v/VERIFICATION_SUMMARY.md`; explicit human sign-off (I3) recorded in the Gate approval record." Keep the threshold as "both records present."

### LG-110: "Startup code" location is ambiguous for user-written `unsafe` fault handlers
- **Affected:** REQ-0204@r2, REQ-0207@r2
- **Severity:** minor
- **Issue:** REQ-0207 requires architecture fault handlers. In `cortex-m-rt` 0.7, the user writes the `#[exception] unsafe fn HardFault(ef: &ExceptionFrame) -> !` handler as an `unsafe fn` in the application binary. REQ-0204 requires `#![deny(unsafe_code)]` on the application crate and permits `unsafe` only in "project-owned PAC/HAL/BSP/startup code". It does not say whether a startup module inside the application crate may carry `#[allow(unsafe_code)]`.
- **Recommendation:** State that a single authorized `startup`/`fault` module inside the application crate may carry `#[allow(unsafe_code)]`, under the REQ-0104(c) pattern. Otherwise require fault handlers to live in the BSP crate.

### LG-111: The routing rule misclassifies ESP-IDF std firmware and bare-metal non-MCU targets
- **Affected:** REQ-0011@r2, REQ-0201@r2
- **Severity:** minor
- **Issue:**
  - ESP32 projects on `esp-idf` (`xtensa-esp32-espidf`, `riscv32imc-esp-espidf`) are MCU firmware that use `std`. They match no firmware indicator, so they route to `build-agent-rust`, whose guidance (tokio, host evidence) does not fit.
  - `aarch64-unknown-none` OS kernels and `*-uefi` targets match or nearly match the firmware indicators, even though REQ-0201 requires an MCU with an embedded-hal 1.0 HAL.
- **Recommendation:** Add explicit cases. Suggested routing: esp-idf std goes to embedded (or halt and ask). Bare-metal targets without an embedded-hal MCU HAL halt via REQ-0210 #9. Record the choice in SD-08 or REQ-0011.

### LG-112: Xtensa toolchain and runner conflict with the toolchain baseline and the probe-rs mandate
- **Affected:** REQ-0201@r2, REQ-0203@r2, REQ-0209@r2 (MSRV check), A-03
- **Severity:** minor
- **Issue:**
  - Xtensa needs the `esp` toolchain channel (installed with espup), which is neither stable nor the SD-07 nightly. A-03 permits only stable plus the listed nightly exceptions.
  - `cargo +<rust-version>` MSRV checks cannot run for Xtensa.
  - REQ-0203 mandates a `probe-rs` runner, but ESP projects conventionally use `espflash` (probe-rs Xtensa support is partial).
- **Recommendation:** Add an A-03 exception for vendor toolchains recorded in the manifest. Mark the MSRV check "N/A for vendor-toolchain targets, with the reason recorded". Allow "`probe-rs` (or `espflash` for ESP targets)".

### LG-113: The label-adjacency rule is ambiguous
- **Affected:** REQ-0005@r2, REQ-0010@r2
- **Severity:** minor
- **Issue:** "The line directly before" the fence breaks under normal Markdown formatting, where a blank line usually separates the label from the fence. The REQs also do not say what happens when a block is both `WRONG` and `Fragment`, or whether the 30-line limit counts the header line.
- **Recommendation:** Use "the nearest preceding non-blank line". State that `WRONG` blocks are excluded from both the header and the compile requirements. State whether the limit includes the header.

### LG-114: The shared routing text references `build-agent-rust` unconditionally
- **Affected:** REQ-0011@r2 vs REQ-0007@r2
- **Severity:** minor
- **Issue:** REQ-0011 requires identical routing text in both files and routes non-firmware work to `build-agent-rust` unconditionally. Inside `build-agent-embedded-rust`, `build-agent-rust` is a non-core skill that may not be installed, and REQ-0007 requires such references to be conditional. The same applies in reverse.
- **Recommendation:** Phrase both routes conditionally in the shared text, for example "route to `build-agent-rust` / `build-agent-embedded-rust` if installed; otherwise halt and ask."

### LG-115: Provenance of SD-06 to SD-09 is not evidenced
- **Affected:** SD-06 to SD-09; REQ-0001, REQ-0010, REQ-0011, REQ-0101, REQ-0108@r2 (lineage)
- **Severity:** minor
- **Issue:** SD-06 to SD-09 are recorded as stakeholder directives dated the same day as the findings, but nothing records that the human made them; they may be architect proposals. If they are, the typed lineage repeats the r1 LG-010 misattribution.
- **Recommendation:** At Gate 1, have the stakeholder confirm SD-06 to SD-09 explicitly, for example in the APPROVAL record or the DECISION_LOG. Otherwise reclassify them as A-xx assumptions.

### LG-116: The MSRV check with `--all-targets` produces false failures
- **Affected:** REQ-0108@r2 (L2), REQ-0209@r2 (L2)
- **Severity:** minor
- **Issue:** `cargo +<rust-version> check --workspace --all-targets` also compiles tests and benches, whose dev-dependencies are not bound by the package's MSRV. Projects that honour their MSRV can still fail this check. REQ-0209's embedded MSRV item also does not state its command.
- **Recommendation:** Use `cargo +<rust-version> check --workspace --lib --bins` (exit 0), and add `--target <triple>` in the embedded skill.

---

## Summary

| ID | REQ(s)@r2 | Severity | One-line issue |
|---|---|---|---|
| LG-101 | 0103 vs 0102 | blocking | `#[allow(..., reason)]` fallback for MSRV < 1.81 does not compile (`reason` also needs 1.81) |
| LG-102 | 0011 vs 0108, 0212 | blocking | `#![no_main]` indicator routes cargo-fuzz targets (and UEFI/WASI) to the firmware skill |
| LG-103 | 0011 vs 0203/0208/0209, SD-02 | blocking | A firmware project's host-testable crate routes to `build-agent-rust`, yet the embedded skill governs it |
| LG-104 | 0209, 0203 | major | `clippy --target <bare-metal> --all-targets` fails without `test = false` / `harness = false` |
| LG-105 | 0209, 0203 | major | `RUSTFLAGS="-D warnings"` replaces the `.cargo/config.toml` link args; `build.rustflags` is ignored when target rustflags exist |
| LG-106 | 0010, 0005 | major | Compile check cannot apply to toml/ld config examples; the 25 % fragment cap becomes unachievable |
| LG-107 | 0101, 0102 to 0110 | major | no_std libraries and WASM are in scope but have no guidance or evidence (no no_std build check, no wasm runner) |
| LG-108 | 0102, 0103, A-03 | major | MSRV floors are missing resolver 3 (1.84) and the lint-table fallback; `[lints]` is not stated for single packages |
| LG-109 | 0108 L3 | major | "Separate agent context" is only I1; L3 needs an I2 role-separated verifier plus I3 human sign-off in the gate record |
| LG-110 | 0204, 0207 | minor | Where the user-written `unsafe fn HardFault` lives under app-crate `deny(unsafe_code)` is unclear |
| LG-111 | 0011, 0201 | minor | ESP-IDF std firmware routes to the host skill; aarch64-none and UEFI are unaddressed |
| LG-112 | 0201, 0203, 0209, A-03 | minor | Xtensa `esp` toolchain, espflash runner, and MSRV check conflict with the baseline |
| LG-113 | 0005, 0010 | minor | "Line directly before" label rule is ambiguous; WRONG/Fragment interplay undefined |
| LG-114 | 0011 vs 0007 | minor | Identical routing text references a non-core skill unconditionally |
| LG-115 | SD-06 to SD-09 | minor | Human provenance of the new stakeholder decisions is not evidenced |
| LG-116 | 0108, 0209 | minor | MSRV check with `--all-targets` gives false failures from dev-deps; embedded command not stated |

r1 status: 23 resolved; 4 partially resolved (LG-003, LG-008, LG-011, LG-017), with residuals tracked as LG-101, LG-102, LG-103, LG-106, and LG-108. LG-010 is resolved; LG-115 is a provenance follow-up. LG-005 and LG-007 are resolved; their new-text defects are LG-104, LG-105, and LG-110.

The proposed L1 risk class remains appropriate.
