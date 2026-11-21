# Requirements (Blueprint)
<!-- Project: build-agent-rust + build-agent-embedded-rust domain skills | Cycle: C1 | Revision set: r1 | Created: 2026-11-21 | Human Gate 1: PENDING -->

## Context

| Field | Value |
|---|---|
| Deliverable | Two standalone Agent Skills contracts: `build-agent-rust/SKILL.md` and `build-agent-embedded-rust/SKILL.md` |
| Destination | Authored here, then moved by the stakeholder to another project. No integration into this repository. |
| Proposed risk class | `L1` (routine documentation/skill contract; no production code path). Confirm at Gate 1. |
| Lifecycle | All REQs `draft_persisted`; awaiting Logic Gatekeeper independent findings. Not synthesis input until `baselined`. |

### Stakeholder Decisions (2026-11-21, Wes Cravens)

| ID | Decision |
|---|---|
| SD-01 | Skills only; no plugin.json, catalog, profile, routing-guide, README, or test integration. |
| SD-02 | `build-agent-embedded-rust` is a **sibling**: extends `build-agent` directly; no dependency on `build-agent-embedded` (C/C++) or `build-agent-rust`. |
| SD-03 | Embedded concurrency: **Embassy primary**, RTIC documented as the alternative. |
| SD-04 | Host async: **runtime-neutral library code; tokio in application examples**. |
| SD-05 | Embedded scope: **up to L2 only**; L3/L4 and safety-critical work halts and escalates. |

### Assumptions requiring Gate 1 confirmation

| ID | Assumption |
|---|---|
| A-01 | Frontmatter omits `metadata.status` (matches existing released domain skills); destination project decides draft/released status. |
| A-02 | Because files will leave this repo, they must not contain relative links to files that exist only in this repo (e.g. `docs/agile-v-runtime/*.md`); Agile V concepts are referenced by skill name. |
| A-03 | Toolchain baseline: Rust stable, edition 2024 default (2021 allowed when MSRV requires), `embedded-hal` 1.0. |

### AI Influence

```yaml
ai_influence_expected: "substantial"   # Claude drafts both SKILL.md files
ai_bom_required: true
allowed_ai_components:
  models: ["claude-opus-5-5"]
  tools: ["Claude Code"]
  rag_sources: ["this repository's existing domain skills and DOMAIN_SKILL_TEMPLATE.md (read-only reference)"]
```

---

## Common Requirements (both skills)

## REQ-0001
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-01)
- **Requirement:** Deliver exactly two files, `build-agent-rust/SKILL.md` and `build-agent-embedded-rust/SKILL.md`, each self-contained in its own directory with no supporting files required.
- **Constraint:** No edits to any other file in this repository as part of synthesis.
- **Verification Criteria:** `git status` after synthesis shows only the two new SKILL.md files (plus `.agile-v/` evidence records).
- **Done Criteria:** [ ] Both files exist · [ ] Directory name equals frontmatter `name` · [ ] No other tracked files modified

## REQ-0002
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Each file begins with valid YAML frontmatter containing: `name`, `description` (states domain, "Extends build-agent", and when to use), `license: CC-BY-SA-4.0`, and `metadata` with `version: "1.0"`, `standard: "Agile V"`, `domain`, `extends: "build-agent"`, `author`, `sections_index`.
- **Constraint:** `name` is lowercase kebab-case and ≤64 chars; `description` ≤1024 chars (AgentSkills.io limits). `sections_index` entries equal the file's `##` section headings, in order. No `metadata.status` (A-01).
- **Verification Criteria:** Parse frontmatter with a YAML parser; check field presence, limits, and `sections_index` == ordered list of `##` headings.
- **Done Criteria:** [ ] YAML parses · [ ] All fields present · [ ] Limits respected · [ ] sections_index matches headings

## REQ-0003
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Each file contains these `##` sections in this order: Inherited Rules, SCOPE-V Participation, {Domain} Architecture & Patterns, Evidence Requirements, Halt Conditions, Context Engineering, Output Format, When to Use.
- **Constraint:** Structure follows the repository's domain skill template so the skills are interchangeable with existing domain skills.
- **Verification Criteria:** Extract `##` headings and compare to the required ordered list.
- **Done Criteria:** [ ] All eight sections present · [ ] Order correct

## REQ-0004
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** The Inherited Rules section states that all `build-agent` rules apply and enumerates: synthesis only from approved AND baselined requirements; typed lineage `artifact -> implements -> REQ-XXXX@revision` with baseline reference; Build Manifest on every delivery; no self-verification (Red Team Protocol); halt on ambiguity; decision logging; multi-cycle artifact versioning (`ART-XXXX.N`); AI_RUN_MANIFEST for AI-influenced work.
- **Constraint:** Rules are referenced, not re-specified; the skill adds only domain-specific conventions.
- **Verification Criteria:** Reviewer confirms each listed rule appears and no base rule is contradicted or weakened.
- **Done Criteria:** [ ] All eight inherited behaviors listed · [ ] No contradictions with build-agent

## REQ-0005
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Every Rust code example that represents a synthesis artifact begins with the traceability header `// Implements: REQ-XXXX@revision; baseline: BASELINE-XXXX; <description>`. TOML/config examples use `# Implements: ...`.
- **Constraint:** Do not use the legacy `Parent: REQ-XXXX` form. Code examples are 5–25 lines each.
- **Verification Criteria:** grep every fenced `rust`/`toml` artifact example for the header; check example length.
- **Done Criteria:** [ ] 100% of artifact examples carry the header · [ ] No `Parent:` headers · [ ] Example length within bounds

## REQ-0006
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Evidence Requirements are organized by Agile V risk class (L0, L1, L2, and L3 where in scope), each stating "base evidence applies" plus Rust-specific additions expressed as concrete, runnable commands or named reports with pass thresholds.
- **Constraint:** Thresholds must be objective (e.g. "0 clippy warnings with `-D warnings`", "0 unignored RUSTSEC advisories").
- **Verification Criteria:** Reviewer confirms each risk class lists commands/reports with an objective pass criterion.
- **Done Criteria:** [ ] Each in-scope class present · [ ] Every addition has a command or report · [ ] Every addition has a pass threshold

## REQ-0007
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-01, A-02)
- **Requirement:** Files are portable: no relative Markdown links or path references to files that exist only in this repository; Agile V concepts are referenced by skill name (`build-agent`, `agile-v-core`, `agile-v-behavioral`, `logic-gatekeeper`, `test-designer`, `red-team-verifier`, `safety-engineer`).
- **Constraint:** Project-runtime paths that Agile V creates in any project (`.agile-v/REQUIREMENTS.md`, `.agile-v/VERIFICATION_SUMMARY.md`, `.agile-v/aibom/...`, `DECISION_LOG.md`) are allowed.
- **Verification Criteria:** grep for `](` links and `docs/`, `domains/`, `templates/` paths; each hit must be external URL or allowed runtime path.
- **Done Criteria:** [ ] No repo-relative references · [ ] Skills referenced by name

## REQ-0008
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** The Output Format section provides an example Build Manifest using `ART-XXXX | REQ-XXXX@revision | baseline-id | implements | path | notes`, with header fields for cycle, task, risk class, toolchain (rustc version, edition, MSRV), and target (target triple; for embedded also MCU, RAM, flash), followed by an evidence summary.
- **Constraint:** Manifest lists paths only, not file contents.
- **Verification Criteria:** Reviewer confirms column format and required header fields.
- **Done Criteria:** [ ] Six-column format · [ ] Toolchain and target fields · [ ] Evidence summary

## REQ-0009
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Each file is 400–900 lines.
- **Constraint:** Fits ≤50% of an agent context when loaded alongside `build-agent` and `agile-v-core`.
- **Verification Criteria:** `wc -l`.
- **Done Criteria:** [ ] Line count within range

---

## build-agent-rust (host / std Rust)

## REQ-0101
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Scope covers `std` Rust: libraries, CLIs, services/APIs, and workspaces. The skill explicitly routes `no_std`/microcontroller work to `build-agent-embedded-rust`.
- **Constraint:** —
- **Verification Criteria:** When to Use section lists in-scope project types and the routing exclusion.
- **Done Criteria:** [ ] Scope listed · [ ] no_std exclusion stated

## REQ-0102
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (A-03)
- **Requirement:** Project-structure guidance covers: Cargo workspace layout (`crates/`, shared `[workspace.dependencies]`, `[workspace.lints]`), declared `rust-version` (MSRV), edition, feature-flag discipline (additive features only), and `Cargo.lock` policy (committed for binaries and workspaces; per-project policy for published libraries).
- **Constraint:** —
- **Verification Criteria:** Reviewer finds each listed topic with an example `Cargo.toml`.
- **Done Criteria:** [ ] Workspace example · [ ] MSRV · [ ] Features · [ ] Lockfile policy

## REQ-0103
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Error-handling rules: fallible operations return `Result`; libraries define typed errors (e.g. `thiserror`); applications may aggregate (e.g. `anyhow`) with context; `unwrap`/`expect`/`panic!`/indexing that can panic are prohibited in non-test code unless an `// INVARIANT:` comment justifies infallibility.
- **Constraint:** Implements build-agent secure-coding rule 2 (explicit error handling).
- **Verification Criteria:** Rules present; recommended clippy lints (`unwrap_used`, `expect_used`, `panic`) listed for enforcement.
- **Done Criteria:** [ ] Rules stated · [ ] Lint configuration example

## REQ-0104
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** `unsafe` policy: crates default to `#![forbid(unsafe_code)]`; `unsafe` is permitted only when a baselined REQ or recorded decision authorizes it, confined to a dedicated module, and every `unsafe` block carries a `// SAFETY:` comment stating the upheld invariants.
- **Constraint:** Unauthorized `unsafe` is a halt condition (REQ-0109).
- **Verification Criteria:** Policy stated with example; halt condition cross-referenced.
- **Done Criteria:** [ ] Default forbid · [ ] Authorization rule · [ ] SAFETY comment example

## REQ-0105
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-04)
- **Requirement:** Async guidance: library crates stay runtime-agnostic (no hard dependency on a specific executor unless a REQ requires it); application examples use tokio; all I/O and waits are bounded (timeouts, bounded channels); blocking work is kept off async executors.
- **Constraint:** Implements build-agent secure-coding rule 5 (bounded operations).
- **Verification Criteria:** Reviewer confirms each rule with a tokio example showing timeout + bounded channel.
- **Done Criteria:** [ ] Runtime-neutral library rule · [ ] tokio example · [ ] Bounded ops · [ ] Blocking guidance

## REQ-0106
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Security patterns cover: input validation via newtypes / parse-don't-validate at trust boundaries; parameterized queries (e.g. `sqlx` bind parameters, never `format!` into SQL); secrets from environment or secret manager, never hardcoded or logged (e.g. `secrecy`); least-privilege file paths; dependency vetting (`cargo deny` for advisories, licenses, sources).
- **Constraint:** Maps to build-agent secure-coding rules 1, 3, 4, 6, 7.
- **Verification Criteria:** Each listed topic present with example or command.
- **Done Criteria:** [ ] Validation · [ ] Queries · [ ] Secrets · [ ] Paths · [ ] Dependencies

## REQ-0107
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Testability guidance covers unit tests in-module (`#[cfg(test)]`), integration tests in `tests/`, doctests for public API, and property-based tests (e.g. `proptest`) — stated as structure the Build Agent makes possible, while test-suite design remains with `test-designer`.
- **Constraint:** Must not instruct the Build Agent to verify its own artifacts.
- **Verification Criteria:** Reviewer confirms testing section and explicit independence statement.
- **Done Criteria:** [ ] Test layouts described · [ ] Independence stated

## REQ-0108
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Evidence by risk class: **L1** `cargo fmt --check`, `cargo clippy --all-targets --all-features -- -D warnings`, `cargo test` (all pass); **L2** adds coverage (`cargo llvm-cov`, threshold for critical modules), `cargo deny check` (0 unignored advisories, licenses allowed), SBOM (`cargo cyclonedx` or equivalent), MSRV build check; **L3** adds `cargo miri test` for crates containing `unsafe`, `unsafe` inventory (e.g. `cargo geiger`), fuzzing for parsers at trust boundaries (`cargo fuzz`), optional model checking (Kani), and independent review.
- **Constraint:** Each item has an objective threshold (REQ-0006).
- **Verification Criteria:** Commands and thresholds present per class.
- **Done Criteria:** [ ] L0 · [ ] L1 · [ ] L2 · [ ] L3 items present with thresholds

## REQ-0109
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Rust-specific halt conditions include at minimum: unauthorized `unsafe`; new dependency with an unignored RUSTSEC advisory, disallowed license, or unvetted source; change requiring an MSRV or edition bump not in the baseline; public API break not authorized by a REQ; `unwrap`/`panic` path in non-test code without invariant justification; required runtime/executor choice not specified. Includes a halt protocol (stop, flag, present, wait for human, resume after gate).
- **Constraint:** Inherited build-agent halts also listed.
- **Verification Criteria:** Reviewer confirms each condition and protocol.
- **Done Criteria:** [ ] Inherited halts · [ ] Six Rust halts · [ ] Halt protocol

## REQ-0110
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Context engineering: never load `target/`, vendored crates, generated code, or full `Cargo.lock` into context; read `Cargo.toml` from disk; one crate (or module) per sub-agent context; reference dependency docs by crate/version rather than pasting.
- **Constraint:** —
- **Verification Criteria:** Each rule present.
- **Done Criteria:** [ ] All rules present

## REQ-0111
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** When to Use lists project types, auto-trigger file patterns (`**/*.rs`, `**/Cargo.toml`, `rust-toolchain.toml`, `clippy.toml`, `deny.toml`), and task keywords; and states the routing exclusion for `no_std`/embedded.
- **Constraint:** —
- **Verification Criteria:** Section contains patterns, keywords, exclusion.
- **Done Criteria:** [ ] Project types · [ ] File patterns · [ ] Keywords · [ ] Exclusion

---

## build-agent-embedded-rust (no_std / MCU Rust)

## REQ-0201
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-02, SD-05)
- **Requirement:** Scope covers `no_std`/`no_main` firmware for microcontrollers (e.g. Cortex-M, RISC-V; `thumbv*`/`riscv32*` targets) at risk classes L0–L2. The skill is self-contained: it extends `build-agent` and does not require `build-agent-embedded` or `build-agent-rust`.
- **Constraint:** Hardware awareness (build-agent Principle #4) is restated in embedded-Rust terms within this file.
- **Verification Criteria:** Frontmatter `extends: build-agent`; no prerequisite on other domain skills; scope statement present.
- **Done Criteria:** [ ] Scope stated · [ ] Sibling independence confirmed

## REQ-0202
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-05)
- **Requirement:** L3/L4 boundary: tasks classified L3 or higher, or subject to functional-safety standards (ISO 26262, IEC 61508, IEC 62304, DO-178C, or equivalent), are a halt condition — the skill stops and escalates to the human and `safety-engineer`, and states that it provides no guidance or claim for qualified toolchains or certification.
- **Constraint:** Evidence Requirements section stops at L2.
- **Verification Criteria:** Halt condition and scope disclaimer present; no L3 evidence section.
- **Done Criteria:** [ ] Halt present · [ ] Disclaimer present · [ ] No L3 section

## REQ-0203
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Project structure covers: `.cargo/config.toml` target and runner (`probe-rs`), `memory.x` / linker configuration, `rust-toolchain.toml` with target, layered crates PAC → HAL → BSP → application, and separation of hardware-independent logic into a host-testable crate.
- **Constraint:** `memory.x` values must come from baselined requirements or Logic Gatekeeper constraints.
- **Verification Criteria:** Reviewer finds each item with an example.
- **Done Criteria:** [ ] Config · [ ] memory.x · [ ] Toolchain · [ ] Layering · [ ] Host-testable crate

## REQ-0204
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Drivers are written against `embedded-hal` 1.0 (and `embedded-hal-async` where async) traits rather than concrete HAL types; `unsafe` is confined to PAC/HAL/startup layers with `// SAFETY:` comments; application code contains no direct register access.
- **Constraint:** Direct register access in application code is a halt condition.
- **Verification Criteria:** Example generic driver over `embedded-hal` traits; `unsafe` confinement rule stated.
- **Done Criteria:** [ ] Trait-based driver example · [ ] unsafe confinement · [ ] No-register-access rule

## REQ-0205
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21 (SD-03)
- **Requirement:** Concurrency guidance uses Embassy as the primary model (tasks, `embassy-sync` bounded channels/mutexes, timers) and documents RTIC as the alternative when hard real-time priority-based preemption is required. Shared state uses `critical-section`/framework primitives; `static mut` is prohibited.
- **Constraint:** All channels and queues are bounded; interrupt priorities must come from requirements.
- **Verification Criteria:** Embassy example present; RTIC selection criteria present; `static mut` prohibition stated.
- **Done Criteria:** [ ] Embassy example · [ ] RTIC alternative · [ ] Shared-state rule · [ ] Bounded queues

## REQ-0206
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Memory guidance: no heap by default (`heapless`, `static_cell`); a global allocator is permitted only when a REQ authorizes it; stack overflow protection is addressed (e.g. `flip-link`); RAM/flash budgets are tracked against requirement limits.
- **Constraint:** —
- **Verification Criteria:** Each rule present with example.
- **Done Criteria:** [ ] No-heap default · [ ] Allocator authorization · [ ] Stack protection · [ ] Budget tracking

## REQ-0207
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Panic and fault handling: the panic handler is an explicit, requirement-driven choice (e.g. `panic-probe` for development; reset/halt behavior for production per REQ); watchdog use and fault/HardFault handling follow requirements; logging uses `defmt`.
- **Constraint:** Missing production panic behavior in requirements is a halt condition.
- **Verification Criteria:** Rules present; halt cross-referenced.
- **Done Criteria:** [ ] Panic handler rule · [ ] Watchdog/fault rule · [ ] defmt logging

## REQ-0208
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Testability guidance: host-side unit tests for hardware-independent logic (with mock HALs such as `embedded-hal-mock`), on-target tests (e.g. `defmt-test` or `embedded-test` via `probe-rs`), and HIL tests — with test-suite design remaining with `test-designer`.
- **Constraint:** Must not instruct the Build Agent to verify its own artifacts.
- **Verification Criteria:** Each test tier described; independence stated.
- **Done Criteria:** [ ] Host tests · [ ] On-target tests · [ ] HIL · [ ] Independence stated

## REQ-0209
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Evidence by risk class (L0–L2 only): **L1** `cargo fmt --check`, `cargo clippy --target <triple> -- -D warnings`, `cargo build --release --target <triple>` (0 warnings), host unit tests pass; **L2** adds flash/RAM usage report (e.g. `cargo size`) against MCU limits, stack usage estimate with margin, on-target/HIL test results, `cargo deny check`, `unsafe` inventory, measured timing against requirement deadlines.
- **Constraint:** Each item has an objective threshold (REQ-0006).
- **Verification Criteria:** Commands and thresholds present for L0, L1, L2.
- **Done Criteria:** [ ] L0 · [ ] L1 · [ ] L2 items with thresholds

## REQ-0210
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Embedded-Rust halt conditions include at minimum: L3+/safety-critical task (REQ-0202); pin, clock, peripheral, interrupt-priority, or power assignment not in baselined requirements or Logic Gatekeeper constraints; memory map undefined or flash/RAM budget exceeded; stack margin below the project threshold; unauthorized `unsafe` outside PAC/HAL/startup; direct register access in application code; heap use without authorization; unspecified production panic behavior; target lacks HAL support; timing deadline unmet. Includes the halt protocol.
- **Constraint:** Inherited build-agent halts also listed.
- **Verification Criteria:** Reviewer confirms each condition and protocol.
- **Done Criteria:** [ ] Inherited halts · [ ] Ten embedded halts · [ ] Halt protocol

## REQ-0211
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** Context engineering: never load PAC sources, SVD files, full datasheets/reference manuals, ELF/bin/hex artifacts, or `target/` into context; extract needed hardware facts into `.agile-v/REQUIREMENTS.md` or a referenced constraints file; one driver/module per sub-agent context.
- **Constraint:** —
- **Verification Criteria:** Each rule present.
- **Done Criteria:** [ ] All rules present

## REQ-0212
- **Lifecycle:** draft_persisted | **Revision:** r1 | **Lineage:** Stakeholder Directive: Yes — Wes Cravens, 2026-11-21
- **Requirement:** When to Use lists project types, auto-trigger file patterns (`memory.x`, `.cargo/config.toml` with embedded target, `Embed.toml`, `build.rs` with linker args, source containing `#![no_std]`), and keywords (Embassy, RTIC, embedded-hal, probe-rs, defmt, no_std, MCU families); states that `std` Rust routes to `build-agent-rust` and C/C++ firmware to `build-agent-embedded`.
- **Constraint:** —
- **Verification Criteria:** Section contains patterns, keywords, routing statements.
- **Done Criteria:** [ ] Project types · [ ] File patterns · [ ] Keywords · [ ] Routing

---

## Traceability Summary

| Area | REQs |
|---|---|
| Common (both files) | REQ-0001 – REQ-0009 |
| build-agent-rust | REQ-0101 – REQ-0111 |
| build-agent-embedded-rust | REQ-0201 – REQ-0212 |

## Review Log

| Stage | Status | Date |
|---|---|---|
| draft_persisted | Complete (r1) | 2026-11-21 |
| independent_findings (Logic Gatekeeper) | Pending | — |
| architect_revisions | Pending | — |
| Gate 1 | Pending | — |
| approved / baselined | Pending | — |
