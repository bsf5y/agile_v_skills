# Logic Gatekeeper Independent Findings
<!-- Target: .agile-v/REQUIREMENTS.md | Revision set: r1 (draft_persisted) | Cycle: C1 | Reviewer: logic-gatekeeper (independent context) | Date: 2026-11-21 -->

Reviewed: REQ-0001 to REQ-0009, REQ-0101 to REQ-0111, REQ-0201 to REQ-0212 (all @r1) | Skipped: none | Requirements edited: no | Baseline edited: no

## Verdict

**FAIL**: three blocking findings (LG-001 to LG-003). Do not proceed to Gate 1 until the Requirement Architect issues r2 that resolves the blocking and major findings, and the r2 revision is re-reviewed.

Halt records (Procedure 4):

```
REQ-0001 vs SD-01/A-01 (+ repo test suite) | any non-dot output location fails catalog/plugin tests unless forbidden files are edited | fix the output location | HALTED
REQ-0104 (forbid + module confinement) | self-contradictory: forbid(unsafe_code) cannot be overridden in a module | change the lint mechanism | HALTED
REQ-0103 vs REQ-0108 | the INVARIANT comment does not suppress the clippy lints that L1 evidence promotes to errors | define the suppression mechanism | HALTED
```

Reference material consulted (read-only): build-agent/SKILL.md, docs/DOMAIN_SKILL_TEMPLATE.md, domains/build-agent-embedded/SKILL.md, domains/build-agent-python/SKILL.md, docs/agile-v-runtime/04_RISK_CLASSIFICATION.md, .claude-plugin/plugin.json, tests/test_skill_catalog.py, tests/test_distribution_manifests.py, tests/test_skill_frontmatter.py.

---

## Findings

### LG-001: Output location unspecified; repo-root or `domains/` placement breaks the repository test suite
- **Affected:** REQ-0001@r1 (also SD-01, A-01)
- **Severity:** blocking
- **Issue:** REQ-0001 does not say where the two directories go. The repository tests discover every `SKILL.md` outside dot-directories. `tests/test_skill_catalog.py::test_catalog_completely_and_uniquely_covers_skills` requires every discovered SKILL.md to appear in `catalog/skills.json`. `tests/test_distribution_manifests.py::test_plugin_catalog_contains_exactly_released_discoverable_skills` globs `*/SKILL.md` and `domains/*/SKILL.md` and requires every non-draft skill to appear in `.claude-plugin/plugin.json`. Because A-01 omits `metadata.status`, placing the files at `<root>/build-agent-rust/` or `domains/build-agent-rust/` makes CI fail unless catalog/plugin.json are edited. SD-01 and the REQ-0001 constraint forbid those edits. The requirement cannot be satisfied as written.
- **Recommendation:** Name an explicit output path that the tests exclude, for example `.agile-v/staging/build-agent-rust/SKILL.md` and `.agile-v/staging/build-agent-embedded-rust/SKILL.md` (dot-directories are skipped by the catalog, frontmatter, and graduation tests and are not matched by the distribution globs). Another option is a path outside the repository. Add the verification criterion "existing repository test suite result is unchanged after synthesis."

### LG-002: `#![forbid(unsafe_code)]` cannot coexist with "unsafe confined to a dedicated module"
- **Affected:** REQ-0104@r1, REQ-0109@r1
- **Severity:** blocking
- **Issue:** `forbid` cannot be overridden by an inner `#[allow(unsafe_code)]`, so a crate with `#![forbid(unsafe_code)]` cannot contain an authorized `unsafe` module. As written, the policy contradicts itself and cannot be implemented. The REQ also gives no mechanical way to enforce the `// SAFETY:` comment rule.
- **Recommendation:** Restate the policy as follows. Crates with no authorized unsafe use `#![forbid(unsafe_code)]`. Crates with authorized unsafe use `#![deny(unsafe_code)]` at crate level plus `#[allow(unsafe_code)]` on the single authorized module, or move the unsafe code into a separate dedicated crate. Require `clippy::undocumented_unsafe_blocks` (and optionally `clippy::multiple_unsafe_ops_per_block`) to enforce SAFETY comments. Note that edition 2024 warns on `unsafe_op_in_unsafe_fn` by default and requires `unsafe extern` blocks and `#[unsafe(no_mangle)]`/`#[unsafe(export_name)]`/`#[unsafe(link_section)]`.

### LG-003: The INVARIANT comment does not suppress the clippy lints that L1 evidence makes fatal
- **Affected:** REQ-0103@r1 vs REQ-0108@r1 (and REQ-0109@r1)
- **Severity:** blocking
- **Issue:** REQ-0103 allows `unwrap`/`expect`/`panic!` when an `// INVARIANT:` comment justifies them, and recommends enabling `clippy::unwrap_used`, `expect_used`, and `panic`. REQ-0108 L1 requires `cargo clippy ... -- -D warnings` with 0 warnings. A comment does not silence a lint, so every justified use fails L1 evidence. Also, "indexing that can panic" has no enforcing lint in the list, and the REQ does not say how test code is exempted.
- **Recommendation:** Define the justification mechanism as the narrowest-scope `#[expect(clippy::expect_used, reason = "INVARIANT: ...")]` (lint reasons are stable since Rust 1.81; `#[allow(..., reason)]` is acceptable if MSRV is lower), placed next to the `// INVARIANT:` comment. Add `clippy::indexing_slicing`. Exempt tests through `clippy.toml` (`allow-unwrap-in-tests`, `allow-expect-in-tests`, and the panic/indexing test allowances where the pinned clippy version supports them) or `#[cfg(test)]`-scoped allows.

### LG-004: Miri and cargo-fuzz need nightly, which conflicts with the stable-only toolchain baseline
- **Affected:** REQ-0108@r1 vs A-03
- **Severity:** major
- **Issue:** A-03 fixes "Rust stable". `cargo miri test` runs only on nightly (`rustup +nightly component add miri`), and `cargo fuzz` (libFuzzer with sanitizers) also needs nightly. Miri also cannot execute FFI calls or most inline assembly, so "crates containing `unsafe`" cannot be fully exercised when the unsafe code is FFI. Neither item has a pass threshold.
- **Recommendation:** Amend A-03 or REQ-0108 to allow a pinned nightly (`cargo +nightly-YYYY-MM-DD miri test`) for analysis tools only, recorded in the manifest. Thresholds: "0 Miri UB reports; every test skipped under Miri carries `#[cfg_attr(miri, ignore)]` with a justification." Fuzzing: "≥ N CPU-minutes (or a defined iteration count) per target with 0 crashes or timeouts; the corpus is committed or archived."

### LG-005: Several evidence items lack an objective threshold or a runnable command (violates REQ-0006)
- **Affected:** REQ-0006@r1, REQ-0108@r1, REQ-0209@r1
- **Severity:** major
- **Issue:**
  - Coverage: "threshold for critical modules" has no number, and "critical module" is undefined. `cargo llvm-cov --fail-under-lines` applies to the whole report, not to individual modules.
  - "MSRV build check" has no command.
  - Kani is "optional" with no trigger condition and no threshold.
  - "Independent review" at L3 is undefined. Per the risk classification, L3 requires I2 independent verification and I3 human sign-off, and AI output is never independent assurance.
  - `unsafe` inventory: no threshold. `cargo geiger` is weakly maintained and often fails on current Cargo, workspaces, and no_std cross targets.
  - Embedded "stack usage estimate with margin" has no method and no margin value.
  - `cargo build --release` "0 warnings" is not enforced by the exit code.
  - On-target/HIL results have no threshold.
  - For workspaces, `cargo fmt --check` and `cargo test` should be `--all` / `--workspace`.
- **Recommendation:** Give each item a command and threshold:
  - Coverage: `cargo llvm-cov --workspace --fail-under-lines <N>`, plus a per-file threshold for modules tagged critical in the REQs (checked from `--json` output).
  - MSRV: `cargo +<rust-version> check --workspace --all-targets` (or `cargo msrv verify`).
  - Kani: required when a REQ marks a function for proof; threshold "all harnesses VERIFICATION:SUCCESSFUL".
  - Unsafe inventory: every unsafe block listed with file:line, an authorization reference, and a SAFETY comment, with 0 unlisted blocks (grep, or `cargo geiger` where it works).
  - Warnings: `RUSTFLAGS="-D warnings"` or `--config 'build.rustflags=["-D","warnings"]'`.
  - Stack: name a method (static analysis such as `cargo-call-stack`/`-Z emit-stack-sizes` on a pinned nightly, or runtime stack painting) and a default margin (for example ≥ 20 % unless a REQ specifies otherwise).
  - On-target/HIL: 100 % of designated tests pass.

### LG-006: Host unit tests in an embedded workspace have no workable command
- **Affected:** REQ-0209@r1, REQ-0203@r1
- **Severity:** major
- **Issue:** REQ-0203 sets the embedded target in `.cargo/config.toml`. In that workspace, a plain `cargo test` cross-compiles for `thumbv*`/`riscv32*` and fails, because there is no `test` crate for no_std targets and no host runner. REQ-0209 L1 requires "host unit tests pass" but gives no command, so the evidence cannot be produced as specified.
- **Recommendation:** Require the host-testable crate to live in a separate workspace or directory without the target override, or to be tested with an explicit host target (for example `cargo test -p <core-crate> --target x86_64-unknown-linux-gnu`). State the command and the "all pass" threshold.

### LG-007: The two skills apply inconsistent unsafe-authorization rules; BSP layer omitted
- **Affected:** REQ-0204@r1, REQ-0210@r1 vs REQ-0104@r1; REQ-0203@r1
- **Severity:** major
- **Issue:** The host skill requires a REQ or decision to authorize any `unsafe`. The embedded skill halts only on unsafe "outside PAC/HAL/startup", which implies unsafe inside those layers needs no authorization. The BSP layer (REQ-0203) is not in the permitted list. PACs and HALs are usually third-party crates (svd2rust output, embassy-*, *-hal), and the REQs do not say whether the Build Agent may write PAC or HAL code at all. The REQs also do not state the lint posture of the application crate.
- **Recommendation:** Align the skills: any project-authored `unsafe` requires REQ or decision authorization in both. List the layers where authorized unsafe may appear (project-owned PAC, HAL, BSP, startup). State that third-party PAC/HAL crates are dependencies, not artifacts. Require `#![deny(unsafe_code)]` (or `forbid` where macro expansion permits) in the application crate.

### LG-008: Routing has gaps, and the two skills' auto-triggers overlap with no precedence rule
- **Affected:** REQ-0101@r1, REQ-0111@r1, REQ-0201@r1, REQ-0212@r1
- **Severity:** major
- **Issue:** REQ-0101 sends all `no_std` work to embedded, but REQ-0201 covers only "no_std/no_main firmware for microcontrollers". The following cases have no clear owner:
  - Portable `no_std` library crates, such as embedded-hal driver crates or `no_std + alloc` libraries.
  - WebAssembly.
  - Embedded Linux running std.
  - Targets outside `thumbv*`/`riscv32*`, such as Xtensa ESP32 (`xtensa-*-none-elf`, which needs the esp toolchain) and AVR.

  Both skills auto-trigger on `**/*.rs` and `**/Cargo.toml`, and neither states which skill wins.
- **Recommendation:** Add a deterministic classification rule. For example: any crate with `#![no_std]` or a non-host target triple goes to embedded-rust; std, including embedded Linux, goes to host. State WASM explicitly as in or out of scope. For unsupported targets, either list them or halt. State precedence: embedded indicators override host triggers.

### LG-009: Host skill's L4 treatment and both skills' L0 content are undefined
- **Affected:** REQ-0006@r1, REQ-0108@r1, REQ-0209@r1
- **Severity:** major
- **Issue:** The domain template requires L0 to L4 coverage. The embedded skill explicitly halts at L3 and above (SD-05), but the host skill specifies L0 to L3 and says nothing about L4 (safety-critical or externally assured work). The Done Criteria in REQ-0108 and REQ-0209 check for an L0 item, but neither requirement body defines L0 content.
- **Recommendation:** Decide whether host L4 is a halt-and-escalate condition (mirroring REQ-0202) or gets an L4 section, and record that as a stakeholder decision. State "L0: base evidence applies; no Rust-specific additions" explicitly in both skills.

### LG-010: Lineage is misattributed: every REQ claims a direct stakeholder directive
- **Affected:** all REQs@r1 (notably REQ-0005, REQ-0006, REQ-0102 to REQ-0110, REQ-0203 to REQ-0211)
- **Severity:** major
- **Issue:** Every REQ records "Stakeholder Directive: Yes — Wes Cravens, 2026-11-21". The only recorded stakeholder statements are SD-01 to SD-05 and the assumptions A-01 to A-03. Most technical content (lint sets, thresholds, tool choices, halt lists) was derived by the architect from build-agent rules, the template, or ecosystem practice. Misattribution weakens the trace chain and hides which content the human has actually decided.
- **Recommendation:** Use typed lineage per REQ. Examples: `derives_from: SD-03`, `derives_from: build-agent Secure Coding rule 5`, `derives_from: DOMAIN_SKILL_TEMPLATE §5`. Otherwise record "Stakeholder Directive: No — architect-derived; ratified at Gate 1".

### LG-011: Nothing requires the code/config examples to be correct, and ecosystem versions are not pinned
- **Affected:** REQ-0005@r1, REQ-0102 to REQ-0108@r1, REQ-0203 to REQ-0208@r1; A-03
- **Severity:** major
- **Issue:** Every technical verification criterion is "reviewer confirms/finds". No REQ requires the Rust or TOML examples to compile or pass clippy. A-03 pins only `embedded-hal` 1.0. Several required crates have unstable or recently changed APIs:
  - Embassy crates are 0.x with frequent breaking changes.
  - `embedded-hal-async` 1.0.
  - `heapless` 0.8 vs 0.9.
  - `static_cell` 2.x.
  - `defmt` 1.0.
  - `secrecy` 0.10, which removed `Secret<T>`.
  - `embedded-hal-mock` needs its `eh1` feature for embedded-hal 1.0.
  - `sqlx`, `tokio`, and `probe-rs` CLI changes.

  Examples written without pinned versions will be wrong or quickly rot, and that cannot be checked.
- **Recommendation:** Add a REQ requiring each skill to include an "Ecosystem versions (as of YYYY-MM-DD)" table. Each fenced rust/toml artifact example must either type-check (`cargo check`, or `cargo check --target <triple>` for embedded) in a scratch crate against those versions, or be explicitly labeled illustrative/WRONG. Give red-team-verifier a verification path for this (extract and compile).

### LG-012: "Artifact example" is undefined; header syntax is missing for non-Rust/TOML artifacts
- **Affected:** REQ-0005@r1
- **Severity:** major
- **Issue:** Which examples "represent a synthesis artifact" is a judgment call, so the grep-based verification is circular. The REQ does not say whether WRONG/anti-pattern examples are exempt. `memory.x` is a linker script and is neither `rust` nor `toml`, so it has no specified header syntax. The 5 to 25 line bound departs from the template's 5 to 15 guideline, and a workspace `Cargo.toml` or Embassy `main` may exceed 25 lines.
- **Recommendation:** Define the rule as: every fenced block tagged `rust`, `toml`, or `ld`/`text` for linker scripts is an artifact example, unless its preceding label is `WRONG`. Linker scripts use `/* Implements: ... */`. Set the line bound explicitly and allow a documented exception, or split large examples.

### LG-013: REQ-0003 claims template conformance, but the template has 7 sections; the {Domain} heading is unspecified
- **Affected:** REQ-0003@r1, REQ-0002@r1
- **Severity:** minor
- **Issue:** The `sections_index` in DOMAIN_SKILL_TEMPLATE.md lists 7 sections, with no "Output Format". The released domain skills (python, embedded) use the 8 sections that REQ-0003 requires. The exact `{Domain} Architecture & Patterns` heading text is not fixed, so the ordered heading list cannot be determined in advance.
- **Recommendation:** Cite the released domain skills as the structural reference instead of the template. Fix the headings, for example "Rust Architecture & Patterns" and "Embedded Rust Architecture & Patterns".

### LG-014: Frontmatter values are unspecified, and `metadata` conflicts with the AgentSkills spec
- **Affected:** REQ-0002@r1
- **Severity:** minor
- **Issue:** The values for `author` (agile-v.org or the stakeholder) and `domain` are not given. The AgentSkills.io specification defines `metadata` as a string-to-string map. A nested `sections_index` list follows this repository's convention but may fail strict validators in the destination project.
- **Recommendation:** Specify the `author` and `domain` values. Either confirm that the destination project's validator accepts list-valued metadata, or record the deviation as a decision.

### LG-015: Inherited-rule list is incomplete; AI_RUN_MANIFEST template path is repo-only
- **Affected:** REQ-0004@r1, REQ-0007@r1, REQ-0109@r1, REQ-0210@r1
- **Severity:** minor
- **Issue:**
  - build-agent creates AI_RUN_MANIFEST from `templates/AI_RUN_MANIFEST.yaml`, a path that exists only in this repository (conflicts with A-02 and REQ-0007).
  - The inherited list omits secure coding rules, pre-execution validation, the 3-attempt post-verification feedback loop, Human Gates, hardware awareness, and qualification evidence duties. The released embedded skill lists several of these.
  - "Rules are referenced, not re-specified" sits in mild tension with "Inherited build-agent halts also listed".
- **Recommendation:** Add the missing inherited behaviors by name. Reference the manifest only by its runtime path (`.agile-v/aibom/<task_id>/AI_RUN_MANIFEST.yaml`). State that inherited halts may be listed by name without being re-specified.

### LG-016: Allowed skill-name list is incomplete; routing targets may not exist in the destination
- **Affected:** REQ-0007@r1, REQ-0212@r1, REQ-0201@r1
- **Severity:** minor
- **Issue:** The REQ-0007 list omits names that other REQs require: `build-agent-rust`, `build-agent-embedded-rust`, and `build-agent-embedded`. It also omits likely escalation targets such as `threat-modeler` and `agile-v-compliance`. After the files move, `build-agent-embedded` may not exist in the destination.
- **Recommendation:** Mark the list as illustrative, or make it complete. Phrase cross-skill routing conditionally ("route to `build-agent-embedded` if installed; otherwise halt").

### LG-017: Cargo.lock policy is non-deterministic; workspace resolver and MSRV interplay missing
- **Affected:** REQ-0102@r1
- **Severity:** minor
- **Issue:**
  - "Per-project policy for published libraries" leaves the agent to guess. Cargo's guidance since 2023 is to commit `Cargo.lock` by default for all packages.
  - Edition 2024's default resolver "3" applies only to packages, so virtual workspaces need an explicit `resolver = "3"`, otherwise they fall back to resolver 1 with a warning.
  - `[workspace.lints]` requires Rust ≥ 1.74, and edition 2024 requires ≥ 1.85. These floors constrain the declared MSRV.
- **Recommendation:** Commit `Cargo.lock` by default; any deviation requires a DECISION_LOG entry. Require `resolver = "3"` in virtual workspaces. State the toolchain floors that edition 2024 and the other features impose on MSRV.

### LG-018: Runtime-neutral libraries cannot implement timeouts themselves
- **Affected:** REQ-0105@r1 (SD-04)
- **Severity:** minor
- **Issue:** "All I/O and waits are bounded (timeouts)" in runtime-agnostic library code needs a timer, and timers are runtime-specific.
- **Recommendation:** Specify that libraries expose bounded APIs (caller-supplied deadlines, limits, bounded buffers) and that applications apply timeouts with `tokio::time::timeout`.

### LG-019: Ambiguous security terms
- **Affected:** REQ-0106@r1, REQ-0109@r1
- **Severity:** minor
- **Issue:** "Least-privilege file paths" and "unvetted source" are undefined. The REQs give no rule for when a RUSTSEC advisory may be ignored in `deny.toml`.
- **Recommendation:** Define the terms:
  - Least-privilege file paths: canonicalize paths and require root containment (reject traversal), with minimal file permissions.
  - Unvetted source: not on the `deny.toml` `[sources]` allow list.
  - Advisory ignores: each `[advisories] ignore` entry requires a DECISION_LOG reference.

### LG-020: RTIC selection criterion is technically imprecise
- **Affected:** REQ-0205@r1 (SD-03)
- **Severity:** minor
- **Issue:** Embassy supports priority-based preemption through multiple executors (`InterruptExecutor`), so "hard real-time priority-based preemption" does not distinguish RTIC. RTIC's real differentiator is compile-time Stack Resource Policy analysis, which gives deadlock-free, bounded-blocking resource sharing.
- **Recommendation:** Restate the criterion around RTIC's compile-time resource and priority analysis and its schedulability evidence. Note that Embassy can also use interrupt executors.

### LG-021: Some tool mandates are architecture-specific
- **Affected:** REQ-0206@r1, REQ-0207@r1
- **Severity:** minor
- **Issue:** `flip-link` supports only ARM Cortex-M (cortex-m-rt). HardFault handling is Cortex-M-specific. Mandating `defmt` excludes cases where a REQ specifies another logging transport.
- **Recommendation:** Give per-architecture alternatives, such as MPU or stack-guard regions, stack painting, or HAL-provided guards on RISC-V/ESP. Make `defmt` the default unless a REQ specifies otherwise. Phrase fault handling as "architecture fault handlers (e.g. Cortex-M HardFault)".

### LG-022: Some halt triggers are undefined or unreachable
- **Affected:** REQ-0210@r1, REQ-0209@r1
- **Severity:** minor
- **Issue:**
  - "Stack margin below the project threshold" is undefined when the project has no threshold.
  - The "timing deadline unmet" halt can never fire at L1, because timing evidence is collected only at L2.
  - "Target lacks HAL support" is vague.
- **Recommendation:** Add a default margin, or halt when no threshold is defined. Require timing evidence whenever a REQ states a deadline, regardless of risk class. Define HAL support as "no maintained HAL implementing embedded-hal 1.0 for the target; requires authorization to write one".

### LG-023: L3 escalation target is too narrow
- **Affected:** REQ-0202@r1
- **Severity:** minor
- **Issue:** L3 triggers include regulated, sensitive-data, and trust-boundary changes, not only safety. Common firmware features such as secure boot, OTA update, and key storage are trust-boundary work and therefore L3, yet REQ-0202 escalates only to `safety-engineer`.
- **Recommendation:** Escalate to the human plus the applicable skill (`safety-engineer`, `threat-modeler`, or compliance). Name common firmware L3 examples explicitly.

### LG-024: Context-size constraint cannot be verified with `wc -l`
- **Affected:** REQ-0009@r1
- **Severity:** minor
- **Issue:** "Fits ≤50 % of an agent context" depends on the model and is not checked by `wc -l`. The template target is under 20 % of a 200K window, with 600 to 800 lines.
- **Recommendation:** Replace it with a measurable proxy (for example ≤ N KB or ≤ N tokens by a named tokenizer), or drop the constraint.

### LG-025: Manifest variants and evidence-summary content are underspecified
- **Affected:** REQ-0008@r1
- **Severity:** minor
- **Issue:** REQ-0008 does not mention the multi-cycle 8-column manifest (`... | CYCLE | CR | notes`). It does not define what the evidence summary contains or require a link to the AI_RUN_MANIFEST.
- **Recommendation:** Reference the C2+ format and list the minimum evidence-summary fields, including evidence commands and results, risk class, and the AI_RUN_MANIFEST path.

### LG-026: Embedded auto-trigger overlaps host crates
- **Affected:** REQ-0212@r1
- **Severity:** minor
- **Issue:** The "`build.rs` with linker args" trigger also matches many host crates.
- **Recommendation:** Narrow it to linker args that reference `link.x`, `memory.x`, or `defmt.x`.

### LG-027: Embedded L2 evidence omits items that host L2 requires, without rationale
- **Affected:** REQ-0209@r1 vs REQ-0108@r1
- **Severity:** minor
- **Issue:** Embedded L2 omits coverage, SBOM, and the MSRV check, all of which host L2 requires. Coverage can be measured on the host-testable crate, and an SBOM applies to firmware too.
- **Recommendation:** Add coverage of the host-testable crate (`cargo llvm-cov`), an SBOM (`cargo cyclonedx`), and the MSRV check, or record why each is excluded.

---

## Summary

| ID | REQ(s)@r1 | Severity |
|---|---|---|
| LG-001 | REQ-0001 (SD-01, A-01) | blocking |
| LG-002 | REQ-0104, REQ-0109 | blocking |
| LG-003 | REQ-0103 vs REQ-0108 | blocking |
| LG-004 | REQ-0108 vs A-03 | major |
| LG-005 | REQ-0006, REQ-0108, REQ-0209 | major |
| LG-006 | REQ-0209, REQ-0203 | major |
| LG-007 | REQ-0204, REQ-0210 vs REQ-0104 | major |
| LG-008 | REQ-0101, REQ-0111, REQ-0201, REQ-0212 | major |
| LG-009 | REQ-0006, REQ-0108, REQ-0209 | major |
| LG-010 | all | major |
| LG-011 | REQ-0005, REQ-0102 to REQ-0108, REQ-0203 to REQ-0208, A-03 | major |
| LG-012 | REQ-0005 | major |
| LG-013 to LG-027 | various | minor |

Totals: 3 blocking, 9 major, 15 minor. Proposed risk class L1 for the deliverable is reasonable (not integrated into this repo, no production code path). That is conditional on LG-001 being resolved so that no repository CI or distribution artifacts are affected.
