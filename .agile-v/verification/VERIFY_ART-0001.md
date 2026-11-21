# Re-verification Record (round 3, final): ART-0001.3 (`build-agent-rust`)

| Field | Value |
|---|---|
| Artifact | ART-0001.3, `.agile-v/staging/build-agent-rust/SKILL.md`. sha256 `26dcc510d709e5efc6ad3f74e6c618bc582be13e7a6600c87da0689572c4b1a6` (matches the coordinator's hash). 898 lines, 42,522 bytes. |
| History | Round 1: ART-0001 `a8409223…`, PASS WITH FINDINGS (1 major, 6 minor, 2 obs). Round 2: ART-0001.2 `e9aab3f1…`, PASS WITH FINDINGS (0 major, 3 minor, 1 obs). |
| Requirements | `.agile-v/baselines/BASELINE-0003/REQUIREMENTS.md`. `shasum -c SHA256SUMS` returns OK for both files. In scope: REQ-0001 to REQ-0011 and REQ-0101 to REQ-0112 at r5, except **REQ-0108 at r6** (CR-0002). `diff` against BASELINE-0002 shows the only substantive change is the REQ-0108 L2 MSRV item; the rest is lifecycle/baseline metadata. The CR-0002 approval record exists at `.agile-v/gates/GATE_1_C1_CR-0002.md`. |
| Verifier | `red-team-verifier`, fresh role context. Independence class **I2**. The builder's fix claims were checked against the artifact and by execution. |
| Timestamp | 2026-11-21T18:07Z |
| Harness | `/tmp/agilev-verify-rust/ws3`: all 25 artifact blocks re-extracted verbatim (`blocks.py` → `art3.json`, `place3.py`). The verifier's scaffolding is unchanged from round 2. Toolchains: stable 1.96.0, plus an isolated `RUSTUP_HOME` with 1.85.1, 1.94, and 1.96.0 (llvm-tools, and now the `wasm32-wasip2` target). Tools: cargo-deny 0.22.0, wasm-pack 0.14.0 (local), node v26.7.0, and wasmtime 49.0.7 (installed this round to `/tmp/agilev-verify-rust/tools-wt`). |

## Overall verdict: **PASS**

- All round-2 findings (R-01, R-02, R-03, O-03) are resolved.
- No regressions. Round-1 fixes F-01 to F-07, O-01, and O-02 still hold.
- Every Verification Criteria and Done Criteria item for the in-scope REQs passes against BASELINE-0003, including REQ-0108@r6.
- One new observation (O-04), which affects no REQ.

## Block-level delta from ART-0001.2

A script compared `art2.json` with `art3.json`. Only one artifact example changed: block #8 (line 286, `transport.rs`) gained one doc line, `/// Once \`deadline\` has fired, do not pass it to \`within\` again: re-create it instead.` Prose changes:
- Line 21: provenance.
- Line 87: wasmtime row.
- Line 778: MSRV wording.
- Line 834: merged format line.
- Evidence Summary example (849-858).

## Status of round-2 findings

| ID | Status | Evidence |
|---|---|---|
| R-01 (REQ-0108) | **RESOLVED** | CR-0002 baselined REQ-0108@r6. Line 778 matches r6: per crate at the crate's `rust-version`, `cargo +<crate-rust-version> check -p <crate> --lib --bins`, omitting `--lib` for binary-only crates, with exit 0 for every crate, dev-deps excluded, and the workspace-wide command only when all crates share one `rust-version`. Re-run on ws3: `+1.85 -p <crate> --lib --bins` gives exit 0 for orders-core, ascii-util, orders-config, frame-codec, and text-stats. `+1.85 -p orders-cli --bins` and `+1.85 -p orders-api --bins` give exit 0, with `--lib` controls at exit 101. `+1.94 -p orders-db --lib --bins` gives exit 0, with the `+1.85` control at exit 101. Line 21 now cites `REQ-0108@r6`. |
| R-02 (REQ-0008) | **RESOLVED** | The L2 example Evidence Summary (849-858) now lists all six L2 items: llvm-cov (line coverage), critical-file jq, `cargo deny`, cyclonedx, per-crate MSRV (lib and bin-only forms), and the `unsafe` inventory with its grep. The AI_RUN_MANIFEST path is still present. |
| R-03 (REQ-0010) | **RESOLVED** | Line 87 now pins `wasmtime-cli` 49.0.7. The crates.io API shows 49.0.7 with `rust_version` 1.96.0, not yanked, while 50.0.3 and 51.0.0-rc.1/51.0.0 require 1.97.0. `cargo install --locked wasmtime-cli --version 49.0.7` on 1.96.0 gives exit 0 ("Installed package `wasmtime-cli v49.0.7`"; `wasmtime --version` prints 49.0.7). The evidence file lists `wasmtime-cli 49.0.7 2026-11-19`. The wasip2 evidence path was also executed: `CARGO_TARGET_WASM32_WASIP2_RUNNER=wasmtime cargo +1.96.0 test -p frame-codec --all-features --target wasm32-wasip2` gives exit 0, run under wasmtime. |
| O-03 (REQ-0105) | **RESOLVED** | The doc note at block #8 warns against reusing a fired deadline. The re-run stalled-transport probe passes 5/5 (exit 0): stalled → `TimedOut` <200 ms against a 20 ms deadline; normal read; byte limit; over-report; already-expired deadline. |

## New observation

| ID | Sev | REQ | Line(s) | Note |
|---|---|---|---|---|
| O-04 | obs | REQ-0112@r5 | 73, 547-578, 767 | The `proptest` 1.12.0 dev-dependency in the table does not compile for `wasm32-wasip2`: `wait-timeout` gives error E0433 (`cargo test -p orders-core --target wasm32-wasip2` exit 101). A WASI crate that adopts the proptest pattern cannot use the `cargo test --target wasm32-wasip2` evidence option without gating proptest to non-WASI targets. orders-core is not a WASM crate in the skill, and the WASM examples do not use proptest, so no REQ is affected. |

## Per-requirement results (BASELINE-0003)

| REQ | Rev | Result | Evidence |
|---|---|---|---|
| REQ-0001 | r5 | PASS | `git status --porcelain` filtered for paths outside `.agile-v/` returns nothing (grep exit 1). pytest in the current tree gives exit 0: 1050 passed, 24 skipped. pytest in a worktree of HEAD 4bcc917 (`/tmp/agilev-verify-rust/head3`) gives exit 0: 1050 passed, 24 skipped. The two logs are identical apart from timing. The worktree was removed (exit 0), and `git worktree list` shows only main. The directory name equals `name`. Provenance (line 21) cites each REQ@rev, BASELINE-0003, and ART-0001.3. |
| REQ-0002 | r5 | PASS | `struct_check.py`: all 15 checks PASS. |
| REQ-0003 | r5 | PASS | Exactly the 8 headings, in order. |
| REQ-0004 | r5 | PASS | Items 1-13 unchanged (31-43). |
| REQ-0005 | r5 | PASS | 25 artifact examples, all with headers, all ≤30 lines, no `Parent:`. 1 WRONG, 1 Fragment. `blocks.py` exit 0. |
| REQ-0006 | r5 | PASS | Every L1-L3 addition has a command and an objective threshold. L0 statement and out-of-scope pointer present. |
| REQ-0007 | r5 | PASS | Portability grep returns exit 1 (no hits). Non-core references are conditional. |
| REQ-0008 | r5 | PASS | The merged line 834 still states both the C1 format (`ART-XXXX \| REQ-XXXX@revision \| baseline-id \| implements \| path \| notes`) and the C2+ format (`ART-XXXX.N \| … \| CYCLE \| CR \| notes`) verbatim. Also present: the example C1 manifest; header fields (cycle, task, risk class, toolchain with rustc/edition/MSRV/nightly+purpose, target triples; 860-864); the evidence summary with results and the AI_RUN_MANIFEST path (857, 866); paths-not-contents (830). |
| REQ-0009 | r5 | PASS | 898 lines (400-900), 42,522 bytes (≤45,000). |
| REQ-0010 | r5 | PASS | The dated table opens the section. Fragments 1/25 (4.0%). Mismatch halt present. All non-Fragment examples pass their checks (below). |
| REQ-0011 | r5 | PASS | The 20-line routing subsection is byte-identical (`cmp`) to `.agile-v/staging/build-agent-embedded-rust/SKILL.md` (sha256 `b78eca25…69c`) and unchanged since ART-0001. All 12 cases route as in round 1. |
| REQ-0101 | r5 | PASS | Line 873. |
| REQ-0102 | r5 | PASS | Lines 95-149. The workspace manifest passes `cargo check`. |
| REQ-0103 | r5 | PASS | Lints, `#[expect]` (passes clippy `-D warnings`), and clippy.toml exemptions. |
| REQ-0104 | r5 | PASS | (a)-(e). Example passes check and clippy `-D`. |
| REQ-0105 | r5 | PASS | Rules present. Deadline probe 5/5. The tokio example passes. |
| REQ-0106 | r5 | PASS | All topics present. `cargo deny check`: licenses, bans, and sources ok. The environment advisory RUSTSEC-2026-0385 (ctutils 0.4.2) is cleared by the documented `cargo update -p ctutils --precise 0.4.8`, after which `cargo deny check` gives exit 0. |
| REQ-0107 | r5 | PASS | Layouts and the independence statement. `cargo test --workspace` gives exit 0 (doctest and proptest pass). |
| REQ-0108 | **r6** | PASS | L0; L1 (3 items plus REQ-0112); L2: llvm-cov, per-file ≥90% via `--json`, deny, cyclonedx, per-crate MSRV per r6 (verified above), and the `unsafe` inventory; L3: Miri, fuzz, Kani, I2+I3; L4 out of scope; pinned nightly recorded. |
| REQ-0109 | r5 | PASS | Inherited halts, the nine Rust halts, and the protocol. |
| REQ-0110 | r5 | PASS | Four rules present. |
| REQ-0111 | r5 | PASS | Types, routing, patterns, and keywords. |
| REQ-0112 | r5 | PASS | Guidance and evidence. thumbv7em `--no-default-features` build gives exit 0 (also with alloc, plus clippy). `cargo test -p frame-codec --all-features` gives exit 0. The wasm32-unknown-unknown build gives exit 0. `wasm-pack test --node` gives exit 0 (1 passed). The wasip2 test with the wasmtime 49.0.7 runner gives exit 0. |

## REQ-0010 file-type checks (ws3)

Every non-Fragment block was checked in the same configuration as round 2, with the same results:
- **Per-crate check:** `cargo check -p <crate> --all-targets` gives exit 0 for all 8 crates. frame-codec with `--all-features` also gives exit 0.
- **Workspace clippy:** `cargo clippy --workspace --all-targets --all-features -- -D warnings` gives exit 0. This covers clippy.toml (#6), `#[expect]` (#5), and unsafe (#7).
- **Formatting:** `cargo fmt --all --check` gives exit 0, and rustfmt on the fuzz target gives exit 0.
- **Tests:** `cargo test --workspace` gives exit 0.
- **no_std (#19-21):** thumbv7em builds and clippy give exit 0.
- **WASM (#22-24):** wasm32 check, build, and clippy give exit 0, and `wasm-pack test --node` gives exit 0.
- **Fuzz (#25):** `cargo check` in `fuzz/` gives exit 0.
- **deny.toml (#16):** `cargo deny check` reports licenses, bans, and sources ok. It exits 0 after the documented `--precise` remedy for the environment advisory.

## Not run

- Miri, `cargo fuzz run`, and `cargo kani`: no nightly toolchain, and cargo-fuzz and kani are not installed. The fuzz target was compile-checked only.
- `cargo llvm-cov` and `cargo cyclonedx`: the commands are unchanged since round 1, where they were executed and verified. They were not re-run.
- wasm-pack 0.15.1: local 0.14.0 was used.

## Verification records

```
VER-0201 | — | REQ-0108@r6 | PASS | — | R-01 resolved: per-crate MSRV per CR-0002; +1.85/+1.94 per-crate exit 0, controls 101 | SKILL.md:778
VER-0202 | — | REQ-0008@r5 | PASS | — | R-02 resolved: L2 example lists all six L2 items; merged format line complete | SKILL.md:834,849-858
VER-0203 | — | REQ-0010@r5 | PASS | — | R-03 resolved: wasmtime-cli 49.0.7 installs on 1.96.0 (exit 0); wasip2 test via wasmtime exit 0 | SKILL.md:87
VER-0204 | — | REQ-0105@r5 | PASS | — | O-03 resolved; deadline_probe 5/5 exit 0 | SKILL.md:286-315
VER-0205 | — | REQ-0112@r5 | PASS | — | O-04 observation: proptest dev-dep does not build for wasm32-wasip2 (no REQ impact) | SKILL.md:73
All in-scope REQs: PASS against BASELINE-0003.
```

Residual risk: this is I2 role-separated verification, not I3/I4 assurance. The L3 hooks (Miri, fuzz, Kani) were not executed. The `cargo deny` advisory result depends on the advisory database date.
