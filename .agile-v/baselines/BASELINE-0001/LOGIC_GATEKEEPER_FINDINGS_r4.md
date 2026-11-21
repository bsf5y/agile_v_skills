# Logic Gatekeeper Independent Findings
<!-- Target: .agile-v/REQUIREMENTS.md | Revision set: r4 (architect_revisions) | Cycle: C1 | Reviewer: logic-gatekeeper (independent context) | Date: 2026-11-21 -->
<!-- Prior: r1–r3 findings archived under .agile-v/archive/ -->

Reviewed: full r4 text and a line diff against `archive/REQUIREMENTS_C1_r3.md`. Changed REQs: REQ-0010, REQ-0011, REQ-0108, REQ-0112, REQ-0203, REQ-0206, REQ-0209, and REQ-0212, plus the Lineage Key and the Assumptions order. All other REQs differ only by revision tag. | Requirements edited: no | Baseline edited: no

## Verdict

**PASS WITH FINDINGS.** There are 0 blocking, 0 major, and 3 minor findings.

- All six r3 findings (LG-201 to LG-206) are addressed. Five are fully resolved. LG-203 is partially resolved, and its residual is tracked as minor LG-303.
- The `--config` array-merge claim in REQ-0209 is accurate, with two edge caveats (LG-301).
- The r4 edits introduce no contradictions between REQs. The only edits outside the resolution scope are revision tags.

**Gate 1 readiness:** r4 is **ready for Gate 1**. Carry LG-301 to LG-303 as known minor items. None affects routing determinism, the risk boundaries, independence, or traceability. The Build Agent can apply all three as clarifications during synthesis, or the Requirement Architect can fold them into r5 after Gate 1 under change control.

---

## Status of r3 findings (checked against the r4 text)

| r3 ID | Status | Verification note |
|---|---|---|
| LG-201 | Resolved | REQ-0011 now counts only `.cargo/config.toml` `[build] target` set to a bare-metal triple, and excludes `wasm32*`. A target listed only in `rust-toolchain.toml` `targets` is explicitly not an indicator. Two cases were added: a no_std lib with toolchain targets → rust, and `wasm32v1-none` → rust. REQ-0212 matches ("bare-metal `[build] target`"). I re-ran all 11 cases against the rule, and each routes as specified. |
| LG-202 | Resolved | Crates that a firmware crate depends on by `path` belong to the firmware project, including those in a sibling directory or separate workspace. The sibling-directory case was added. A residual scope question is in LG-302. |
| LG-203 | Partially resolved | `RUSTFLAGS` is replaced by `--config 'target.<triple>.rustflags=[...]'`, and both host-test variants (outside the target-configured directory vs `--target <host-triple>`) are stated. The host-crate clippy and coverage commands were not given the `--target <host-triple>` variant; see LG-303. |
| LG-204 | Resolved | REQ-0010's Verification Criteria and Done Criteria now reference the per-file-type table, and a WASM `.rs` row was added. |
| LG-205 | Resolved | REQ-0108 L1 carries the REQ-0112 evidence, and the Done Criteria move it from L2 to L1. REQ-0112 commands use `-p <crate>`. `wasm-pack test --node <crate-dir>` and `cargo test -p <crate> --target wasm32-wasip2` are valid forms. |
| LG-206 | Resolved | Xtensa is removed from REQ-0206. The Lineage Key covers all review rounds. A-04 to A-08 are in order. |

## Check: the `--config` array-merge claim (REQ-0209 L1)

The claim is accurate. Cargo merges configuration values from all sources, and `--config` on the command line is the highest-precedence source. Arrays are joined, with higher-precedence items placed later in the merged array; they are not replaced. So `--config 'target.<triple>.rustflags=["-D","warnings"]'` appends `-D warnings` to any `target.<triple>.rustflags` in `.cargo/config.toml`. Matching `target.'cfg(...)'.rustflags` entries are also joined with `target.<triple>` flags. The statement that `RUSTFLAGS` replaces them is also correct: the `RUSTFLAGS` environment variable takes precedence over, and excludes, all config-file rustflags. `--cap-lints` keeps `-D warnings` from failing on registry dependencies, so only workspace and path crates are held to it. That is the intended scope. The two edge caveats are in LG-301.

## r4 contradiction check

These pairs remain consistent:

- REQ-0011 ↔ REQ-0212 (indicator text aligned)
- REQ-0011 ↔ REQ-0112 (standalone no_std library with toolchain targets → rust)
- REQ-0011 ↔ REQ-0203 (outside-directory host crate stays embedded through the path-dependency rule)
- REQ-0108 ↔ REQ-0112 (L1 placement)
- REQ-0108 Done Criteria ↔ L-lists
- REQ-0209 ↔ REQ-0203 (link args in `build.rs` remain valid; the old "so RUSTFLAGS cannot override" rationale is now redundant but harmless)
- REQ-0010 ↔ REQ-0005 (label and fragment rules)
- A-07 / A-08 ↔ REQ-0109 #9 / REQ-0210 #12

No regressions were found in the r1 or r2 resolutions; the unchanged REQs differ from r3 only by revision tag.

---

## New findings (r4)

### LG-301: `--config` rustflags edge cases: dotted triples and `build.rustflags`
- **Affected:** REQ-0209@r4 (L1 build)
- **Severity:** minor
- **Issue:** There are two edge cases:
  - **Dotted triples:** some Cortex-M triples contain a dot (`thumbv8m.main-none-eabihf`, `thumbv8m.base-none-eabi`). Unquoted in a dotted TOML key, `target.thumbv8m.main-none-eabihf.rustflags` parses as a different key path, so the flags silently do not apply.
  - **`build.rustflags`:** Cargo uses `target.<triple>`/`target.<cfg>` rustflags in preference to `build.rustflags`; they are not merged. A project that keeps its flags in `[build] rustflags` would have them dropped once `--config` introduces `target.<triple>.rustflags`, and would still exit 0.
- **Recommendation:**
  - Write the key with the triple quoted: `--config 'target."<triple>".rustflags=["-D","warnings"]'`.
  - State that firmware projects keep non-link rustflags in `target.<triple>` or `target.'cfg(...)'`, never in `[build] rustflags`. Alternatively, when `build.rustflags` is used, use `--config 'build.rustflags=[...]'`.

### LG-302: Path-dependency inclusion: transitive and shared crates
- **Affected:** REQ-0011@r4
- **Severity:** minor
- **Issue:** The REQ does not say whether "depends on by `path`" is transitive, i.e. a path dependency of a path dependency. It also does not explicitly cover a crate that is path-depended on by both a firmware crate and a host application, such as a shared `protocol` crate. As written, such a crate routes to embedded unless a baselined REQ designates it a host product. That outcome is deterministic and acceptable, but it is implicit.
- **Recommendation:** Add "(transitively)". Add one sentence: a crate shared by firmware and host crates routes to embedded unless a REQ designates it a host product.

### LG-303: Host-crate clippy and coverage lack the `--target <host-triple>` variant
- **Affected:** REQ-0209@r4 (L1 clippy, L2 coverage), REQ-0203@r4
- **Severity:** minor (residual of LG-203)
- **Issue:** REQ-0209 gives both variants for the host-crate *test* command. The host-crate clippy command (`cargo clippy --all-targets -- -D warnings`, L1) and the coverage command (`cargo llvm-cov --fail-under-lines 80`, L2) have only the outside-directory form. Run inside the target-configured directory, they cross-compile and fail. REQ-0203 says "both host test command variants stated", which makes the omission visible.
- **Recommendation:** Add `-p <crate> --target <host-triple>` variants:
  - `cargo clippy -p <crate> --all-targets --target <host-triple> -- -D warnings`
  - `cargo llvm-cov -p <crate> --target <host-triple> --fail-under-lines 80`

---

## Summary

| ID | REQ(s)@r4 | Severity | One-line issue |
|---|---|---|---|
| LG-301 | 0209 | minor | `--config` key must quote dotted triples (`thumbv8m.main-…`); `build.rustflags` would be dropped |
| LG-302 | 0011 | minor | Path-dependency inclusion: transitivity and firmware/host shared crates not stated |
| LG-303 | 0209, 0203 | minor | Host-crate clippy and coverage lack the `--target <host-triple>` variant (residual of LG-203) |

Totals: 0 blocking, 0 major, 3 minor. Five r3 findings are fully resolved and LG-203 is partially resolved (residual LG-303). The REQ-0209 `--config` merge claim is verified accurate with caveats. No new contradictions and no regressions.

**Gate 1 statement:** r4 is ready for Gate 1, with LG-301 to LG-303 recorded as known minor items. At Gate 1 the stakeholder should also re-confirm SD-01 to SD-09 (LG-115) and ratify assumptions A-01 to A-08 and the proposed L1 risk class.
