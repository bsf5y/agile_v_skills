# Requirements (Blueprint)
<!-- Project: build-agent-rust + build-agent-embedded-rust domain skills | Cycle: C1 | Revision set: r2 | Created: 2026-11-21 | Human Gate 1: PENDING -->
<!-- Revision history: r1 (2026-11-21, archived at .agile-v/archive/REQUIREMENTS_C1_r1.md) -> Logic Gatekeeper FAIL (27 findings) -> r2 (2026-11-21) -->

## Context

| Field | Value |
|---|---|
| Deliverable | Two standalone Agent Skills contracts: `.agile-v/staging/build-agent-rust/SKILL.md` and `.agile-v/staging/build-agent-embedded-rust/SKILL.md` |
| Destination | Written to staging here; the stakeholder moves them to another project. No integration into this repository. |
| Proposed risk class | `L1`: a routine documentation/skill contract with no production code path. Confirm at Gate 1. |
| Lifecycle | All REQs are `architect_revisions` (r2) and awaiting Logic Gatekeeper re-review. They are not synthesis input until `baselined`. |

### Lineage Key

Each REQ records typed lineage as `derives_from:`, using these source types:

| Prefix | Source |
|---|---|
| `SD-xx` | Stakeholder decision (table below): a direct directive |
| `A-xx` | Assumption: ratified only at Gate 1 |
| `BA:<rule>` | Rule in the `build-agent` skill contract |
| `REF:domain-skills` | Structure of the released Agile V domain skills (python, embedded) |
| `RISK:L0-L4` | Agile V risk classification model (via `agile-v-core`) |
| `ECO` | Architect-derived Rust/embedded-Rust ecosystem practice. **Not a stakeholder directive**; ratified at Gate 1. |
| `LG-xxx` | Resolution of a Logic Gatekeeper r1 finding |

### Stakeholder Decisions (Wes Cravens)

| ID | Date | Decision |
|---|---|---|
| SD-01 | 2026-11-21 | Skills only. No plugin.json, catalog, profile, routing-guide, README, or test integration. |
| SD-02 | 2026-11-21 | `build-agent-embedded-rust` is a **sibling**: it extends `build-agent` directly and does not depend on `build-agent-embedded` (C/C++) or `build-agent-rust`. |
| SD-03 | 2026-11-21 | Embedded concurrency: **Embassy primary**, with RTIC as the documented alternative. |
| SD-04 | 2026-11-21 | Host async: **library code stays runtime-neutral; application examples use tokio**. |
| SD-05 | 2026-11-21 | Embedded scope: **up to L2 only**. L3/L4 work halts and escalates. |
| SD-06 | 2026-11-21 | Output location: **`.agile-v/staging/<skill-name>/SKILL.md`**. |
| SD-07 | 2026-11-21 | A **pinned nightly toolchain is allowed only to run L3 analysis tools** (Miri, cargo-fuzz) in the host skill. Product code builds on stable/MSRV. |
| SD-08 | 2026-11-21 | Routing: **firmware vs everything else**. `build-agent-embedded-rust` = bare-metal MCU firmware. `build-agent-rust` = everything else (std, `no_std` libraries, WASM, embedded-Linux userspace). Firmware indicators take precedence. |
| SD-09 | 2026-11-21 | Examples are **pinned to a dated ecosystem version table** in each skill. Artifact examples must compile against it. |

### Assumptions requiring Gate 1 ratification

| ID | Assumption |
|---|---|
| A-01 | Frontmatter omits `metadata.status`. The destination project decides draft/released status. |
| A-02 | The files are portable: they reference Agile V concepts by skill name or by Agile V runtime paths (`.agile-v/...`), never by repository-relative documentation paths. |
| A-03 | Toolchain baseline: Rust stable, edition 2024 default (2021 allowed when MSRV < 1.85), `embedded-hal` 1.0. A pinned nightly is permitted only per SD-07, and for embedded stack analysis per REQ-0209. |
| A-04 | **Host L4 halts and escalates** (mirrors SD-05). Neither skill provides L4 guidance. *(resolves LG-009)* |
| A-05 | Frontmatter keeps the Agile V convention of a list-valued `metadata.sections_index`, even though AgentSkills.io describes `metadata` as string→string. The destination project's validator must accept it, or the stakeholder flattens it after the move. *(LG-014)* |
| A-06 | `author: agile-v.org`, matching the released domain skills. *(LG-014)* |

### AI Influence

```yaml
ai_influence_expected: "substantial"   # Claude drafts both SKILL.md files
ai_bom_required: true
allowed_ai_components:
  models: ["claude-opus-5-5"]
  tools: ["Claude Code"]
  rag_sources: ["this repository's released domain skills and build-agent contract (read-only reference)", "crates.io / docs.rs for version verification"]
```

---

## Common Requirements (both skills)

## REQ-0001
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-01, SD-06, LG-001
- **Requirement:** Deliver exactly two files, `.agile-v/staging/build-agent-rust/SKILL.md` and `.agile-v/staging/build-agent-embedded-rust/SKILL.md`. Each is self-contained and needs no supporting files.
- **Constraint:** Synthesis edits no tracked file outside `.agile-v/`.
- **Verification Criteria:** (1) `git status --porcelain` shows no changes outside `.agile-v/`. (2) The repository test suite (`pytest`) gives the same pass/fail result before and after synthesis.
- **Done Criteria:** [ ] Both files exist at the staging paths · [ ] Directory name equals frontmatter `name` · [ ] No tracked files outside `.agile-v/` modified · [ ] Test suite result unchanged

## REQ-0002
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** REF:domain-skills, A-01, A-05, A-06, LG-014
- **Requirement:** Each file begins with valid YAML frontmatter containing `name`, `description`, `license: CC-BY-SA-4.0`, and `metadata` with `version: "1.0"`, `standard: "Agile V"`, `extends: "build-agent"`, `author: agile-v.org`, `domain`, and `sections_index`. Required values:

  | Field | build-agent-rust | build-agent-embedded-rust |
  |---|---|---|
  | `name` | `build-agent-rust` | `build-agent-embedded-rust` |
  | `domain` | `"Rust"` | `"Embedded Rust"` |

  The `description` names the domain, contains "Extends build-agent", and states when to use the skill.
- **Constraint:** `name` ≤64 chars, kebab-case. `description` ≤1024 chars. `sections_index` equals the file's `##` headings in order. No `metadata.status` field.
- **Verification Criteria:** Parse with a YAML parser. Assert the fields, values, and limits, and that `sections_index` equals the ordered `##` headings.
- **Done Criteria:** [ ] YAML parses · [ ] Fields and values exact · [ ] Limits respected · [ ] sections_index matches headings

## REQ-0003
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** REF:domain-skills, LG-013
- **Requirement:** Each file contains exactly these `##` sections, in this order: `Inherited Rules`, `SCOPE-V Participation`, `<Domain> Architecture & Patterns`, `Evidence Requirements`, `Halt Conditions`, `Context Engineering`, `Output Format`, `When to Use`. `<Domain>` is `Rust` for build-agent-rust and `Embedded Rust` for build-agent-embedded-rust.
- **Constraint:** The structural reference is the released domain skills (8 sections), not the 7-section template.
- **Verification Criteria:** Extract the `##` headings and compare them to the exact ordered list.
- **Done Criteria:** [ ] Exact headings · [ ] Exact order

## REQ-0004
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Prerequisites, BA:Procedures, BA:Secure Coding, BA:Pre-Execution Validation, BA:Post-Verification Feedback Loop, BA:Multi-Cycle Versioning, BA:AI Influence Traceability, BA:Qualification Evidence Duties, LG-015
- **Requirement:** The Inherited Rules section states that all `build-agent` rules apply, and names each of the following:
  1. Synthesis only from approved AND baselined requirements.
  2. Typed lineage `artifact -> implements -> REQ-XXXX@revision` with a baseline reference.
  3. A Build Manifest on every delivery.
  4. No self-verification (Red Team Protocol).
  5. Human Gates, and halting on ambiguity.
  6. Decision logging.
  7. Multi-cycle artifact versioning (`ART-XXXX.N`).
  8. The secure-coding rules.
  9. Pre-execution validation.
  10. The post-verification feedback loop (≤3 auto-fix attempts, then escalate).
  11. Hardware awareness.
  12. The AI_RUN_MANIFEST at `.agile-v/aibom/<task_id>/AI_RUN_MANIFEST.yaml`.
  13. Qualification evidence duties.
- **Constraint:** Inherited rules and inherited halts are listed by name and not re-specified. The skill specifies only domain additions.
- **Verification Criteria:** Check off each of the 13 items. Confirm that no base rule is contradicted or weakened.
- **Done Criteria:** [ ] 13 items named · [ ] No contradictions with build-agent

## REQ-0005
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Build Manifest (per-artifact header), LG-012
- **Requirement:** An **artifact example** is any fenced code block tagged `rust`, `toml`, or `ld`, unless the line directly before it is a label beginning `WRONG`. Each artifact example starts with a traceability header: `// Implements: REQ-XXXX@revision; baseline: BASELINE-XXXX; <description>` (rust), `# Implements: ...` (toml), or `/* Implements: ... */` (ld).
- **Constraint:** Do not use the legacy `Parent: REQ-XXXX` form. Each example is ≤30 lines; larger examples are split. Shell commands use `bash`/`sh` fences and are not artifact examples.
- **Verification Criteria:** Script: extract the fenced blocks, apply the definition, and assert the header and line limit.
- **Done Criteria:** [ ] 100% of artifact examples carry a header · [ ] No `Parent:` headers · [ ] All examples ≤30 lines

## REQ-0006
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** RISK:L0-L4, BA:Halt Conditions, LG-005, LG-009
- **Requirement:** Evidence Requirements has one subsection per in-scope risk class. Each subsection states that base evidence applies, then lists Rust-specific additions. Each addition gives (a) a runnable command or named report and (b) an objective pass threshold. L0 states "Base evidence applies; no Rust-specific additions." Out-of-scope classes are named in a closing statement that points to the halt condition (REQ-0109 and REQ-0210).
- **Constraint:** A threshold is objective when a script or exit code can decide it (e.g. "exit code 0", "0 findings", "≥ 80% lines").
- **Verification Criteria:** For every addition, the reviewer identifies the command/report and the threshold. Any addition missing either one fails.
- **Done Criteria:** [ ] L0 statement · [ ] Each addition has command + threshold · [ ] Out-of-scope classes point to halts

## REQ-0007
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** A-02, SD-01, LG-015, LG-016
- **Requirement:** The files are portable. They contain no repository-relative links or paths such as `docs/`, `domains/`, or `templates/`. Other skills are referenced by name only. A skill other than `build-agent` and `agile-v-core` is referenced conditionally where it may not be installed ("route to X if installed; otherwise halt and ask").
- **Constraint:** Agile V runtime paths (`.agile-v/REQUIREMENTS.md`, `.agile-v/VERIFICATION_SUMMARY.md`, `.agile-v/aibom/...`, `DECISION_LOG.md`) and external `https://` URLs are allowed.
- **Verification Criteria:** grep for `](`, `docs/`, `domains/`, `templates/`. Every hit must be an external URL or an allowed runtime path. Cross-routing to non-core skills must be phrased conditionally.
- **Done Criteria:** [ ] No repo-relative references · [ ] Conditional routing phrasing

## REQ-0008
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Build Manifest, BA:Multi-Cycle Versioning, BA:AI Influence Traceability, LG-025
- **Requirement:** The Output Format section contains:
  - An example C1 Build Manifest (`ART-XXXX | REQ-XXXX@revision | baseline-id | implements | path | notes`) and a statement of the C2+ format (`ART-XXXX.N | REQ-XXXX@revision | baseline-id | implements | path | CYCLE | CR | notes`).
  - Header fields: cycle, task, risk class, toolchain (rustc version, edition, MSRV, and any nightly used plus its purpose), and target (target triple; for embedded also the MCU, RAM, and flash).
  - An evidence summary listing each evidence command run with its result, and the AI_RUN_MANIFEST path.
- **Constraint:** The manifest lists paths, not file contents.
- **Verification Criteria:** The reviewer confirms both formats, all header fields, and all evidence-summary fields.
- **Done Criteria:** [ ] C1 example · [ ] C2+ format · [ ] Header fields · [ ] Evidence summary with AI_RUN_MANIFEST path

## REQ-0009
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** REF:domain-skills, BA:Context Engineering, LG-024
- **Requirement:** Each file is 400–900 lines and ≤45,000 bytes.
- **Constraint:** The byte cap is a measurable proxy for keeping the skill small enough to load alongside `build-agent` and `agile-v-core`.
- **Verification Criteria:** `wc -l` and `wc -c`.
- **Done Criteria:** [ ] Lines within range · [ ] Bytes ≤ 45,000

## REQ-0010
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-09, LG-011
- **Requirement:** Each skill's Architecture & Patterns section opens with `### Ecosystem Versions (as of 2026-11-21)`. This is a table of every crate and tool that the skill's examples or evidence commands use, with the version each example targets. Every artifact example (REQ-0005) must type-check against the table: `cargo check` in a scratch crate for host examples, and `cargo check --target <triple>` for embedded examples. Examples labelled `Fragment` on the line before the fence are exempt from the compile check only. The skill instructs the Build Agent to compare the project's actual dependency versions to the table before reusing a pattern, and to halt if a major-version (or 0.x minor-version) mismatch changes an API the pattern uses.
- **Constraint:** Versions are verified against crates.io at synthesis time. At most 25% of artifact examples per skill may be `Fragment`.
- **Verification Criteria:** `red-team-verifier` (if installed) or an independent reviewer extracts the non-fragment artifact examples into scratch crates with the table's versions. Each must pass `cargo check` (exit 0). Count fragments against the 25% limit.
- **Done Criteria:** [ ] Version table present and dated · [ ] All non-fragment examples type-check · [ ] Fragments ≤ 25% · [ ] Version-mismatch halt stated

## REQ-0011
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-08, LG-008
- **Requirement:** Both skills contain the same deterministic routing rule in When to Use. A crate is **firmware** if any of the following holds:
  - it contains `#![no_main]`;
  - `.cargo/config.toml` or `rust-toolchain.toml` sets a bare-metal target (`*-none-*`, `*-none-elf*`, `avr-*`);
  - a `memory.x` file is present;
  - the build links `link.x`, `memory.x`, or `defmt.x`.

  Firmware routes to `build-agent-embedded-rust`. Everything else routes to `build-agent-rust`, including `std` crates, `no_std` libraries (e.g. `embedded-hal` driver crates), WASM, and embedded-Linux userspace. Firmware indicators take precedence over the generic `*.rs`/`Cargo.toml` triggers. C/C++ firmware routes to `build-agent-embedded` if installed; otherwise halt and ask.
- **Constraint:** A workspace that mixes firmware and host crates is routed per crate.
- **Verification Criteria:** The routing text is identical in both files. The reviewer applies it to these cases and gets these results:
  - `no_std` driver lib → rust
  - Cortex-M bin → embedded
  - ESP32 Xtensa `no_main` bin → embedded
  - `wasm32-unknown-unknown` lib → rust
  - aarch64 Linux daemon → rust
- **Done Criteria:** [ ] Identical rule in both files · [ ] All five cases route as specified · [ ] Precedence stated

---

## build-agent-rust (everything except bare-metal firmware)

## REQ-0101
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-08, A-04, LG-008, LG-009
- **Requirement:** The skill's scope is all Rust except bare-metal firmware (REQ-0011): libraries, including `no_std`/`no_std + alloc`, CLIs, services/APIs, WASM, embedded-Linux userspace, and workspaces. It covers risk classes L0–L3. L4 is a halt condition (REQ-0109).
- **Constraint:** —
- **Verification Criteria:** When to Use lists the in-scope project types, the L0–L3 scope, and references REQ-0011 routing.
- **Done Criteria:** [ ] Scope listed · [ ] L4 out of scope stated

## REQ-0102
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** ECO, A-03, LG-017
- **Requirement:** Project-structure guidance covers:
  - Cargo workspace layout: `crates/`, `[workspace.dependencies]`, and `[workspace.lints]`, which requires Rust ≥1.74.
  - An explicit `resolver = "3"` in virtual workspaces.
  - A declared `rust-version` (MSRV) consistent with the features used. Floors: edition 2024 needs ≥1.85, and `#[expect]`/lint reasons need ≥1.81.
  - Additive-only feature flags.
  - `Cargo.lock` committed by default for every package type. Any deviation needs a `DECISION_LOG.md` entry.
- **Constraint:** —
- **Verification Criteria:** Each topic is present. An example workspace `Cargo.toml` passes REQ-0010.
- **Done Criteria:** [ ] Workspace + lints example · [ ] resolver 3 · [ ] MSRV floors · [ ] Features · [ ] Lockfile default

## REQ-0103
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Secure Coding 2, ECO, LG-003
- **Requirement:** Error-handling rules:
  - Fallible operations return `Result`.
  - Libraries define typed errors (e.g. `thiserror`). Applications may aggregate errors (e.g. `anyhow`) with context.
  - `[workspace.lints]` enables `clippy::unwrap_used`, `clippy::expect_used`, `clippy::panic`, and `clippy::indexing_slicing` at `warn` level (promoted to errors by L1 evidence).
  - A justified exception uses the narrowest-scope `#[expect(<lint>, reason = "INVARIANT: <why infallible>")]`. Use `#[allow(..., reason = ...)]` when MSRV < 1.81.
  - Test code is exempted via `clippy.toml` (`allow-unwrap-in-tests`, `allow-expect-in-tests`, and the panic/indexing equivalents where the pinned clippy supports them) or via `#[cfg(test)]`-scoped allows.
- **Constraint:** A bare `// INVARIANT:` comment without the attribute does not satisfy this REQ.
- **Verification Criteria:** The rules, lint configuration, `clippy.toml`, and an `#[expect]` example are present and pass REQ-0010. The `#[expect]` example passes `cargo clippy -- -D warnings`.
- **Done Criteria:** [ ] Four lints configured · [ ] `#[expect]` mechanism · [ ] Test exemption

## REQ-0104
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Secure Coding, ECO, LG-002, LG-007
- **Requirement:** `unsafe` policy:
  - (a) Any project-authored `unsafe` requires authorization by a baselined REQ or a `DECISION_LOG.md` entry.
  - (b) Crates with no authorized `unsafe` use `#![forbid(unsafe_code)]`.
  - (c) Crates with authorized `unsafe` use `#![deny(unsafe_code)]` at the crate root and `#[allow(unsafe_code)]` only on the authorized module. Alternatively, the `unsafe` lives in a separate dedicated crate.
  - (d) Every `unsafe` block has a `// SAFETY:` comment stating the upheld invariants, enforced by `clippy::undocumented_unsafe_blocks`.
  - (e) Edition 2024 unsafe rules are noted: `unsafe_op_in_unsafe_fn`, `unsafe extern`, and `#[unsafe(no_mangle)]`.
- **Constraint:** Unauthorized `unsafe` is a halt condition (REQ-0109).
- **Verification Criteria:** Items (a)–(e) are present. The deny/allow example passes REQ-0010 and `cargo clippy -- -D warnings`.
- **Done Criteria:** [ ] Authorization rule · [ ] forbid vs deny+allow distinction · [ ] SAFETY lint · [ ] Edition 2024 notes

## REQ-0105
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-04, BA:Secure Coding 5, LG-018
- **Requirement:** Async guidance:
  - Library crates don't depend on a specific executor unless a REQ requires one.
  - Libraries expose bounded APIs: caller-supplied deadlines/limits, bounded buffers, and bounded channels.
  - Applications apply timeouts with tokio (`tokio::time::timeout`).
  - Blocking work runs off the async executor (`spawn_blocking` or a dedicated thread).
- **Constraint:** —
- **Verification Criteria:** Each rule is present, including a tokio application example with a timeout and a bounded channel, which passes REQ-0010.
- **Done Criteria:** [ ] Runtime-neutral library rule · [ ] Bounded API rule · [ ] tokio timeout + bounded channel example · [ ] Blocking guidance

## REQ-0106
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Secure Coding 1, 3, 4, 6, 7, ECO, LG-019
- **Requirement:** Security patterns:
  - **Input validation:** newtypes / parse-don't-validate at trust boundaries.
  - **Queries:** parameterized (e.g. `sqlx` bind parameters; never `format!` into SQL).
  - **Secrets:** from the environment or a secret manager; never hardcoded or logged (e.g. `secrecy`).
  - **Least-privilege file access:** canonicalize, then require containment under an allowed root (reject traversal), with minimal file permissions.
  - **Dependency vetting:** `cargo deny` with `[advisories]`, `[licenses]`, and `[sources]` allow lists. Each `[advisories] ignore` entry requires a `DECISION_LOG.md` reference.
- **Constraint:** —
- **Verification Criteria:** Each topic is present with an example or command. The examples pass REQ-0010.
- **Done Criteria:** [ ] Validation · [ ] Queries · [ ] Secrets · [ ] Path containment · [ ] cargo-deny config + ignore rule

## REQ-0107
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Red Team Readiness, ECO
- **Requirement:** Testability guidance covers:
  - Unit tests in-module (`#[cfg(test)]`).
  - Integration tests in `tests/`.
  - Doctests for public API.
  - Property-based tests (e.g. `proptest`).

  These are stated as structure the Build Agent makes possible. Test-suite design remains with `test-designer` (if installed).
- **Constraint:** The skill must not instruct the Build Agent to verify its own artifacts.
- **Verification Criteria:** The testing section and an explicit independence statement are present.
- **Done Criteria:** [ ] Test layouts · [ ] Independence statement

## REQ-0108
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** RISK:L0-L4, SD-07, ECO, LG-004, LG-005, LG-009
- **Requirement:** Evidence by risk class. Each item's threshold is in parentheses.
  - **L0:** base evidence only.
  - **L1:**
    - `cargo fmt --all --check` (exit 0)
    - `cargo clippy --workspace --all-targets --all-features -- -D warnings` (exit 0)
    - `cargo test --workspace` (exit 0)
  - **L2:** adds
    - `cargo llvm-cov --workspace --fail-under-lines 80` (exit 0), plus ≥90% line coverage per file for modules the REQs tag as critical (checked from `--json`)
    - `cargo deny check` (exit 0)
    - an SBOM via `cargo cyclonedx` (file produced; path in manifest)
    - an MSRV check, `cargo +<rust-version> check --workspace --all-targets` (exit 0)
    - an `unsafe` inventory listing each block's file:line, authorization reference, and SAFETY comment (0 unlisted blocks, found by grep for `unsafe`)
  - **L3:** adds
    - `cargo +nightly-YYYY-MM-DD miri test` for crates containing `unsafe` (0 UB reports; every Miri-skipped test has `#[cfg_attr(miri, ignore)]` with a justification; FFI paths listed as Miri-uncovered)
    - `cargo +nightly-YYYY-MM-DD fuzz run <target> -- -max_total_time=600` for each parser at a trust boundary (0 crashes or timeouts; corpus archived)
    - `cargo kani` when a REQ marks a function for proof (all harnesses `VERIFICATION:- SUCCESSFUL`)
    - independent verification by a separate agent context, plus explicit human sign-off recorded in `.agile-v/VERIFICATION_SUMMARY.md` (sign-off record present)
  - **L4:** out of scope; see REQ-0109.
- **Constraint:** The nightly date is pinned and recorded in the manifest (SD-07).
- **Verification Criteria:** Every item has the stated command and threshold (REQ-0006).
- **Done Criteria:** [ ] L0 · [ ] L1 (3 items) · [ ] L2 (5 items) · [ ] L3 (4 items) · [ ] L4 out-of-scope statement

## REQ-0109
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Halt Conditions, A-04, ECO, LG-002, LG-003, LG-019
- **Requirement:** The skill names the inherited `build-agent` halts and adds these Rust halts:
  1. Unauthorized `unsafe` (REQ-0104).
  2. A new dependency with an advisory not ignored per REQ-0106, a license not allowed, or a source not on the `[sources]` allow list.
  3. An MSRV or edition change not in the baseline.
  4. A public API break not authorized by a REQ.
  5. Use of `unwrap`/`expect`/`panic!`/indexing without the REQ-0103 `#[expect]` justification.
  6. A required executor/runtime choice that is not specified.
  7. An L4 task: escalate to the human, plus `safety-engineer`, `threat-modeler`, or `agile-v-compliance` as applicable, if installed.
  8. A dependency version mismatch per REQ-0010.

  The section includes the halt protocol: stop, flag, present the specific issue, wait for the human, and resume only after the gate clears.
- **Constraint:** —
- **Verification Criteria:** The inherited halts are named, all eight Rust halts are present, and the protocol is present.
- **Done Criteria:** [ ] Inherited halts · [ ] Eight Rust halts · [ ] Halt protocol

## REQ-0110
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Context Engineering, ECO
- **Requirement:** Context-engineering rules:
  - Never load `target/`, vendored crates, generated code, or the full `Cargo.lock` into context.
  - Read `Cargo.toml` from disk.
  - Use one crate (or module) per sub-agent context.
  - Reference dependency documentation by crate and version (docs.rs) rather than pasting it.
- **Constraint:** —
- **Verification Criteria:** Each rule is present.
- **Done Criteria:** [ ] All rules present

## REQ-0111
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** REF:domain-skills, SD-08
- **Requirement:** When to Use contains:
  - The project types.
  - The REQ-0011 routing rule.
  - Auto-trigger file patterns: `**/*.rs`, `**/Cargo.toml`, `rust-toolchain.toml`, `clippy.toml`, `deny.toml`.
  - Task keywords.
- **Constraint:** Generic triggers yield to the firmware indicators in REQ-0011.
- **Verification Criteria:** The section contains the patterns, keywords, and routing rule.
- **Done Criteria:** [ ] Project types · [ ] Routing rule · [ ] File patterns · [ ] Keywords

---

## build-agent-embedded-rust (bare-metal MCU firmware)

## REQ-0201
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-02, SD-05, SD-08, BA:Hardware Awareness
- **Requirement:** The skill's scope is bare-metal firmware as defined in REQ-0011, for any MCU target that has a maintained HAL implementing `embedded-hal` 1.0, at risk classes L0–L2. For example: `thumbv*` Cortex-M, `riscv32*`, and Xtensa via the esp toolchain, with the toolchain recorded in the manifest. The skill extends `build-agent` and does not require any other domain skill. It restates hardware awareness in embedded-Rust terms.
- **Constraint:** —
- **Verification Criteria:** Frontmatter has `extends: build-agent`. There is no prerequisite on another domain skill. The scope statement is present.
- **Done Criteria:** [ ] Scope stated · [ ] Sibling independence

## REQ-0202
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-05, RISK:L0-L4, LG-023
- **Requirement:** L3/L4 boundary. These halt and escalate:
  - Any task classified L3 or higher.
  - Any task subject to a functional-safety standard (ISO 26262, IEC 61508, IEC 62304, DO-178C, or equivalent).
  - Trust-boundary firmware work: secure boot, OTA update / bootloader, key storage, or cryptographic protocol implementation.

  Escalation goes to the human plus the applicable skill if installed: `safety-engineer`, `threat-modeler`, or `agile-v-compliance`. The skill states that it provides no guidance or claim for qualified toolchains or certification.
- **Constraint:** Evidence Requirements has no L3 or L4 subsection.
- **Verification Criteria:** The halt, the trust-boundary examples, the escalation targets, and the disclaimer are present. There is no L3 or L4 evidence subsection.
- **Done Criteria:** [ ] Halt · [ ] Trust-boundary examples · [ ] Escalation targets · [ ] Disclaimer

## REQ-0203
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** ECO, BA:Hardware Awareness, LG-006
- **Requirement:** Project-structure guidance covers:
  - `.cargo/config.toml` target and `probe-rs` runner.
  - `memory.x` / linker configuration.
  - `rust-toolchain.toml` with the target.
  - The layering PAC → HAL → BSP → application.
  - Hardware-independent logic in a **host-testable crate that sits outside the target-configured directory** (or is tested with an explicit host target), with its host test command stated.
- **Constraint:** The `memory.x` values come from baselined requirements or Logic Gatekeeper constraints.
- **Verification Criteria:** Each item is present with an example. The examples pass REQ-0010. The host test command runs without cross-compiling.
- **Done Criteria:** [ ] Config · [ ] memory.x · [ ] Toolchain · [ ] Layering · [ ] Host-testable crate + command

## REQ-0204
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** ECO, LG-007
- **Requirement:**
  - Drivers are written against `embedded-hal` 1.0 traits (and `embedded-hal-async` where async), not concrete HAL types.
  - Third-party PAC/HAL crates are dependencies, not Build Agent artifacts.
  - Any project-authored `unsafe` requires authorization by a REQ or decision record (same rule as REQ-0104(a)). It may appear only in project-owned PAC, HAL, BSP, or startup code, carries `// SAFETY:` comments enforced by `clippy::undocumented_unsafe_blocks`, and its module is the only one with `#[allow(unsafe_code)]`.
  - The application crate uses `#![deny(unsafe_code)]` and does no direct register access.
- **Constraint:** Violations are halt conditions (REQ-0210).
- **Verification Criteria:** A generic driver example over `embedded-hal` traits passes REQ-0010. The layer and authorization rules are stated.
- **Done Criteria:** [ ] Trait-based driver example · [ ] Dependency vs artifact rule · [ ] Authorization + layer list · [ ] App crate deny

## REQ-0205
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-03, ECO, LG-020
- **Requirement:** Concurrency guidance:
  - Embassy is the primary model: tasks, `embassy-sync` bounded channels and mutexes, timers, and `InterruptExecutor` for priority preemption.
  - RTIC is documented as the alternative. Choose it when a REQ needs compile-time Stack Resource Policy analysis (deadlock-free, bounded-blocking resource sharing) or schedulability evidence derived from static priorities.
  - Shared state uses `critical-section` or framework primitives. `static mut` is prohibited.
- **Constraint:** All channels and queues are bounded. Interrupt priorities come from requirements.
- **Verification Criteria:** An Embassy example passes REQ-0010. The RTIC selection criteria are stated as above. The `static mut` prohibition is stated.
- **Done Criteria:** [ ] Embassy example · [ ] RTIC criteria · [ ] Shared-state rule · [ ] Bounded queues

## REQ-0206
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** ECO, LG-021
- **Requirement:** Memory guidance:
  - No heap by default (`heapless`, `static_cell`). A global allocator is permitted only when a REQ authorizes it.
  - Stack-overflow protection per architecture: `flip-link` on Cortex-M; stack-guard / PMP regions or HAL-provided guards on RISC-V/Xtensa; stack painting as a fallback.
  - RAM and flash budgets are tracked against the limits in the requirements.
- **Constraint:** —
- **Verification Criteria:** Each rule is present, with architecture alternatives.
- **Done Criteria:** [ ] No-heap default · [ ] Allocator authorization · [ ] Per-arch stack protection · [ ] Budget tracking

## REQ-0207
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** ECO, LG-021
- **Requirement:** Panic and fault handling:
  - The panic handler is an explicit choice driven by requirements, e.g. `panic-probe` for development and reset/halt behaviour for production per a REQ.
  - Watchdog use and architecture fault handlers (e.g. Cortex-M HardFault) follow the requirements.
  - Logging defaults to `defmt` unless a REQ specifies another transport.
- **Constraint:** If the requirements don't specify production panic behaviour, that is a halt condition.
- **Verification Criteria:** The rules are present and the halt is cross-referenced.
- **Done Criteria:** [ ] Panic handler rule · [ ] Watchdog/fault rule · [ ] defmt default

## REQ-0208
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Red Team Readiness, ECO
- **Requirement:** Testability guidance covers:
  - Host-side unit tests for the host-testable crate, using mock HALs (e.g. `embedded-hal-mock` with its embedded-hal 1.0 feature).
  - On-target tests (e.g. `defmt-test` or `embedded-test` via `probe-rs`).
  - HIL tests.

  Test-suite design remains with `test-designer` (if installed).
- **Constraint:** The skill must not instruct the Build Agent to verify its own artifacts.
- **Verification Criteria:** All three test tiers and the independence statement are present.
- **Done Criteria:** [ ] Host tests · [ ] On-target tests · [ ] HIL · [ ] Independence statement

## REQ-0209
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** SD-05, RISK:L0-L4, ECO, LG-005, LG-006, LG-022, LG-027
- **Requirement:** Evidence for L0–L2 only. Each item's threshold is in parentheses.
  - **L0:** base evidence only.
  - **L1:**
    - `cargo fmt --all --check` (exit 0)
    - `cargo clippy --target <triple> --all-targets -- -D warnings` (exit 0)
    - `cargo build --release --target <triple>` with `-D warnings` via `RUSTFLAGS` or `--config` (exit 0)
    - the host-testable crate's tests via the REQ-0203 command (exit 0)
  - **L2:** adds
    - a flash/RAM report (e.g. `cargo size --release`; each ≤ the REQ limit, or ≤ 90% of device capacity if no REQ limit)
    - a stack-usage measurement by static analysis or runtime stack painting (margin ≥ 20% unless a REQ specifies otherwise)
    - on-target/HIL tests (100% of designated tests pass)
    - `cargo deny check` (exit 0)
    - an `unsafe` inventory (0 unlisted blocks, as in REQ-0108)
    - host-crate coverage via `cargo llvm-cov --fail-under-lines 80` (exit 0)
    - an SBOM via `cargo cyclonedx` (file produced)
    - an MSRV check if `rust-version` is declared (exit 0)
  - **Any risk class:** when a REQ states a timing deadline, measured worst-case timing is recorded (≤ deadline).
  - **L3/L4:** out of scope; see REQ-0202.
- **Constraint:** A pinned nightly may be used only for static stack analysis, and it is recorded in the manifest.
- **Verification Criteria:** Every item has the stated command/method and threshold (REQ-0006).
- **Done Criteria:** [ ] L0 · [ ] L1 (4 items) · [ ] L2 (8 items) · [ ] Timing rule · [ ] L3/L4 out-of-scope statement

## REQ-0210
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Halt Conditions, BA:Hardware Awareness, SD-05, ECO, LG-007, LG-022
- **Requirement:** The skill names the inherited `build-agent` halts and adds these embedded halts:
  1. Any REQ-0202 condition.
  2. A pin, clock, peripheral, interrupt priority, or power assignment that is in neither the baselined requirements nor the Logic Gatekeeper constraints.
  3. An undefined memory map, or flash/RAM over budget.
  4. Stack margin below the threshold in REQ-0209.
  5. Unauthorized `unsafe`, or `unsafe` outside project-owned PAC/HAL/BSP/startup code.
  6. Direct register access in application code.
  7. Heap use without authorization.
  8. Production panic behaviour not specified in the requirements.
  9. No maintained HAL implementing `embedded-hal` 1.0 for the target: writing one requires authorization.
  10. A measured timing deadline that is missed.
  11. A dependency version mismatch per REQ-0010.

  The section includes the halt protocol.
- **Constraint:** —
- **Verification Criteria:** The inherited halts are named, all eleven embedded halts are present, and the protocol is present.
- **Done Criteria:** [ ] Inherited halts · [ ] Eleven embedded halts · [ ] Halt protocol

## REQ-0211
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** BA:Context Engineering, ECO
- **Requirement:** Context-engineering rules:
  - Never load PAC sources, SVD files, full datasheets or reference manuals, ELF/bin/hex artifacts, or `target/` into context.
  - Extract the needed hardware facts into `.agile-v/REQUIREMENTS.md` or a referenced constraints file.
  - Use one driver or module per sub-agent context.
- **Constraint:** —
- **Verification Criteria:** Each rule is present.
- **Done Criteria:** [ ] All rules present

## REQ-0212
- **Lifecycle:** architect_revisions | **Revision:** r2 | **derives_from:** REF:domain-skills, SD-08, LG-026
- **Requirement:** When to Use contains:
  - The project types.
  - The REQ-0011 routing rule.
  - Auto-trigger file patterns: `memory.x`, `Embed.toml`, `.cargo/config.toml` with a bare-metal target, sources containing `#![no_main]`, and `build.rs` linker arguments that reference `link.x`, `memory.x`, or `defmt.x`.
  - Keywords: Embassy, RTIC, embedded-hal, probe-rs, defmt, no_main, and MCU families.
- **Constraint:** —
- **Verification Criteria:** The section contains the patterns, keywords, and routing rule.
- **Done Criteria:** [ ] Project types · [ ] Routing rule · [ ] File patterns · [ ] Keywords

---

## Traceability Summary

| Area | REQs |
|---|---|
| Common (both files) | REQ-0001 – REQ-0011 |
| build-agent-rust | REQ-0101 – REQ-0111 |
| build-agent-embedded-rust | REQ-0201 – REQ-0212 |

## LG Finding Resolution (r1 → r2)

| Finding | Resolution |
|---|---|
| LG-001 | SD-06 staging path; REQ-0001 test-suite criterion |
| LG-002 | REQ-0104 forbid vs deny+allow; SAFETY lint |
| LG-003 | REQ-0103 `#[expect(reason)]`, `indexing_slicing`, test exemption |
| LG-004 | SD-07; REQ-0108 pinned nightly + thresholds |
| LG-005 | REQ-0006, REQ-0108, REQ-0209 commands and thresholds |
| LG-006 | REQ-0203 host-testable crate outside target config; REQ-0209 |
| LG-007 | REQ-0204 aligned with REQ-0104(a); BSP added; app crate deny |
| LG-008 | SD-08; new REQ-0011 deterministic routing |
| LG-009 | A-04 host L4 halt; L0 statements |
| LG-010 | Typed `derives_from` lineage; `ECO` marked as not a stakeholder directive |
| LG-011 | SD-09; new REQ-0010 version table + compile check |
| LG-012 | REQ-0005 artifact-example definition, `ld` header, ≤30 lines |
| LG-013 | REQ-0003 references released skills; exact headings |
| LG-014 | REQ-0002 values; A-05, A-06 |
| LG-015 | REQ-0004 13 inherited items; runtime manifest path |
| LG-016 | REQ-0007 conditional routing phrasing |
| LG-017 | REQ-0102 lockfile default, resolver 3, MSRV floors |
| LG-018 | REQ-0105 bounded library APIs; app-level timeouts |
| LG-019 | REQ-0106 definitions; ignore rule |
| LG-020 | REQ-0205 RTIC criterion restated |
| LG-021 | REQ-0206 per-arch stack protection; REQ-0207 defmt default |
| LG-022 | REQ-0209 default margin, timing at any class; REQ-0210 HAL definition |
| LG-023 | REQ-0202 trust-boundary examples, wider escalation |
| LG-024 | REQ-0009 byte cap |
| LG-025 | REQ-0008 C2+ format and evidence-summary fields |
| LG-026 | REQ-0212 narrowed build.rs trigger |
| LG-027 | REQ-0209 adds coverage, SBOM, MSRV |

## Review Log

| Stage | Status | Date |
|---|---|---|
| draft_persisted (r1) | Complete | 2026-11-21 |
| independent_findings (r1) | FAIL: 3 blocking, 9 major, 15 minor | 2026-11-21 |
| architect_revisions (r2) | Complete | 2026-11-21 |
| independent_findings (r2) | Pending | — |
| Gate 1 | Pending | — |
| approved / baselined | Pending | — |
