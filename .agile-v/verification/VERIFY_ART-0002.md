# Re-Verification Report, Round 3: ART-0002.4 (`build-agent-embedded-rust/SKILL.md`)

| Field | Value |
|---|---|
| Artifact | ART-0002.4, `.agile-v/staging/build-agent-embedded-rust/SKILL.md`. sha256 `b78eca250901c50ba90bb6100190edf2c493a021c5a4bb1977179ba85670469c` (matches the coordinator's hash); 830 lines; 41,904 bytes. |
| History | ART-0002.2 `8833f535…` (round 1) → ART-0002.3 `c466fadc…` (round 2) → ART-0002.4 (this round) |
| Requirements | `.agile-v/baselines/BASELINE-0003/REQUIREMENTS.md` (SHA256SUMS OK). In-scope REQs are REQ-0001..0011 and REQ-0201..0212. A scripted diff against BASELINE-0002, ignoring the `Lifecycle` lines, shows their text is **identical**. CR-0002 changed only REQ-0108 (host skill). Revisions: REQ-0209@r6, the rest @r5. |
| Verifier | `red-team-verifier`, fresh role-separated context. Independence class I2, not I3/I4 assurance. The builder's "text only" claim was checked by re-extracting every block and diffing it. |
| Not read | builder workspaces, BUILD_MANIFEST notes, archived findings, `VERIFY_ART-0001.md` |
| Harness | `/tmp/agilev-verify-embedded/`: `scripts/check_structure.py` and `scripts/extract2.py`; `ws4/` is the verbatim extraction; `ot4/` is the on-target Fragment build |
| Timestamp | 2026-11-21T18:05Z |

## Verdict

**PASS.** There are 0 critical, 0 major and 0 minor open findings.

- R-01 and R-02 are resolved.
- Every finding from rounds 1 and 2 (F-01..F-05, O-01) remains resolved.
- One informational observation, O-02, needs no action.

## "Text Only" Claim

I re-extracted all 18 artifact blocks and diffed them against round 2 (`ws3`). The only code difference is the on-target Fragment, which gains `use defmt_rtt as _;` plus a comment. The 16 non-Fragment examples are byte-identical to ART-0002.3. Their fence line numbers are unchanged, and so is the RTIC Fragment.

The prose changes are at:
- line 21 (HTML header: BASELINE-0003 / ART-0002.4)
- line 613 (on-target paragraph)
- line 615 (Fragment label)
- line 682 (the nightly/call-stack paragraph).

The claim is **confirmed**.

## Round-2 Findings

| ID | ART-0002.4 evidence | Status |
|---|---|---|
| R-01 (REQ-0209@r6, REQ-0006@r5) | Line 682 now contains three things. (1) "`cargo-call-stack` always builds with `-Zbuild-std`, so the pinned nightly needs the `rust-src` component (`rustup component add rust-src --toolchain nightly-YYYY-MM-DD`)." This matches the 0.1.16 source, which always passes `-Zbuild-std=core,alloc,compiler_builtins` for `no_std`. (2) "Its last tested nightly predates edition 2024, so it may not build an edition 2024 project at all." This matches the README's "Last tested nightly: 2023-11-13". (3) "If the tool fails to build or run, or reports any entry as unbounded, stack painting is mandatory and its result is the evidence." The stack-usage evidence item now always has a runnable command (painting) and an objective threshold (margin ≥ 20%). | **Resolved** |
| R-02 (REQ-0208@r5) | The line 613 paragraph now states both missing prerequisites: the defmt global logger, and that the test crate must not link `panic-probe`/`panic-reset`. The Fragment now contains `use defmt_rtt as _;` and the label cites the rules. **Empirical test:** I copied the Fragment verbatim to `firmware/app/tests/on_target.rs` and applied only the documented setup (`[[test]] name = "on_target"`, `harness = false`, `embedded-test = "0.7.3"` as a dev-dependency, and `cargo:rustc-link-arg-tests=` for `-Tlink.x`, `-Tdefmt.x`, `-Tembedded-test.x`). `cargo test --test on_target --no-run` gives **exit 0** and produces the on-target executable. Negative control: adding `use panic_reset as _;` gives exit 101 with `E0152 duplicate lang item … panic_impl`, exactly as line 613 states. | **Resolved** |

## Rounds 1 and 2 (Regression)

The underlying examples and prose are unchanged, and the full harness re-ran clean (below).

| Finding | Re-check this round | Status |
|---|---|---|
| F-01 | The call-stack command and method are still at line 674 | Resolved |
| F-02 | Line 588 is still conditional | Resolved |
| F-03 | `cargo deny check` exits 0 | Resolved |
| F-04 | Line 248 is unchanged | Resolved |
| F-05 | Line 283 is unchanged | Resolved |
| O-01 | The on-target test links (see R-02) | Resolved |

## Full Harness Re-Run

### Structure

| Check | Result |
|---|---|
| REQ-0002 | YAML parses; name `build-agent-embedded-rust` (25 chars, kebab-case); description 379 chars; metadata values exact; no `status`; `sections_index` equals the ordered `##` headings → PASS |
| REQ-0003 | 8 headings, in the exact required order → PASS |
| REQ-0009 | 830 lines (400-900); 41,904 bytes (≤45,000) → PASS |
| REQ-0005 | 18 artifact examples (9 toml, 8 rust, 1 ld) plus 3 WRONG and 2 text blocks. All headers are valid, the longest example is 30 lines, and there are 0 `Parent:` headers → PASS |
| REQ-0010 | Fragments are 2/18 = 11.1% → PASS |

### REQ-0010 file-type checks

No scaffolding was added to any non-Fragment example.

| Check | Exit |
|---|---|
| All 8 TOML files parse (`rust-toolchain.toml`, root/app/bsp/sensor-core `Cargo.toml`, `.cargo/config.toml`, `Embed.toml`, `deny.toml`) | 0 |
| `cargo check --target thumbv8m.main-none-eabihf` (from `firmware/`; covers all `.rs` examples and `build.rs`) | 0 |
| `cargo build --release --target thumbv8m.main-none-eabihf` (links `memory.x` through flip-link; uses `config.toml` and `rust-toolchain.toml`) | 0 |
| `cargo test -p sensor-core` (host, 2 passed) | 0 |
| `cargo deny check` with `deny.toml` in place ("advisories ok, bans ok, licenses ok, sources ok") | 0 |

### Evidence commands

| Command | Exit |
|---|---|
| `cargo fmt --all --check` | 0 |
| `cargo clippy --target thumbv8m.main-none-eabihf -- -D warnings` | 0 |
| `cargo clippy -p sensor-core --all-targets -- -D warnings` | 0 |
| `cargo +1.88 check --release --target thumbv8m.main-none-eabihf` (MSRV) | 0 |
| `cargo size --release -- -A` | 0 |
| `cargo llvm-cov -p sensor-core --fail-under-lines 80` | 0 |
| `cargo cyclonedx` | 0 |

### Warning gate (REQ-0209@r6)

| Case | Exit |
|---|---|
| Clean build, unquoted dotted-triple `--config` | 0 |
| Injected unused variable, unquoted key | **101** (`unused variable`) |
| Injected unused variable, quoted key | 0 (silently ignored, as line 662 states) |

### Routing and repository checks

| Check | Result |
|---|---|
| REQ-0011 | The routing section (19 lines) is byte-identical between this file and `build-agent-rust/SKILL.md` (sibling sha256 `26dcc510…b1a6`). Both have section sha256 `15f2d6f9…7138`, the same as rounds 1 and 2, so the comparison was stable on the first check. The 12 baseline cases still route as specified. |
| REQ-0007 | No repo-relative paths. The only URL is the crates.io index in `deny.toml`, an allowed external URL. Non-core skill routes are conditional. |
| REQ-0001 | 0 changes outside `.agile-v/`. pytest gives 1050 passed / 24 skipped, unchanged from the HEAD snapshot. |
| REQ-0004, 0006, 0008, 0201-0212 | Unchanged content, re-confirmed: 13 inherited rules; the L0 statement; 4 L1 and 8 L2 items, each with a command and threshold; the timing rule; the L3/L4 out-of-scope statement and no L3/L4 subsection; 12 embedded halts plus the protocol; the context rules; When to Use. |

## Per-REQ Summary

All of REQ-0001 through REQ-0011, REQ-0201 through REQ-0208, REQ-0209@r6, and REQ-0210 through REQ-0212: **PASS**.

## Observation (informational, no action required)

| ID | Note |
|---|---|
| O-02 | Building the on-target test emits 1 warning from embedded-test's macro expansion (`unexpected cfg condition name: rust_analyzer`). None of the skill's evidence commands builds test targets with `-D warnings`: L1 firmware clippy has no `--all-targets`, and the release gate builds bins only. So no evidence item is affected. It would matter only if a project added `--all-targets -D warnings` for firmware crates. |

## NOT RUN

- `cargo +nightly call-stack`: no nightly toolchain is installed. The text was checked against the cargo-call-stack 0.1.16 source and README.
- On-target and HIL execution, `probe-rs` stack painting, and timing measurement: no hardware attached. The on-target Fragment was built with `--no-run` only.
- The RTIC Fragment was not rebuilt. It is byte-identical to round 1, where it built with exit 0.
- Builds for the non-example targets (thumbv6m, thumbv7em, riscv32imac).
- pytest ran with unpinned packages on Python 3.14, because the pinned `rpds-py` does not build there.

## Verification Records

```
VER-0221 | — | REQ-0209,0006 | PASS | — | R-01 resolved: rust-src, edition-2024 caveat, mandatory painting fallback (line 682)
VER-0222 | — | REQ-0208 | PASS | — | R-02 resolved: Fragment + documented setup, cargo test --test on_target --no-run exit 0; panic crate import -> E0152 as stated
VER-0223 | — | REQ-0002,0003,0005,0009,0010 | PASS | — | structure/labels/headers/size; 18 artifacts, 2 fragments; non-fragment examples byte-identical to ART-0002.3 and pass
VER-0224 | — | REQ-0209 | PASS | — | warning gate: injected warning exit 101 (unquoted), 0 (quoted)
VER-0225 | — | REQ-0010,0209 | PASS | — | fmt, clippy x2, MSRV 1.88, deny, size, llvm-cov, cyclonedx all exit 0
VER-0226 | — | REQ-0011 | PASS | — | routing byte-identical with sibling (stable across rounds)
VER-0227 | — | REQ-0001 | PASS | — | no changes outside .agile-v/; pytest 1050 passed / 24 skipped
```

Gate 2 note: there are no open findings against ART-0002.4. The NOT RUN items above (hardware, nightly) remain residual limitations of this I2 verification.
