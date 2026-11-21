# Requirements (Blueprint)
<!-- Project: build-agent-rust + build-agent-embedded-rust domain skills | Cycle: C1 | Revision set: r5 | Created: 2026-11-21 | Human Gate 1: APPROVED 2026-11-21 (Wes Cravens; r5 -> BASELINE-0001; CR-0001 -> BASELINE-0002) | Baseline: BASELINE-0002 -->
<!-- Revision history: r1 (2026-11-21, archived at .agile-v/archive/REQUIREMENTS_C1_r1.md) -> Logic Gatekeeper FAIL (27 findings) -> r2 (2026-11-21, archived) -> Logic Gatekeeper FAIL (16 new findings) -> r3 (2026-11-21, archived) -> Logic Gatekeeper PASS WITH FINDINGS (1 major, 5 minor) -> r4 (2026-11-21, archived) -> Logic Gatekeeper PASS WITH FINDINGS (3 minor; ready for Gate 1) -> r5 (2026-11-21, minor fixes per stakeholder instruction) -->

## Context

| Field | Value |
|---|---|
| Deliverable | Two standalone Agent Skills contracts: `.agile-v/staging/build-agent-rust/SKILL.md` and `.agile-v/staging/build-agent-embedded-rust/SKILL.md` |
| Destination | Written to staging here; the stakeholder moves them to another project. No integration into this repository. |
| Proposed risk class | `L1`: a routine documentation/skill contract with no production code path. Confirm at Gate 1. |
| Lifecycle | All 35 REQs are `baselined` in BASELINE-0002: REQ-0209 at r6 (CR-0001) and the rest at r5. BASELINE-0002 is the valid synthesis input and supersedes BASELINE-0001. Any change requires a CR in `.agile-v/CHANGE_LOG.md` and a new revision. |

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
| `LG-xxx` | Resolution of a Logic Gatekeeper finding (any review round: LG-0xx r1, LG-1xx r2, LG-2xx r3) |

### Stakeholder Decisions (Wes Cravens)

Evidence: SD-01 came from the stakeholder's chat instruction. SD-02 to SD-05 and SD-06 to SD-09 came from the stakeholder's explicit selections in structured multiple-choice prompts during the Claude Code session on 2026-11-21. The stakeholder re-confirms all of them at Gate 1. *(LG-115)*

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
| A-03 | Toolchain baseline: Rust stable, `embedded-hal` 1.0. A pinned nightly is permitted only per SD-07, and for embedded stack analysis per REQ-0209. |
| A-04 | **Host L4 halts and escalates** (mirrors SD-05). Neither skill provides L4 guidance. *(resolves LG-009)* |
| A-05 | Frontmatter keeps the Agile V convention of a list-valued `metadata.sections_index`, even though AgentSkills.io describes `metadata` as string→string. The destination project's validator must accept it, or the stakeholder flattens it after the move. *(LG-014)* |
| A-06 | `author: agile-v.org`, matching the released domain skills. *(LG-014)* |
| A-07 | **Skill-wide MSRV floor 1.85 and edition 2024.** This covers resolver 3 (Cargo ≥1.84), `[lints]` tables (≥1.74), and lint `reason =` / `#[expect]` (≥1.81), so the skills need no fallbacks for older toolchains. A project that needs an older MSRV is a halt. *(resolves LG-101, LG-108)* |
| A-08 | **Out of scope for both skills; halt and ask:** Xtensa targets (these need the non-stable `esp` toolchain), ESP-IDF `std` targets (`*-espidf`), and UEFI targets (`*-uefi`). Bare-metal targets without a maintained `embedded-hal` 1.0 HAL fall under the REQ-0210 HAL halt. ESP32-C/H RISC-V parts remain in scope. *(resolves LG-111, LG-112)* |

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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-01, SD-06, LG-001
- **Requirement:** Deliver exactly two files, `.agile-v/staging/build-agent-rust/SKILL.md` and `.agile-v/staging/build-agent-embedded-rust/SKILL.md`. Each is self-contained and needs no supporting files.
- **Constraint:** Synthesis edits no tracked file outside `.agile-v/`.
- **Verification Criteria:** (1) `git status --porcelain` shows no changes outside `.agile-v/`. (2) The repository test suite (`pytest`) gives the same pass/fail result before and after synthesis.
- **Done Criteria:** [ ] Both files exist at the staging paths · [ ] Directory name equals frontmatter `name` · [ ] No tracked files outside `.agile-v/` modified · [ ] Test suite result unchanged

## REQ-0002
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** REF:domain-skills, A-01, A-05, A-06, LG-014
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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** REF:domain-skills, LG-013
- **Requirement:** Each file contains exactly these `##` sections, in this order: `Inherited Rules`, `SCOPE-V Participation`, `<Domain> Architecture & Patterns`, `Evidence Requirements`, `Halt Conditions`, `Context Engineering`, `Output Format`, `When to Use`. `<Domain>` is `Rust` for build-agent-rust and `Embedded Rust` for build-agent-embedded-rust.
- **Constraint:** The structural reference is the released domain skills (8 sections), not the 7-section template.
- **Verification Criteria:** Extract the `##` headings and compare them to the exact ordered list.
- **Done Criteria:** [ ] Exact headings · [ ] Exact order

## REQ-0004
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Prerequisites, BA:Procedures, BA:Secure Coding, BA:Pre-Execution Validation, BA:Post-Verification Feedback Loop, BA:Multi-Cycle Versioning, BA:AI Influence Traceability, BA:Qualification Evidence Duties, LG-015
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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Build Manifest (per-artifact header), LG-012, LG-113
- **Requirement:** An **artifact example** is any fenced code block tagged `rust`, `toml`, or `ld`, unless it carries the `WRONG` label. A **label** is the last non-blank line before the opening fence, written exactly as `**WRONG:** <text>` or `**Fragment:** <text>`. A block has at most one label. `WRONG` blocks are exempt from the header and the compile check. `Fragment` blocks are exempt from the compile check only. Each artifact example starts with a traceability header: `// Implements: REQ-XXXX@revision; baseline: BASELINE-XXXX; <description>` (rust), `# Implements: ...` (toml), or `/* Implements: ... */` (ld).
- **Constraint:** Do not use the legacy `Parent: REQ-XXXX` form. Each example is ≤30 lines; larger examples are split. Shell commands use `bash`/`sh` fences and are not artifact examples.
- **Verification Criteria:** Script: extract the fenced blocks, apply the definition, and assert the header and line limit.
- **Done Criteria:** [ ] 100% of artifact examples carry a header · [ ] No `Parent:` headers · [ ] All examples ≤30 lines

## REQ-0006
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** RISK:L0-L4, BA:Halt Conditions, LG-005, LG-009
- **Requirement:** Evidence Requirements has one subsection per in-scope risk class. Each subsection states that base evidence applies, then lists Rust-specific additions. Each addition gives (a) a runnable command or named report and (b) an objective pass threshold. L0 states "Base evidence applies; no Rust-specific additions." Out-of-scope classes are named in a closing statement that points to the halt condition (REQ-0109 and REQ-0210).
- **Constraint:** A threshold is objective when a script or exit code can decide it (e.g. "exit code 0", "0 findings", "≥ 80% lines").
- **Verification Criteria:** For every addition, the reviewer identifies the command/report and the threshold. Any addition missing either one fails.
- **Done Criteria:** [ ] L0 statement · [ ] Each addition has command + threshold · [ ] Out-of-scope classes point to halts

## REQ-0007
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** A-02, SD-01, LG-015, LG-016
- **Requirement:** The files are portable. They contain no repository-relative links or paths such as `docs/`, `domains/`, or `templates/`. Other skills are referenced by name only. A skill other than `build-agent` and `agile-v-core` is referenced conditionally where it may not be installed ("route to X if installed; otherwise halt and ask").
- **Constraint:** Agile V runtime paths (`.agile-v/REQUIREMENTS.md`, `.agile-v/VERIFICATION_SUMMARY.md`, `.agile-v/aibom/...`, `DECISION_LOG.md`) and external `https://` URLs are allowed.
- **Verification Criteria:** grep for `](`, `docs/`, `domains/`, `templates/`. Every hit must be an external URL or an allowed runtime path. Cross-routing to non-core skills must be phrased conditionally.
- **Done Criteria:** [ ] No repo-relative references · [ ] Conditional routing phrasing

## REQ-0008
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Build Manifest, BA:Multi-Cycle Versioning, BA:AI Influence Traceability, LG-025
- **Requirement:** The Output Format section contains:
  - An example C1 Build Manifest (`ART-XXXX | REQ-XXXX@revision | baseline-id | implements | path | notes`) and a statement of the C2+ format (`ART-XXXX.N | REQ-XXXX@revision | baseline-id | implements | path | CYCLE | CR | notes`).
  - Header fields: cycle, task, risk class, toolchain (rustc version, edition, MSRV, and any nightly used plus its purpose), and target (target triple; for embedded also the MCU, RAM, and flash).
  - An evidence summary listing each evidence command run with its result, and the AI_RUN_MANIFEST path.
- **Constraint:** The manifest lists paths, not file contents.
- **Verification Criteria:** The reviewer confirms both formats, all header fields, and all evidence-summary fields.
- **Done Criteria:** [ ] C1 example · [ ] C2+ format · [ ] Header fields · [ ] Evidence summary with AI_RUN_MANIFEST path

## REQ-0009
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** REF:domain-skills, BA:Context Engineering, LG-024
- **Requirement:** Each file is 400–900 lines and ≤45,000 bytes.
- **Constraint:** The byte cap is a measurable proxy for keeping the skill small enough to load alongside `build-agent` and `agile-v-core`.
- **Verification Criteria:** `wc -l` and `wc -c`.
- **Done Criteria:** [ ] Lines within range · [ ] Bytes ≤ 45,000

## REQ-0010
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-09, LG-011, LG-106, LG-204
- **Requirement:** Each skill's Architecture & Patterns section opens with `### Ecosystem Versions (as of 2026-11-21)`. This is a table of every crate and tool that the skill's examples or evidence commands use, with the version each example targets. Every non-`Fragment` artifact example (REQ-0005) must pass the check for its file type, using the table's versions in a scratch crate:

  | File type | Check (exit 0) |
  |---|---|
  | `.rs` source, `Cargo.toml` | `cargo check` (host) or `cargo check --target <triple>` (embedded) |
  | `clippy.toml`, `[lints]` | `cargo clippy -- -D warnings` with the file in place |
  | `deny.toml` | `cargo deny check` with the file in place |
  | `memory.x` / `ld` | `cargo build --release --target <triple>` links |
  | WASM `.rs` source | `cargo check --target <wasm-triple>` |
  | `.cargo/config.toml`, `rust-toolchain.toml`, `Embed.toml` | Parses as TOML, and the scratch build above uses it without error (`Embed.toml`: parses as TOML only) |
 The skill instructs the Build Agent to compare the project's actual dependency versions to the table before reusing a pattern, and to halt if a major-version (or 0.x minor-version) mismatch changes an API the pattern uses.
- **Constraint:** Versions are verified against crates.io at synthesis time. At most 25% of artifact examples per skill may be `Fragment`.
- **Verification Criteria:** `red-team-verifier` (if installed) or an independent reviewer extracts the non-fragment artifact examples into scratch crates with the table's versions. Each must pass the check for its file type in the table above (exit 0). Count fragments against the 25% limit.
- **Done Criteria:** [ ] Version table present and dated · [ ] All non-fragment examples pass their file-type check · [ ] Fragments ≤ 25% · [ ] Version-mismatch halt stated

## REQ-0011
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-08, A-08, LG-008, LG-102, LG-103, LG-111, LG-114, LG-201, LG-202, LG-302
- **Requirement:** Both skills contain the same deterministic routing rule in When to Use. A **workspace** (or standalone crate) is a **firmware project** if any of the following holds:
  - `.cargo/config.toml` sets `[build] target` to a bare-metal triple (`*-none`, `*-none-*`, `avr-*`), excluding `wasm32*`. A target listed only in `rust-toolchain.toml` `targets` is **not** an indicator, because that list only installs targets;
  - a `memory.x` file is present;
  - the build links `link.x`, `memory.x`, or `defmt.x`.

  `#![no_main]` alone is **not** an indicator, because cargo-fuzz targets, UEFI programs and WASI programs use it too.

  Routing:
  - Every crate in a firmware project routes to `build-agent-embedded-rust`, including its host-testable crate. Crates that a firmware crate depends on by `path`, directly or transitively, are part of the firmware project, even when they sit in a sibling directory or separate workspace. A crate shared by firmware and host crates is part of the firmware project. The one exception is a crate that a baselined REQ designates as a separate host product (e.g. a desktop companion tool): it routes to `build-agent-rust`.
  - Everything else routes to `build-agent-rust`: `std` crates, `no_std` libraries outside a firmware project (e.g. `embedded-hal` driver crates), WASM, and embedded-Linux userspace.
  - A-08 targets (Xtensa, `*-espidf`, `*-uefi`) halt and ask.
  - Firmware indicators take precedence over the generic `*.rs`/`Cargo.toml` triggers.
  - Every cross-skill route is conditional: route to the named skill (`build-agent-rust`, `build-agent-embedded-rust`, or `build-agent-embedded` for C/C++ firmware) if it is installed; otherwise halt and ask.
- **Constraint:** —
- **Verification Criteria:** The routing text is identical in both files. The reviewer applies it to these cases and gets these results:
  - standalone `no_std` driver lib → rust
  - Cortex-M bin with `memory.x` → embedded
  - host-testable core crate in that Cortex-M workspace → embedded
  - host-testable core crate in a sibling directory, path-depended on by that Cortex-M bin → embedded
  - standalone `no_std` lib whose `rust-toolchain.toml` lists `thumbv7em-none-eabihf` for cross-build checks → rust
  - `wasm32v1-none` lib → rust
  - crate path-depended on transitively (bin → BSP → shared `protocol` crate), also used by a host CLI → embedded
  - `fuzz/` cargo-fuzz target in a host workspace → rust
  - `wasm32-unknown-unknown` lib → rust
  - aarch64 Linux daemon → rust
  - `xtensa-esp32-none-elf` bin → halt
  - `x86_64-unknown-uefi` app → halt
- **Done Criteria:** [ ] Identical rule in both files · [ ] All twelve cases route as specified · [ ] Precedence stated · [ ] All cross-skill routes conditional

---

## build-agent-rust (everything except bare-metal firmware)

## REQ-0101
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-08, A-04, LG-008, LG-009
- **Requirement:** The skill's scope is all Rust except bare-metal firmware (REQ-0011): libraries, including `no_std`/`no_std + alloc`, CLIs, services/APIs, WASM, embedded-Linux userspace, and workspaces. It covers risk classes L0–L3. L4 is a halt condition (REQ-0109). `no_std` library and WASM guidance is in REQ-0112.
- **Constraint:** —
- **Verification Criteria:** When to Use lists the in-scope project types, the L0–L3 scope, and references REQ-0011 routing.
- **Done Criteria:** [ ] Scope listed · [ ] L4 out of scope stated

## REQ-0102
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** ECO, A-03, A-07, LG-017, LG-108
- **Requirement:** Project-structure guidance covers:
  - Cargo workspace layout: `crates/`, `[workspace.dependencies]`, and `[workspace.lints]` with `[lints] workspace = true` in members. Single-package crates use `[lints]` directly.
  - An explicit `resolver = "3"` in virtual workspaces.
  - A declared `rust-version` of at least 1.85, and edition 2024 (A-07). A requirement for an older MSRV is a halt (REQ-0109).
  - Additive-only feature flags.
  - `Cargo.lock` committed by default for every package type. Any deviation needs a `DECISION_LOG.md` entry.
- **Constraint:** —
- **Verification Criteria:** Each topic is present. An example workspace `Cargo.toml` passes REQ-0010.
- **Done Criteria:** [ ] Workspace + lints example · [ ] resolver 3 · [ ] MSRV floors · [ ] Features · [ ] Lockfile default

## REQ-0103
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Secure Coding 2, ECO, A-07, LG-003, LG-101
- **Requirement:** Error-handling rules:
  - Fallible operations return `Result`.
  - Libraries define typed errors (e.g. `thiserror`). Applications may aggregate errors (e.g. `anyhow`) with context.
  - `[workspace.lints]` enables `clippy::unwrap_used`, `clippy::expect_used`, `clippy::panic`, and `clippy::indexing_slicing` at `warn` level (promoted to errors by L1 evidence).
  - A justified exception uses the narrowest-scope `#[expect(<lint>, reason = "INVARIANT: <why infallible>")]` (valid at the A-07 MSRV floor).
  - Test code is exempted via `clippy.toml` (`allow-unwrap-in-tests`, `allow-expect-in-tests`, and the panic/indexing equivalents where the pinned clippy supports them) or via `#[cfg(test)]`-scoped allows.
- **Constraint:** A bare `// INVARIANT:` comment without the attribute does not satisfy this REQ.
- **Verification Criteria:** The rules, lint configuration, `clippy.toml`, and an `#[expect]` example are present and pass REQ-0010. The `#[expect]` example passes `cargo clippy -- -D warnings`.
- **Done Criteria:** [ ] Four lints configured · [ ] `#[expect]` mechanism · [ ] Test exemption

## REQ-0104
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Secure Coding, ECO, LG-002, LG-007
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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-04, BA:Secure Coding 5, LG-018
- **Requirement:** Async guidance:
  - Library crates don't depend on a specific executor unless a REQ requires one.
  - Libraries expose bounded APIs: caller-supplied deadlines/limits, bounded buffers, and bounded channels.
  - Applications apply timeouts with tokio (`tokio::time::timeout`).
  - Blocking work runs off the async executor (`spawn_blocking` or a dedicated thread).
- **Constraint:** —
- **Verification Criteria:** Each rule is present, including a tokio application example with a timeout and a bounded channel, which passes REQ-0010.
- **Done Criteria:** [ ] Runtime-neutral library rule · [ ] Bounded API rule · [ ] tokio timeout + bounded channel example · [ ] Blocking guidance

## REQ-0106
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Secure Coding 1, 3, 4, 6, 7, ECO, LG-019
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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Red Team Readiness, ECO
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
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** RISK:L0-L4, SD-07, ECO, LG-004, LG-005, LG-009, LG-109, LG-116
- **Requirement:** Evidence by risk class. Each item's threshold is in parentheses.
  - **L0:** base evidence only.
  - **L1:**
    - `cargo fmt --all --check` (exit 0)
    - `cargo clippy --workspace --all-targets --all-features -- -D warnings` (exit 0)
    - `cargo test --workspace` (exit 0)
    - the REQ-0112 `no_std`/WASM evidence, when applicable (exit 0)
  - **L2:** adds
    - `cargo llvm-cov --workspace --fail-under-lines 80` (exit 0), plus ≥90% line coverage per file for modules the REQs tag as critical (checked from `--json`)
    - `cargo deny check` (exit 0)
    - an SBOM via `cargo cyclonedx` (file produced; path in manifest)
    - an MSRV check, `cargo +<rust-version> check --workspace --lib --bins` (exit 0; dev-dependencies are excluded from MSRV)
    - an `unsafe` inventory listing each block's file:line, authorization reference, and SAFETY comment (0 unlisted blocks, found by grep for `unsafe`)
  - **L3:** adds
    - `cargo +nightly-YYYY-MM-DD miri test` for crates containing `unsafe` (0 UB reports; every Miri-skipped test has `#[cfg_attr(miri, ignore)]` with a justification; FFI paths listed as Miri-uncovered)
    - `cargo +nightly-YYYY-MM-DD fuzz run <target> -- -max_total_time=600` for each parser at a trust boundary (0 crashes or timeouts; corpus archived)
    - `cargo kani` when a REQ marks a function for proof (all harnesses `VERIFICATION:- SUCCESSFUL`)
    - I2 role-separated independent verification by `red-team-verifier` (if installed; otherwise halt), recorded in `.agile-v/VERIFICATION_SUMMARY.md` (record present, verdict PASS), plus I3 explicit human sign-off recorded in the Gate 2 approval record (record present). AI verification alone is not independent assurance.
  - **L4:** out of scope; see REQ-0109.
- **Constraint:** The nightly date is pinned and recorded in the manifest (SD-07).
- **Verification Criteria:** Every item has the stated command and threshold (REQ-0006).
- **Done Criteria:** [ ] L0 · [ ] L1 (3 items, plus the REQ-0112 items when applicable) · [ ] L2 (5 items) · [ ] L3 (4 items) · [ ] L4 out-of-scope statement

## REQ-0109
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Halt Conditions, A-04, A-07, A-08, ECO, LG-002, LG-003, LG-019
- **Requirement:** The skill names the inherited `build-agent` halts and adds these Rust halts:
  1. Unauthorized `unsafe` (REQ-0104).
  2. A new dependency with an advisory not ignored per REQ-0106, a license not allowed, or a source not on the `[sources]` allow list.
  3. An MSRV or edition change not in the baseline.
  4. A public API break not authorized by a REQ.
  5. Use of `unwrap`/`expect`/`panic!`/indexing without the REQ-0103 `#[expect]` justification.
  6. A required executor/runtime choice that is not specified.
  7. An L4 task: escalate to the human, plus `safety-engineer`, `threat-modeler`, or `agile-v-compliance` as applicable, if installed.
  8. A dependency version mismatch per REQ-0010.
  9. A required MSRV below the A-07 floor, or an A-08 target.

  The section includes the halt protocol: stop, flag, present the specific issue, wait for the human, and resume only after the gate clears.
- **Constraint:** —
- **Verification Criteria:** The inherited halts are named, all nine Rust halts are present, and the protocol is present.
- **Done Criteria:** [ ] Inherited halts · [ ] Nine Rust halts · [ ] Halt protocol

## REQ-0110
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Context Engineering, ECO
- **Requirement:** Context-engineering rules:
  - Never load `target/`, vendored crates, generated code, or the full `Cargo.lock` into context.
  - Read `Cargo.toml` from disk.
  - Use one crate (or module) per sub-agent context.
  - Reference dependency documentation by crate and version (docs.rs) rather than pasting it.
- **Constraint:** —
- **Verification Criteria:** Each rule is present.
- **Done Criteria:** [ ] All rules present

## REQ-0111
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** REF:domain-skills, SD-08
- **Requirement:** When to Use contains:
  - The project types.
  - The REQ-0011 routing rule.
  - Auto-trigger file patterns: `**/*.rs`, `**/Cargo.toml`, `rust-toolchain.toml`, `clippy.toml`, `deny.toml`.
  - Task keywords.
- **Constraint:** Generic triggers yield to the firmware indicators in REQ-0011.
- **Verification Criteria:** The section contains the patterns, keywords, and routing rule.
- **Done Criteria:** [ ] Project types · [ ] Routing rule · [ ] File patterns · [ ] Keywords

## REQ-0112
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-08, ECO, LG-107, LG-205
- **Requirement:** Guidance for the `no_std` and WASM parts of the scope:
  - **`no_std` libraries:**
    - `#![no_std]` with an optional `std` feature (additive) and `alloc` behind a feature.
    - No panicking or heap-allocating APIs without authorization.
    - Evidence (L1+): `cargo build -p <crate> --no-default-features --target thumbv7em-none-eabihf` or another bare-metal triple named in the REQs (exit 0), plus host `cargo test -p <crate> --all-features` (exit 0).
  - **WASM:**
    - Target selection (`wasm32-unknown-unknown` with `wasm-bindgen`, or `wasm32-wasip2`).
    - No reliance on threads, filesystem, or system time unless the target provides them.
    - Evidence (L1+): `cargo build -p <crate> --target <wasm-triple>` (exit 0), plus `wasm-pack test --node <crate-dir>` or `cargo test -p <crate> --target wasm32-wasip2` with a configured runner (exit 0).
- **Constraint:** These items are required only when the project contains a `no_std` library or a WASM target.
- **Verification Criteria:** Both subsections are present, with commands and thresholds (REQ-0006). The examples pass REQ-0010.
- **Done Criteria:** [ ] no_std guidance + evidence · [ ] WASM guidance + evidence · [ ] Conditional applicability stated

---

## build-agent-embedded-rust (bare-metal MCU firmware)

## REQ-0201
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-02, SD-05, SD-08, A-08, BA:Hardware Awareness, LG-112
- **Requirement:** The skill's scope is bare-metal firmware as defined in REQ-0011, for any stable-Rust bare-metal MCU target that has a maintained HAL implementing `embedded-hal` 1.0, at risk classes L0–L2. For example: `thumbv*` Cortex-M and `riscv32*` (including ESP32-C/H). A-08 targets are out of scope. The skill extends `build-agent` and does not require any other domain skill. It restates hardware awareness in embedded-Rust terms.
- **Constraint:** —
- **Verification Criteria:** Frontmatter has `extends: build-agent`. There is no prerequisite on another domain skill. The scope statement is present.
- **Done Criteria:** [ ] Scope stated · [ ] Sibling independence

## REQ-0202
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-05, RISK:L0-L4, LG-023
- **Requirement:** L3/L4 boundary. These halt and escalate:
  - Any task classified L3 or higher.
  - Any task subject to a functional-safety standard (ISO 26262, IEC 61508, IEC 62304, DO-178C, or equivalent).
  - Trust-boundary firmware work: secure boot, OTA update / bootloader, key storage, or cryptographic protocol implementation.

  Escalation goes to the human plus the applicable skill if installed: `safety-engineer`, `threat-modeler`, or `agile-v-compliance`. The skill states that it provides no guidance or claim for qualified toolchains or certification.
- **Constraint:** Evidence Requirements has no L3 or L4 subsection.
- **Verification Criteria:** The halt, the trust-boundary examples, the escalation targets, and the disclaimer are present. There is no L3 or L4 evidence subsection.
- **Done Criteria:** [ ] Halt · [ ] Trust-boundary examples · [ ] Escalation targets · [ ] Disclaimer

## REQ-0203
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** ECO, BA:Hardware Awareness, LG-006, LG-104, LG-105
- **Requirement:** Project-structure guidance covers:
  - `.cargo/config.toml` target and `probe-rs` runner.
  - `memory.x`, with linker arguments (`-Tlink.x`, `-Tdefmt.x`) emitted from `build.rs` via `cargo:rustc-link-arg`, **not** set in `target.<triple>.rustflags`, so that `RUSTFLAGS` cannot override them.
  - Firmware binary targets declare `test = false` and `bench = false` in `Cargo.toml`.
  - `rust-toolchain.toml` with the target.
  - The layering PAC → HAL → BSP → application.
  - Hardware-independent logic in a **host-testable crate that sits outside the target-configured directory** (or is tested with an explicit host target), with both host test command variants stated (REQ-0209).
- **Constraint:** The `memory.x` values come from baselined requirements or Logic Gatekeeper constraints.
- **Verification Criteria:** Each item is present with an example. The examples pass REQ-0010. The host test command runs without cross-compiling.
- **Done Criteria:** [ ] Config · [ ] memory.x + build.rs link args · [ ] test/bench = false · [ ] Toolchain · [ ] Layering · [ ] Host-testable crate + command

## REQ-0204
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** ECO, LG-007, LG-110
- **Requirement:**
  - Drivers are written against `embedded-hal` 1.0 traits (and `embedded-hal-async` where async), not concrete HAL types.
  - Third-party PAC/HAL crates are dependencies, not Build Agent artifacts.
  - Any project-authored `unsafe` requires authorization by a REQ or decision record (same rule as REQ-0104(a)). It may appear only in project-owned PAC, HAL, BSP, or startup code, carries `// SAFETY:` comments enforced by `clippy::undocumented_unsafe_blocks`, and its module is the only one with `#[allow(unsafe_code)]`.
  - The application crate uses `#![deny(unsafe_code)]` and does no direct register access.
  - User-written exception and fault handlers that require `unsafe` (e.g. cortex-m-rt `#[exception] unsafe fn HardFault`) are startup code. They live in the BSP crate's authorized startup module, not in the application crate.
- **Constraint:** Violations are halt conditions (REQ-0210).
- **Verification Criteria:** A generic driver example over `embedded-hal` traits passes REQ-0010. The layer and authorization rules are stated.
- **Done Criteria:** [ ] Trait-based driver example · [ ] Dependency vs artifact rule · [ ] Authorization + layer list · [ ] App crate deny

## REQ-0205
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** SD-03, ECO, LG-020
- **Requirement:** Concurrency guidance:
  - Embassy is the primary model: tasks, `embassy-sync` bounded channels and mutexes, timers, and `InterruptExecutor` for priority preemption.
  - RTIC is documented as the alternative. Choose it when a REQ needs compile-time Stack Resource Policy analysis (deadlock-free, bounded-blocking resource sharing) or schedulability evidence derived from static priorities.
  - Shared state uses `critical-section` or framework primitives. `static mut` is prohibited.
- **Constraint:** All channels and queues are bounded. Interrupt priorities come from requirements.
- **Verification Criteria:** An Embassy example passes REQ-0010. The RTIC selection criteria are stated as above. The `static mut` prohibition is stated.
- **Done Criteria:** [ ] Embassy example · [ ] RTIC criteria · [ ] Shared-state rule · [ ] Bounded queues

## REQ-0206
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** ECO, LG-021, LG-206
- **Requirement:** Memory guidance:
  - No heap by default (`heapless`, `static_cell`). A global allocator is permitted only when a REQ authorizes it.
  - Stack-overflow protection per architecture: `flip-link` on Cortex-M; PMP stack-guard regions or HAL-provided guards on RISC-V; stack painting as a fallback.
  - RAM and flash budgets are tracked against the limits in the requirements.
- **Constraint:** —
- **Verification Criteria:** Each rule is present, with architecture alternatives.
- **Done Criteria:** [ ] No-heap default · [ ] Allocator authorization · [ ] Per-arch stack protection · [ ] Budget tracking

## REQ-0207
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** ECO, LG-021
- **Requirement:** Panic and fault handling:
  - The panic handler is an explicit choice driven by requirements, e.g. `panic-probe` for development and reset/halt behaviour for production per a REQ.
  - Watchdog use and architecture fault handlers (e.g. Cortex-M HardFault) follow the requirements.
  - Logging defaults to `defmt` unless a REQ specifies another transport.
- **Constraint:** If the requirements don't specify production panic behaviour, that is a halt condition.
- **Verification Criteria:** The rules are present and the halt is cross-referenced.
- **Done Criteria:** [ ] Panic handler rule · [ ] Watchdog/fault rule · [ ] defmt default

## REQ-0208
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Red Team Readiness, ECO
- **Requirement:** Testability guidance covers:
  - Host-side unit tests for the host-testable crate, using mock HALs (e.g. `embedded-hal-mock` with its embedded-hal 1.0 feature).
  - On-target tests (e.g. `defmt-test` or `embedded-test` via `probe-rs`).
  - HIL tests.

  Test-suite design remains with `test-designer` (if installed).
- **Constraint:** The skill must not instruct the Build Agent to verify its own artifacts.
- **Verification Criteria:** All three test tiers and the independence statement are present.
- **Done Criteria:** [ ] Host tests · [ ] On-target tests · [ ] HIL · [ ] Independence statement

## REQ-0209
- **Lifecycle:** baselined | **Revision:** r6 | **Baseline:** BASELINE-0002 | **derives_from:** SD-05, RISK:L0-L4, A-07, ECO, LG-005, LG-006, LG-022, LG-027, LG-104, LG-105, LG-116, LG-301, LG-303, CR-0001
- **Requirement:** Evidence for L0–L2 only. Each item's threshold is in parentheses.
  - **L0:** base evidence only.
  - **L1:**
    - `cargo fmt --all --check` (exit 0)
    - `cargo clippy --target <triple> -- -D warnings` on firmware crates, plus `cargo clippy -p <crate> --all-targets -- -D warnings` on the host-testable crate, adding `--target <host-triple>` when the crate is inside the target-configured directory (exit 0 for both)
    - `cargo build --release --target <triple> --config 'target.<triple>.rustflags=["-D","warnings"]'` (exit 0). Cargo appends `--config` arrays to the existing target rustflags. `RUSTFLAGS` is not used, because it replaces them. Triples containing a dot are written **unquoted** in the key (e.g. `target.thumbv8m.main-none-eabihf.rustflags`), and the same applies to `[target.<triple>]` tables in `.cargo/config.toml`. Cargo silently ignores quoted dotted triples. A `cfg(...)` target table is an accepted alternative. Firmware projects put rustflags under `target.<triple>`, never under `[build] rustflags`, which target rustflags override.
    - the host-testable crate's tests (exit 0): `cargo test -p <crate>` when the crate is outside the target-configured directory, or `cargo test -p <crate> --target <host-triple>` (e.g. `x86_64-unknown-linux-gnu`, `aarch64-apple-darwin`) when it is inside
  - **L2:** adds
    - a flash/RAM report (e.g. `cargo size --release`; each ≤ the REQ limit, or ≤ 90% of device capacity if no REQ limit)
    - a stack-usage measurement by static analysis or runtime stack painting (margin ≥ 20% unless a REQ specifies otherwise)
    - on-target/HIL tests (100% of designated tests pass)
    - `cargo deny check` (exit 0)
    - an `unsafe` inventory (0 unlisted blocks, as in REQ-0108)
    - host-crate coverage via `cargo llvm-cov -p <crate> --fail-under-lines 80`, adding `--target <host-triple>` when the crate is inside the target-configured directory (exit 0)
    - an SBOM via `cargo cyclonedx` (file produced)
    - an MSRV check, `cargo +<rust-version> check --release --target <triple>` (exit 0)
  - **Any risk class:** when a REQ states a timing deadline, measured worst-case timing is recorded (≤ deadline).
  - **L3/L4:** out of scope; see REQ-0202.
- **Constraint:** A pinned nightly may be used only for static stack analysis, and it is recorded in the manifest.
- **Verification Criteria:** Every item has the stated command/method and threshold (REQ-0006).
- **Done Criteria:** [ ] L0 · [ ] L1 (4 items) · [ ] L2 (8 items) · [ ] Timing rule · [ ] L3/L4 out-of-scope statement

## REQ-0210
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Halt Conditions, BA:Hardware Awareness, SD-05, ECO, LG-007, LG-022
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
  12. A required MSRV below the A-07 floor, or an A-08 target.

  The section includes the halt protocol.
- **Constraint:** —
- **Verification Criteria:** The inherited halts are named, all twelve embedded halts are present, and the protocol is present.
- **Done Criteria:** [ ] Inherited halts · [ ] Twelve embedded halts · [ ] Halt protocol

## REQ-0211
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** BA:Context Engineering, ECO
- **Requirement:** Context-engineering rules:
  - Never load PAC sources, SVD files, full datasheets or reference manuals, ELF/bin/hex artifacts, or `target/` into context.
  - Extract the needed hardware facts into `.agile-v/REQUIREMENTS.md` or a referenced constraints file.
  - Use one driver or module per sub-agent context.
- **Constraint:** —
- **Verification Criteria:** Each rule is present.
- **Done Criteria:** [ ] All rules present

## REQ-0212
- **Lifecycle:** baselined | **Revision:** r5 | **Baseline:** BASELINE-0002 (carried forward from BASELINE-0001) | **derives_from:** REF:domain-skills, SD-08, LG-026, LG-102, LG-201
- **Requirement:** When to Use contains:
  - The project types.
  - The REQ-0011 routing rule.
  - Auto-trigger file patterns: `memory.x`, `Embed.toml`, `.cargo/config.toml` with a bare-metal `[build] target`, and `build.rs` linker arguments that reference `link.x`, `memory.x`, or `defmt.x`. (`#![no_main]` alone is not a trigger; see REQ-0011.)
  - Keywords: Embassy, RTIC, embedded-hal, probe-rs, defmt, no_main, and MCU families.
- **Constraint:** —
- **Verification Criteria:** The section contains the patterns, keywords, and routing rule.
- **Done Criteria:** [ ] Project types · [ ] Routing rule · [ ] File patterns · [ ] Keywords

---

## Traceability Summary

| Area | REQs |
|---|---|
| Common (both files) | REQ-0001 – REQ-0011 |
| build-agent-rust | REQ-0101 – REQ-0112 |
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

## LG Finding Resolution (r2 → r3)

| Finding | Resolution |
|---|---|
| LG-101 | A-07 MSRV floor 1.85; REQ-0103 fallback removed |
| LG-102 | REQ-0011 / REQ-0212: `#![no_main]` is no longer an indicator; cargo-fuzz case added |
| LG-103 | REQ-0011 routes by firmware project (workspace); host-testable crate → embedded; host-product exception |
| LG-104 | REQ-0203 `test = false`/`bench = false`; REQ-0209 splits clippy into firmware and host-crate runs |
| LG-105 | REQ-0203 link args via `build.rs`; REQ-0209 `RUSTFLAGS` safe |
| LG-106 | REQ-0010 per-file-type check table |
| LG-107 | New REQ-0112 for no_std libraries and WASM |
| LG-108 | A-07; REQ-0102 `[lints]` for single packages, member `workspace = true` |
| LG-109 | REQ-0108 L3: I2 `red-team-verifier` + I3 Gate 2 approval record |
| LG-110 | REQ-0204 fault handlers in BSP startup module |
| LG-111 | A-08 ESP-IDF/UEFI halt; aarch64-none covered by the `*-none` indicator and the HAL halt |
| LG-112 | A-08 Xtensa out of scope; REQ-0201 limited to stable-Rust targets |
| LG-113 | REQ-0005 exact label syntax; WRONG/Fragment exemptions |
| LG-114 | REQ-0011 all cross-skill routes conditional |
| LG-115 | Stakeholder decision evidence note; re-confirmed at Gate 1 |
| LG-116 | REQ-0108 MSRV `--lib --bins`; REQ-0209 MSRV command stated |

## LG Finding Resolution (r3 → r4)

| Finding | Resolution |
|---|---|
| LG-201 | REQ-0011 counts only `.cargo/config.toml` `[build] target`, excludes `wasm32*`; `rust-toolchain.toml` excluded; two routing cases added; REQ-0212 aligned |
| LG-202 | REQ-0011 counts `path` dependencies of firmware crates as part of the firmware project; routing case added |
| LG-203 | REQ-0209 `--config` target rustflags instead of `RUSTFLAGS`; both host-test command variants |
| LG-204 | REQ-0010 criteria reference the file-type table; WASM row |
| LG-205 | REQ-0108 attaches REQ-0112 evidence at L1; `-p <crate>` added |
| LG-206 | REQ-0206 Xtensa removed; lineage key covers all rounds; assumptions reordered |

## LG Finding Resolution (r4 → r5)

| Finding | Resolution |
|---|---|
| LG-301 | REQ-0209 dotted-triple quoting; rustflags only under `target.<triple>` |
| LG-302 | REQ-0011 transitive path dependencies and shared crates are firmware; routing case 12 |
| LG-303 | REQ-0209 host-crate clippy and coverage `--target <host-triple>` variants |

## Review Log

| Stage | Status | Date |
|---|---|---|
| draft_persisted (r1) | Complete | 2026-11-21 |
| independent_findings (r1) | FAIL: 3 blocking, 9 major, 15 minor | 2026-11-21 |
| architect_revisions (r2) | Complete | 2026-11-21 |
| independent_findings (r2) | FAIL: 3 blocking, 6 major, 7 minor (23/27 r1 findings resolved) | 2026-11-21 |
| architect_revisions (r3) | Complete | 2026-11-21 |
| independent_findings (r3) | PASS WITH FINDINGS: 0 blocking, 1 major, 5 minor; all r2 resolved, no r1 regressions | 2026-11-21 |
| architect_revisions (r4) | Complete | 2026-11-21 |
| independent_findings (r4) | PASS WITH FINDINGS: 0 blocking, 0 major, 3 minor; ready for Gate 1 | 2026-11-21 |
| architect_revisions (r5) | Complete: 3 minor fixes, applied per stakeholder instruction without a further review round | 2026-11-21 |
| Gate 1 | APPROVED by Wes Cravens. Covers r5, SD-01–SD-09 re-confirmed, A-01–A-08 and ECO-derived content ratified, risk class L1 confirmed. Record: `.agile-v/gates/GATE_1_C1.md` | 2026-11-21 |
| approved | r5 (all 35 REQs) | 2026-11-21 |
| baselined | BASELINE-0001: `.agile-v/baselines/BASELINE-0001/` | 2026-11-21 |
| CR-0001 | REQ-0209 r6 (dotted-triple quoting corrected). Approved by Wes Cravens at Gate 1. Record: `.agile-v/gates/GATE_1_C1_CR-0001.md` | 2026-11-21 |
| baselined | BASELINE-0002: `.agile-v/baselines/BASELINE-0002/` (supersedes BASELINE-0001) | 2026-11-21 |
