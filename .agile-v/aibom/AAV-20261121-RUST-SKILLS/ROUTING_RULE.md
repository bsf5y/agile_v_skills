### Routing Rule (shared by build-agent-rust and build-agent-embedded-rust)

Apply this rule per workspace before synthesis. The rule text is identical in both skills.

**Firmware project indicators.** A workspace, or a standalone crate, is a **firmware project** if any of the following holds:

1. `.cargo/config.toml` sets `[build] target` to a bare-metal triple (`*-none`, `*-none-*`, `avr-*`), excluding `wasm32*`. A target listed only in `rust-toolchain.toml` `targets` is not an indicator, because that list only installs targets.
2. A `memory.x` file is present.
3. The build links `link.x`, `memory.x`, or `defmt.x`.

`#![no_main]` alone is not an indicator, because cargo-fuzz targets, UEFI programs, and WASI programs use it too.

**Routing.**

- Every crate in a firmware project routes to `build-agent-embedded-rust`, including its host-testable crate. Crates that a firmware crate depends on by `path`, directly or transitively, are part of the firmware project, even when they sit in a sibling directory or a separate workspace. A crate shared by firmware and host crates is part of the firmware project. The one exception is a crate that a baselined REQ designates as a separate host product (for example, a desktop companion tool). That crate routes to `build-agent-rust`.
- Everything else routes to `build-agent-rust`: `std` crates, `no_std` libraries outside a firmware project (for example, `embedded-hal` driver crates), WASM, and embedded-Linux userspace.
- Xtensa targets, ESP-IDF `std` targets (`*-espidf`), and UEFI targets (`*-uefi`) are out of scope for both skills. Halt and ask.
- Firmware indicators take precedence over the generic `*.rs` and `Cargo.toml` triggers.
- Every cross-skill route is conditional. Route to the named skill (`build-agent-rust`, `build-agent-embedded-rust`, or `build-agent-embedded` for C/C++ firmware) if it is installed; otherwise halt and ask.
