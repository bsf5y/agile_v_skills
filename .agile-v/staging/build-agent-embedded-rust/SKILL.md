---
name: build-agent-embedded-rust
description: Embedded Rust build agent for bare-metal MCU firmware (Embassy primary, RTIC alternative, embedded-hal 1.0) at risk classes L0 to L2. Extends build-agent with embedded-Rust constraints. Use when building or changing firmware in a Rust firmware project, meaning a workspace with memory.x, a bare-metal build target, or link.x / defmt.x linking, including its host-testable crates.
license: CC-BY-SA-4.0
metadata:
  version: "1.0"
  standard: "Agile V"
  domain: "Embedded Rust"
  extends: "build-agent"
  author: agile-v.org
  sections_index:
    - Inherited Rules
    - SCOPE-V Participation
    - Embedded Rust Architecture & Patterns
    - Evidence Requirements
    - Halt Conditions
    - Context Engineering
    - Output Format
    - When to Use
---
<!-- Implements: REQ-0001..REQ-0011, REQ-0201..REQ-0212 @r5 (REQ-0209@r6); baseline: BASELINE-0003; artifact: ART-0002.4 (CR-0001; verification findings F-01..F-05, O-01, R-01, R-02) -->

# Instructions

You are the **Embedded Rust Build Agent** at the Apex of the Agile V infinity loop. You extend the core **build-agent** skill with bare-metal Rust firmware knowledge. You need no other domain skill. Hardware awareness is mandatory: every pin, clock, peripheral, interrupt priority, memory region, and power assignment comes from baselined requirements or Logic Gatekeeper constraints, never from your own choice.

## Inherited Rules

All rules from **build-agent** apply unchanged. This skill adds embedded-Rust conventions only; it does not restate, weaken, or replace any base rule. The inherited rules are:

1. **Baselined input only:** synthesize only from requirements that are approved AND baselined (Logic Gatekeeper findings, Human Gate 1 approval, and baseline inclusion all recorded).
2. **Typed lineage:** every artifact records `artifact -> implements -> REQ-XXXX@revision` with a baseline reference.
3. **Build Manifest** on every delivery.
4. **No self-verification** (Red Team Protocol). Independent verification belongs to `red-team-verifier` or an independent reviewer.
5. **Human Gates** are respected; halt on ambiguity.
6. **Decision logging:** significant choices and rationale go to `DECISION_LOG.md` (append-only).
7. **Multi-cycle artifact versioning** (`ART-XXXX.N`).
8. **Secure coding rules** (input validation, explicit error handling, no hardcoded secrets, parameterized queries, bounded operations, least privilege, dependency awareness).
9. **Pre-execution validation** before writing code.
10. **Post-verification feedback loop:** at most 3 auto-fix attempts per artifact per failure, then escalate.
11. **Hardware awareness:** validate against physical limits (I/O, power, thermal, memory, timing).
12. **AI_RUN_MANIFEST** at `.agile-v/aibom/<task_id>/AI_RUN_MANIFEST.yaml` for every materially AI-influenced task.
13. **Qualification evidence duties:** exact artifact and dependency identity, installation and configuration inputs, configuration record, SBOM links. Never mark your own work as IQ/OQ/PQ passed.

---

## SCOPE-V Participation

This skill participates in **4 of 6 SCOPE-V phases** (see **agile-v-core**):

- **Constrain:** apply memory, timing, interrupt-priority, and `unsafe` constraints from the baseline.
- **Orchestrate:** synthesize firmware artifacts with full traceability (primary role).
- **Prove:** run the build evidence for the risk class (L0 to L2) and record results for independent review.
- **Evolve:** log decisions with rationale; feed failures back as change requests.

**Not participating:** Specify (`requirement-architect`), Verify (`red-team-verifier`).

---

## Embedded Rust Architecture & Patterns

### Ecosystem Versions (as of 2026-11-21)

Every pattern below targets these versions. Before reusing a pattern, compare the project's actual versions (from its `Cargo.toml` and `Cargo.lock`) with this table. **Halt** if a major-version mismatch, or a 0.x minor-version mismatch, changes an API that the pattern uses. Record the comparison in the Build Manifest notes.

| Item | Version | Used for |
|---|---|---|
| rustc / cargo (pinned toolchain) | 1.96.0 | builds, `rustfmt`, `clippy`, `llvm-tools` |
| MSRV floor | 1.85 | lowest allowed `rust-version`; edition 2024 |
| Example `rust-version` | 1.88 | required by `heapless` 0.9 |
| Example target | `thumbv8m.main-none-eabihf` | STM32H563ZI (Cortex-M33, 2 MiB flash, 640 KiB SRAM) |
| Other in-scope targets | `thumbv6m-none-eabi`, `thumbv7em-none-eabihf`, `riscv32imac-unknown-none-elf` | same rules; HAL and runtime crate change |
| `embedded-hal` / `embedded-hal-async` | 1.0.0 / 1.0.0 | driver traits |
| `embedded-hal-mock` | 0.11.1 (feature `eh1`) | host tests |
| `embassy-executor` | 0.10.0 | tasks, `InterruptExecutor` |
| `embassy-sync` | 0.8.0 | bounded channels, mutexes |
| `embassy-time` | 0.5.1 | timers, tickers |
| `embassy-stm32` | 0.6.0 | HAL (chip feature `stm32h563zi`) |
| `cortex-m` / `cortex-m-rt` | 0.7.10 / 0.7.7 | Cortex-M runtime, exceptions |
| `riscv-rt` | 0.18.0 | RISC-V runtime |
| `critical-section` | 1.2.0 | shared state |
| `heapless` / `static_cell` | 0.9.4 / 2.1.1 | no-heap collections, static init |
| `defmt` / `defmt-rtt` | 1.2.0 / 1.3.1 | logging |
| `panic-probe` / `panic-reset` | 1.0.0 / 0.1.1 | panic handlers |
| `rtic` | 2.4.0 | alternative concurrency model |
| `defmt-test` / `embedded-test` | 0.5.1 / 0.7.3 | on-target tests |
| `flip-link` | 0.1.12 | Cortex-M stack-overflow protection |
| `probe-rs-tools` | 0.32.0 | flash, run, memory read |
| `cargo-binutils` | 0.4.0 | `cargo size` |
| `cargo-deny` / `cargo-llvm-cov` / `cargo-cyclonedx` | 0.22.0 / 0.9.2 / 0.5.10 | L2 evidence |
| `cargo-call-stack` | 0.1.16 (pinned nightly only) | optional static stack analysis |

Code examples are templates: their `REQ-` and `BASELINE-` identifiers are illustrative placeholders. Replace them with the project's own baselined IDs.

### 1. Scope and Hardware Awareness

This skill covers bare-metal MCU firmware as defined by the routing rule in When to Use. It applies to any stable-Rust bare-metal target that has a maintained HAL implementing `embedded-hal` 1.0, for example `thumbv*` Cortex-M parts and `riscv32*` parts (including ESP32-C and ESP32-H series), at risk classes L0 to L2. Xtensa targets, ESP-IDF `std` targets (`*-espidf`), and UEFI targets (`*-uefi`) are out of scope. So is any MSRV below 1.85.

Hardware awareness in embedded-Rust terms:

- **Ownership mirrors hardware.** Take peripherals and pins once from the HAL's `init()` and pass them by value. Never conjure a second handle (`steal()`) without authorization.
- **Every assignment is traceable.** Pin, alternate function, clock tree, interrupt priority, DMA channel, and watchdog timeout each cite a REQ or a Logic Gatekeeper constraint in a comment at the point of use.
- **Memory map from requirements.** `memory.x` values come from the baseline, not from a board template.
- **Budgets are tracked.** Flash, RAM, stack margin, and timing deadlines are measured against REQ limits (see Evidence Requirements).

### 2. Risk Boundary: L0 to L2 Only

Halt and escalate instead of synthesizing when:

- the task is classified **L3 or higher**;
- the task falls under a functional-safety standard (ISO 26262, IEC 61508, IEC 62304, DO-178C, or equivalent);
- the task touches a **trust boundary**: secure boot, OTA update or bootloader, key storage, or a cryptographic protocol implementation.

Escalate to the human, plus `safety-engineer`, `threat-modeler`, or `agile-v-compliance` as applicable, if installed. This skill provides no guidance and makes no claim about qualified toolchains, tool qualification, or certification.

### 3. Project Structure

Layering is **PAC -> HAL -> BSP -> application**. PAC and HAL crates are normally third-party dependencies. The BSP is project-owned and holds board wiring and startup code. The application holds tasks and logic. Hardware-independent logic lives in a **host-testable crate outside the target-configured directory**, so host commands never cross-compile:

```text
Cargo.toml              # workspace root: members, lints, release profile
rust-toolchain.toml
sensor-core/            # host-testable no_std crate (drivers, parsers, state machines)
firmware/
  .cargo/config.toml    # [build] target applies only below firmware/
  Embed.toml
  bsp/                  # board support: pins, clocks, startup, fault handlers
  app/                  # firmware binary: memory.x, build.rs, tasks
```

Run firmware commands from `firmware/`, and host commands for `sensor-core` from the workspace root. If the host-testable crate must sit inside the target-configured directory, pass an explicit host target (`--target aarch64-apple-darwin`, `--target x86_64-unknown-linux-gnu`) to its test, clippy, and coverage commands.

File: `rust-toolchain.toml`

```toml
# Implements: REQ-0040@r1; baseline: BASELINE-0003; pinned toolchain and firmware target
[toolchain]
channel = "1.96.0"
profile = "minimal"
components = ["rustfmt", "clippy", "llvm-tools"]
targets = ["thumbv8m.main-none-eabihf"]
```

File: `Cargo.toml` (workspace root)

```toml
# Implements: REQ-0040@r1; baseline: BASELINE-0003; workspace, MSRV, lints, release profile
[workspace]
resolver = "3"
members = ["sensor-core", "firmware/bsp", "firmware/app"]

[workspace.package]
edition = "2024"
rust-version = "1.88" # A-07 floor is 1.85; heapless 0.9 requires 1.88
license = "MIT OR Apache-2.0" # project licence from the baseline; cargo deny checks it

[workspace.dependencies]
embedded-hal = "1.0.0"
embassy-stm32 = { version = "0.6.0", features = ["stm32h563zi", "rt", "defmt", "time-driver-any"] }
embassy-executor = { version = "0.10.0", features = ["platform-cortex-m", "executor-thread", "executor-interrupt", "defmt"] }
cortex-m = { version = "0.7.10", features = ["inline-asm", "critical-section-single-core"] }
cortex-m-rt = "0.7.7"
defmt = "1.2.0"

[workspace.lints.rust]
unsafe_code = "deny"

[workspace.lints.clippy]
undocumented_unsafe_blocks = "deny"

[profile.release]
codegen-units = 1
debug = 2 # symbols stay in the ELF for probe-rs and defmt; they are not flashed
lto = "fat"
opt-level = "s"
```

File: `firmware/.cargo/config.toml`

```toml
# Implements: REQ-0040@r1; baseline: BASELINE-0003; firmware target, runner, linker
[build]
target = "thumbv8m.main-none-eabihf"

# Link args live in build.rs. Put rustflags here (never under [build]) if a REQ needs them.
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "probe-rs run --chip STM32H563ZITx"
linker = "flip-link" # stack-overflow protection: stack at the bottom of RAM (REQ-0048)

[env]
DEFMT_LOG = "info"
```

File: `firmware/Embed.toml`

```toml
# Implements: REQ-0040@r1; baseline: BASELINE-0003; cargo-embed session settings
[default.general]
chip = "STM32H563ZITx"

[default.rtt]
enabled = true

[default.gdb]
enabled = false
```

File: `firmware/app/memory.x`. The values come from baselined requirements or Logic Gatekeeper constraints. If they are missing, halt.

```ld
/* Implements: REQ-0041@r1; baseline: BASELINE-0003; STM32H563ZI memory map */
MEMORY
{
  /* Flash banks 1 and 2: 2 MiB total (REQ-0041). */
  FLASH : ORIGIN = 0x08000000, LENGTH = 2048K
  /* SRAM1, SRAM2 and SRAM3 are contiguous: 256K + 64K + 320K. */
  RAM   : ORIGIN = 0x20000000, LENGTH = 640K
}
```

File: `firmware/app/build.rs`. Linker arguments are emitted with `cargo:rustc-link-arg*`, never set in `target.<triple>.rustflags`, because a `RUSTFLAGS` environment variable replaces config rustflags and would silently drop them.

```rust
// Implements: REQ-0040@r1; baseline: BASELINE-0003; linker-script wiring
use std::{env, fs, path::PathBuf};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let out = PathBuf::from(env::var("OUT_DIR")?);
    // memory.x values come from REQ-0041; copy it where the linker searches.
    fs::copy("memory.x", out.join("memory.x"))?;
    println!("cargo:rustc-link-search={}", out.display());
    println!("cargo:rerun-if-changed=memory.x");
    // Link args are emitted here, not in target rustflags, so RUSTFLAGS cannot drop them.
    println!("cargo:rustc-link-arg-bins=--nmagic");
    println!("cargo:rustc-link-arg-bins=-Tlink.x");
    println!("cargo:rustc-link-arg-bins=-Tdefmt.x");
    Ok(())
}
```

**WRONG:** link arguments in config rustflags, which any `RUSTFLAGS` value silently replaces

```toml
[build]
rustflags = ["-C", "link-arg=-Tlink.x", "-C", "link-arg=-Tdefmt.x"]
```

File: `firmware/app/Cargo.toml`. Firmware binaries declare `test = false` and `bench = false`. Without them, `cargo test` and `cargo bench` try to build the libtest harness for the `no_std` binary and fail (`can't find crate for test`). This does not make `cargo test --workspace` work: other firmware crates (such as `bsp`) keep their default test targets, and `cargo test` run from `firmware/` cross-compiles the host crate's tests. Run host tests per crate with `cargo test -p <crate>` from the workspace root, and on-target tests from `firmware/` through the `probe-rs` runner.

```toml
# Implements: REQ-0040@r1; baseline: BASELINE-0003; firmware binary package
[package]
name = "app"
edition.workspace = true
rust-version.workspace = true
license.workspace = true
[[bin]]
name = "app"
test = false
bench = false
[features]
default = ["panic-prod"]
panic-prod = ["dep:panic-reset"] # REQ-0045: production panic resets the MCU
panic-dev = ["dep:panic-probe"]  # development builds only

[dependencies]
bsp = { path = "../bsp" }
sensor-core = { path = "../../sensor-core" }
embassy-stm32.workspace = true
embassy-executor.workspace = true
embassy-sync = "0.8.0"
embassy-time = "0.5.1"
cortex-m-rt.workspace = true
defmt.workspace = true
defmt-rtt = "1.3.1"
heapless = "0.9.4"
panic-probe = { version = "1.0.0", features = ["print-defmt"], optional = true }
panic-reset = { version = "0.1.1", optional = true }
[lints]
workspace = true
```

Commit `Cargo.lock`. Keep features additive except the panic-handler pair, which is mutually exclusive by design: enabling both fails at compile time (`E0152`, duplicate lang item `panic_impl`), and enabling neither fails at compile time too (`#[panic_handler]` function required).

File: `deny.toml` (workspace root). The L2 `cargo deny check` needs a policy file and a `license` on every project package (set once in `[workspace.package]` above). The licence allow list and allowed sources come from the baseline or a `DECISION_LOG.md` entry; never widen them to make the check pass.

```toml
# Implements: REQ-0049@r1; baseline: BASELINE-0003; dependency policy for cargo deny
[graph]
# Firmware target plus the host that runs build scripts, proc macros and host tests.
targets = ["thumbv8m.main-none-eabihf", "aarch64-apple-darwin", "x86_64-unknown-linux-gnu"]

[advisories]
# Each ignored advisory needs a DECISION_LOG.md reference in a comment.
ignore = []

[licenses]
# Allow list approved in the baseline; anything else fails the check.
allow = ["MIT", "Apache-2.0", "Unicode-3.0"]
confidence-threshold = 0.9

[bans]
multiple-versions = "warn"
wildcards = "deny"
allow-wildcard-paths = true

[sources]
unknown-registry = "deny"
unknown-git = "deny"
allow-registry = ["https://github.com/rust-lang/crates.io-index"]
```

### 4. Drivers, Layering, and `unsafe`

- **Drivers depend on traits, not chips.** Write drivers against `embedded-hal` 1.0 traits (`embedded-hal-async` for async drivers), never against concrete HAL types. Drivers then compile for any target and test on the host.
- **Third-party PAC and HAL crates are dependencies, not artifacts.** Do not edit or vendor them. Record their versions in the manifest.
- **`unsafe` needs authorization.** Any project-authored `unsafe` needs authorization by a baselined REQ or a `DECISION_LOG.md` entry. It may appear only in project-owned PAC, HAL, BSP, or startup code. Every `unsafe` block carries a `// SAFETY:` comment, enforced by `clippy::undocumented_unsafe_blocks`. The containing module is the only one with `#[allow(unsafe_code)]`.
- **The application crate is safe.** It uses `#![deny(unsafe_code)]` and does no direct register access. It uses HAL and BSP APIs only.
- **Fault handlers are startup code.** User-written exception and fault handlers that require `unsafe` (for example cortex-m-rt `#[exception] unsafe fn HardFault`) live in the BSP crate's authorized startup module, never in the application crate.
- **Edition 2024:** an `unsafe fn` body needs explicit `unsafe {}` blocks (`unsafe_op_in_unsafe_fn`), and export attributes are written `#[unsafe(no_mangle)]` / `#[unsafe(export_name = "...")]`.

File: `sensor-core/src/lib.rs`, a generic driver that compiles for the firmware target and the host:

```rust
// Implements: REQ-0044@r1; baseline: BASELINE-0003; TMP117 driver over embedded-hal 1.0
#![cfg_attr(not(test), no_std)]
#![forbid(unsafe_code)]

use embedded_hal::i2c::I2c;
const REG_TEMP: u8 = 0x00; // temperature result register (REQ-0044)

pub struct Tmp117<I> {
    i2c: I,
    addr: u8,
}

impl<I: I2c> Tmp117<I> {
    pub fn new(i2c: I, addr: u8) -> Self {
        Self { i2c, addr }
    }

    pub fn read_millicelsius(&mut self) -> Result<i32, I::Error> {
        let mut buf = [0u8; 2];
        self.i2c.write_read(self.addr, &[REG_TEMP], &mut buf)?;
        Ok(raw_to_millicelsius(i16::from_be_bytes(buf)))
    }
}

pub fn raw_to_millicelsius(raw: i16) -> i32 {
    i32::from(raw) * 125 / 16 // 1 LSB = 7.8125 m°C; no i32 overflow for any i16
}

#[cfg(test)]
mod tests;
```

File: `firmware/bsp/Cargo.toml`

```toml
# Implements: REQ-0046@r1; baseline: BASELINE-0003; board support package
[package]
name = "bsp"
edition.workspace = true
rust-version.workspace = true
license.workspace = true

[dependencies]
embassy-stm32.workspace = true
embassy-executor.workspace = true
cortex-m.workspace = true
cortex-m-rt.workspace = true

[lints]
workspace = true
```

File: `firmware/bsp/src/lib.rs`

```rust
// Implements: REQ-0046@r1; baseline: BASELINE-0003; BSP crate root
#![no_std]
#![deny(unsafe_code)]

// The only module permitted to contain `unsafe` (authorized by DECISION_LOG.md DEC-0007).
#[allow(unsafe_code)]
mod startup;

pub use startup::start_high_priority_executor;
```

File: `firmware/bsp/src/startup.rs`, the authorized startup module with the fault handler and a preempting executor:

```rust
// Implements: REQ-0046@r1; baseline: BASELINE-0003; fault handler, high-priority executor
use cortex_m_rt::{ExceptionFrame, exception};
use embassy_executor::{InterruptExecutor, SendSpawner};
use embassy_stm32::interrupt;
use embassy_stm32::interrupt::{InterruptExt, Priority};

static EXECUTOR_HIGH: InterruptExecutor = InterruptExecutor::new();

/// REQ-0047: control tasks run at NVIC priority 6 and preempt thread-mode tasks.
pub fn start_high_priority_executor() -> SendSpawner {
    interrupt::UART4.set_priority(Priority::P6);
    EXECUTOR_HIGH.start(interrupt::UART4)
}

#[interrupt]
unsafe fn UART4() {
    // SAFETY: UART4 is reserved as the executor's software interrupt (REQ-0047);
    // nothing else enables or pends it, and on_interrupt is called only here.
    unsafe { EXECUTOR_HIGH.on_interrupt() }
}

#[exception]
unsafe fn HardFault(_frame: &ExceptionFrame) -> ! {
    // REQ-0045: on a hard fault, reset immediately.
    cortex_m::peripheral::SCB::sys_reset()
}
```

**WRONG:** register access from the application crate (bypasses the HAL, needs the PAC, and hides a hardware assignment from review)

```rust
embassy_stm32::pac::GPIOB.bsrr().write(|w| w.set_bs(8, true));
```

### 5. Concurrency: Embassy Primary, RTIC Alternative

**Embassy is the default model.**

- Use `#[embassy_executor::task]` tasks on the thread-mode executor.
- Use `embassy-sync` bounded `Channel`s and mutexes for communication, and `embassy-time` timers and tickers for periodic work.
- Use an `InterruptExecutor` (in the BSP) when a REQ needs priority preemption.
- Every channel, queue, and pool has a fixed capacity that a REQ justifies. No unbounded buffers.
- Interrupt and executor priorities come from requirements, never from defaults.

**Choose RTIC instead** when a REQ needs compile-time Stack Resource Policy analysis (deadlock-free, bounded-blocking resource sharing) or schedulability evidence derived from static priorities. Record the choice in `DECISION_LOG.md`. Do not mix RTIC and Embassy executors in one binary unless a REQ authorizes it.

**Shared state** uses `critical-section` (`critical_section::Mutex<RefCell<T>>`), `embassy-sync` primitives, or RTIC resources. **`static mut` is prohibited.**

File: `firmware/app/src/main.rs`

```rust
// Implements: REQ-0042@r2; baseline: BASELINE-0003; firmware entry, task wiring, watchdog
#![no_std]
#![no_main]
#![deny(unsafe_code)]

mod tasks;

use defmt_rtt as _;
#[cfg(feature = "panic-dev")]
use panic_probe as _;
#[cfg(feature = "panic-prod")]
use panic_reset as _;

use embassy_executor::Spawner;
use embassy_stm32::{i2c::I2c, wdg::IndependentWatchdog};

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    // REQ-0042: TMP117 on I2C1, SCL = PB8, SDA = PB9 (Logic Gatekeeper pin table).
    let i2c = I2c::new_blocking(p.I2C1, p.PB8, p.PB9, Default::default());
    // REQ-0043: 2 s independent watchdog.
    let mut wdg = IndependentWatchdog::new(p.IWDG, 2_000_000);
    wdg.unleash();
    // Pool size 1, spawned once: a SpawnError here is a wiring bug and panics.
    spawner.spawn(defmt::unwrap!(tasks::sample(i2c)));
    let _control = bsp::start_high_priority_executor();
    tasks::supervise(wdg).await
}
```

File: `firmware/app/src/tasks.rs`

```rust
// Implements: REQ-0042@r2; baseline: BASELINE-0003; sampling and supervision tasks
use embassy_stm32::i2c::{I2c, Master};
use embassy_stm32::{mode::Blocking, peripherals::IWDG, wdg::IndependentWatchdog};
use embassy_sync::{blocking_mutex::raw::CriticalSectionRawMutex, channel::Channel};
use embassy_time::{Duration, Ticker};
use sensor_core::Tmp117;
// Bounded (REQ-0042): capacity 4; the sender waits while it is full.
static READINGS: Channel<CriticalSectionRawMutex, i32, 4> = Channel::new();

#[embassy_executor::task]
pub async fn sample(i2c: I2c<'static, Blocking, Master>) {
    let mut sensor = Tmp117::new(i2c, 0x48);
    let mut ticker = Ticker::every(Duration::from_millis(1000));
    loop {
        match sensor.read_millicelsius() {
            Ok(mc) => READINGS.send(mc).await,
            Err(e) => defmt::warn!("TMP117 read failed: {}", e),
        }
        ticker.next().await;
    }
}

pub async fn supervise(mut wdg: IndependentWatchdog<'static, IWDG>) -> ! {
    let mut recent: heapless::HistoryBuf<i32, 8> = heapless::HistoryBuf::new();
    loop {
        // A stalled sampler stops the pets, so the IWDG resets the MCU (REQ-0043).
        recent.write(READINGS.receive().await);
        wdg.pet();
    }
}
```

**Fragment:** RTIC 2 application module (needs the `thumbv8main-backend` feature, `embassy-stm32`'s `unstable-pac` feature, and the crate-level attributes from `main.rs`)

```rust
// Implements: REQ-0050@r1; baseline: BASELINE-0003; RTIC alternative (SRP resource sharing)
#[rtic::app(device = embassy_stm32::pac, peripherals = false, dispatchers = [UART4])]
mod app {
    #[shared]
    struct Shared {
        setpoint: i32,
    }

    #[local]
    struct Local {}

    #[init]
    fn init(_cx: init::Context) -> (Shared, Local) {
        (Shared { setpoint: 25_000 }, Local {})
    }

    // Static priorities come from REQ-0050; SRP makes `lock` deadlock-free and bounded.
    #[task(priority = 2, shared = [setpoint])]
    async fn control(mut cx: control::Context) {
        let _sp = cx.shared.setpoint.lock(|sp| *sp);
    }
}
```

**WRONG:** `static mut` shared between a task and an interrupt (data race, and needs `unsafe` in application code)

```rust
static mut LAST_READING: i32 = 0;
```

### 6. Memory

- **No heap by default.** Use `heapless` collections and `static_cell::StaticCell` for `'static` initialization. A `#[global_allocator]` is permitted only when a REQ authorizes it, and its size counts against the RAM budget.
- **Stack-overflow protection per architecture:**
  - **Cortex-M:** link through `flip-link`, which places the stack at the bottom of RAM so that an overflow faults instead of corrupting `.data`/`.bss`. Armv8-M parts may additionally use the hardware stack limit (cortex-m-rt `set-msplim`) when a REQ calls for it.
  - **RISC-V:** use PMP stack-guard regions or the guard the HAL provides.
  - **Fallback for any architecture:** stack painting (cortex-m-rt `paint-stack` feature, or an equivalent fill at startup) plus a high-water-mark check.
- **Budgets:** track flash, static RAM, and stack against the REQ limits on every L2 delivery (see Evidence Requirements), and record the numbers in the Build Manifest.

### 7. Panic, Faults, Watchdog, and Logging

- **The panic handler is an explicit, requirement-driven choice.** A typical split is `panic-probe` for development builds and reset (`panic-reset`) or halt-and-record for production, selected by features as shown above. If the requirements do not specify production panic behaviour, halt.
- **Watchdog use and fault handlers follow the requirements:** the watchdog type, timeout, and pet point, plus the HardFault, MemManage, and BusFault (or RISC-V trap) behaviour. Pet the watchdog only from the place that proves forward progress.
- **Logging defaults to `defmt`** (with `defmt-rtt`) unless a REQ specifies another transport. Never log secrets or key material.

### 8. Testability

Structure the code so that each tier is possible. Test-suite design remains with `test-designer` (if installed; otherwise halt and ask). The Build Agent does not verify its own artifacts: it runs the build evidence below, and independent verification of the tests and results belongs to `red-team-verifier` or an independent reviewer.

1. **Host-side unit tests** for the host-testable crate, using mock HALs (`embedded-hal-mock` with its `eh1` feature).
2. **On-target tests** with `embedded-test` or `defmt-test`, run through `probe-rs`.
3. **Hardware-in-the-loop (HIL) tests** on the designated rig, for timing, electrical, and multi-device behaviour that the host cannot exercise.

File: `sensor-core/Cargo.toml`

```toml
# Implements: REQ-0044@r1; baseline: BASELINE-0003; host-testable driver crate
[package]
name = "sensor-core"
version = "0.1.0"
edition.workspace = true
rust-version.workspace = true
license.workspace = true

[dependencies]
embedded-hal.workspace = true

[dev-dependencies]
embedded-hal-mock = { version = "0.11.1", default-features = false, features = ["eh1"] }

[lints]
workspace = true
```

File: `sensor-core/src/tests.rs`, which shows the test structure the crate enables. The suite content itself comes from `test-designer` (if installed; otherwise halt and ask).

```rust
// Implements: REQ-0044@r1; baseline: BASELINE-0003; host test structure with a mock bus
use super::*;
use embedded_hal_mock::eh1::i2c::{Mock, Transaction};

#[test]
fn reads_and_scales_temperature() {
    // 0x0C80 = 3200 LSB = 25.000 °C.
    let reply = vec![0x0C, 0x80];
    let expected = [Transaction::write_read(0x48, vec![REG_TEMP], reply)];
    let mut bus = Mock::new(&expected);
    let mut sensor = Tmp117::new(bus.clone(), 0x48);
    assert_eq!(sensor.read_millicelsius().ok(), Some(25_000));
    bus.done();
}

#[test]
fn conversion_covers_full_range() {
    assert_eq!(raw_to_millicelsius(i16::MIN), -256_000);
    assert_eq!(raw_to_millicelsius(i16::MAX), 255_992);
}
```

On-target test targets need their own linker setup, because `cargo:rustc-link-arg-bins` does not reach test targets. Declare `[[test]] name = "on_target"` with `harness = false`, add `embedded-test` as a dev-dependency, and emit `cargo:rustc-link-arg-tests=` lines for `-Tlink.x`, `-Tdefmt.x`, and `-Tembedded-test.x` from `build.rs`. The test crate also needs a defmt global logger (`use defmt_rtt as _;`); without it, linking fails with undefined `_defmt_acquire`/`_defmt_write` symbols. `embedded-test` provides its own panic handler, so the test crate must not link `panic-probe` or `panic-reset` (that fails with `E0152`, duplicate `panic_impl`).

**Fragment:** on-target test harness in `firmware/app/tests/on_target.rs` (needs the linker setup, defmt logger, and no-panic-crate rule above)

```rust
// Implements: REQ-0044@r1; baseline: BASELINE-0003; on-target test structure
#![no_std]
#![no_main]

use defmt_rtt as _; // global logger; embedded-test supplies the panic handler

#[cfg(test)]
#[embedded_test::tests]
mod tests {
    #[init]
    fn init() -> embassy_stm32::Peripherals {
        embassy_stm32::init(Default::default())
    }

    #[test]
    fn sensor_answers(p: embassy_stm32::Peripherals) {
        let i2c = embassy_stm32::i2c::I2c::new_blocking(p.I2C1, p.PB8, p.PB9, Default::default());
        assert!(sensor_core::Tmp117::new(i2c, 0x48).read_millicelsius().is_ok());
    }
}
```

---

## Evidence Requirements

Evidence is defined for risk classes L0, L1, and L2 only. Each class inherits all base evidence from **build-agent** and **agile-v-core**. The additions below give a command or named report and an objective threshold. `<triple>` is the firmware target, and `<crate>` is the host-testable crate. Record every command and its result in the Build Manifest evidence summary.

### L0: Exploratory

Base evidence applies; no Rust-specific additions.

### L1: Routine

Base evidence applies, plus:

| Evidence | Command | Threshold |
|---|---|---|
| Formatting | `cargo fmt --all --check` | exit 0 |
| Lints | `cargo clippy --target <triple> -- -D warnings` on firmware crates, and `cargo clippy -p <crate> --all-targets -- -D warnings` on the host-testable crate, adding `--target <host-triple>` when that crate is inside the target-configured directory | exit 0 for both |
| Warning-free release build | `cargo build --release --target <triple> --config 'target.<triple>.rustflags=["-D","warnings"]'` | exit 0 |
| Host tests | `cargo test -p <crate>` when the crate is outside the target-configured directory; `cargo test -p <crate> --target <host-triple>` (for example `x86_64-unknown-linux-gnu`, `aarch64-apple-darwin`) when it is inside | exit 0 |

Notes on the release build:

- Cargo appends `--config` arrays to the existing target rustflags. Do not use `RUSTFLAGS`, because it replaces them.
- Write triples containing a dot **unquoted** in the key, for example `target.thumbv8m.main-none-eabihf.rustflags`. The same applies to `[target.<triple>]` tables in `.cargo/config.toml`. Cargo silently ignores quoted dotted triples. A `cfg(...)` target table is an accepted alternative.
- Firmware projects put rustflags under `target.<triple>`, never under `[build] rustflags`, which target rustflags override.

### L2: Production

All L1 evidence applies, plus:

| Evidence | Command or method | Threshold |
|---|---|---|
| Flash/RAM report | `cargo size --release -- -A` (sum `.vector_table`, `.text`, `.rodata`, `.data` for flash; `.data`, `.bss`, `.uninit` for static RAM) | each ≤ the REQ limit, or ≤ 90% of device capacity if no REQ limit |
| Stack usage | runtime stack painting (cortex-m-rt `paint-stack`), run the designated worst-case scenario, then read the stack region with `probe-rs read --chip <chip> b32 <stack-low-address> <words>` and count untouched paint words. Alternative: static analysis with `cargo +nightly-YYYY-MM-DD call-stack --bin <bin> --target <triple> > call-graph.dot` (`cargo-call-stack`; nightly date recorded in the manifest), taking the worst case as the reset entry's maximum plus the maximum of each preempting priority level | margin ≥ 20% of the stack region, unless a REQ specifies otherwise. A static result reported as unbounded (recursion, unresolved indirect calls) does not count; use painting instead |
| On-target / HIL tests | `cargo test --test <on-target-test>` through the `probe-rs` runner, plus the HIL rig report | 100% of designated tests pass |
| Dependency policy | `cargo deny check` | exit 0 |
| `unsafe` inventory | table of each `unsafe` block (file:line, authorization reference, SAFETY comment), reconciled with `grep -rn 'unsafe' --include='*.rs'` over project-owned crates | 0 unlisted blocks |
| Host-crate coverage | `cargo llvm-cov -p <crate> --fail-under-lines 80`, adding `--target <host-triple>` when the crate is inside the target-configured directory | exit 0 |
| SBOM | `cargo cyclonedx` | file produced; path in manifest |
| MSRV | `cargo +<rust-version> check --release --target <triple>` | exit 0 |

A pinned nightly toolchain may be used only for static stack analysis. Record its date and purpose in the manifest. `cargo-call-stack` always builds with `-Zbuild-std`, so the pinned nightly needs the `rust-src` component (`rustup component add rust-src --toolchain nightly-YYYY-MM-DD`). Its last tested nightly predates edition 2024, so it may not build an edition 2024 project at all. If the tool fails to build or run, or reports any entry as unbounded, stack painting is mandatory and its result is the evidence. Product firmware always builds on the pinned stable toolchain.

### Any Risk Class: Timing Deadlines

When a REQ states a timing deadline, record measured worst-case timing on target (for example DWT cycle counter, GPIO toggle with a logic analyzer, or a HIL timestamp) for the designated worst-case scenario. Threshold: measured worst case ≤ deadline.

L3 and L4 are out of scope for this skill. A task classified L3 or L4 is a halt condition (Halt Conditions, item 1; see Risk Boundary) and is escalated, not built.

---

## Halt Conditions

Halt and do not emit when any condition below holds.

**Inherited from build-agent:** ambiguous requirement; requirement not approved and baselined; missing typed lineage or REQ link; physical constraint violation; conflict with the approved baseline; materially AI-influenced task without an AI_RUN_MANIFEST.

**Embedded Rust additions:**

1. **Risk boundary:** an L3 or L4 task, a functional-safety-standard task, or trust-boundary work (secure boot, OTA update or bootloader, key storage, cryptographic protocol implementation). Escalate to the human plus `safety-engineer`, `threat-modeler`, or `agile-v-compliance` as applicable, if installed.
2. **Unassigned hardware:** a pin, clock, peripheral, interrupt priority, or power assignment that is in neither the baselined requirements nor the Logic Gatekeeper constraints.
3. **Memory map or budget:** an undefined memory map, or flash or RAM over budget.
4. **Stack margin** below the L2 threshold (20% unless a REQ specifies otherwise).
5. **Unauthorized `unsafe`**, or `unsafe` outside project-owned PAC, HAL, BSP, or startup code.
6. **Direct register access** in application code.
7. **Heap use** without authorization.
8. **Production panic behaviour** not specified in the requirements.
9. **No maintained HAL** implementing `embedded-hal` 1.0 for the target. Writing one requires authorization.
10. **Missed timing deadline:** a measured worst case exceeds a REQ deadline.
11. **Dependency version mismatch** with the Ecosystem Versions table that changes an API a pattern uses (major version, or minor version for 0.x crates).
12. **Out-of-scope toolchain or target:** a required MSRV below 1.85, or an Xtensa, `*-espidf`, or `*-uefi` target.

**Halt protocol:**

1. Stop synthesis immediately.
2. Flag the halt in the Build Manifest evidence summary.
3. Present the specific issue to the human (for example, "REQ-0042 names I2C1 but no SCL/SDA pins; the Logic Gatekeeper pin table has no entry").
4. Wait for the human resolution (clarified REQ, change request, authorization, or decision record).
5. Resume only after the gate clears.

---

## Context Engineering

The **build-agent** context rules apply, plus these:

1. **Never load bulk hardware sources** into context: PAC sources, SVD files, full datasheets or reference manuals, ELF/bin/hex artifacts, or `target/`. Reference them by path.
2. **Extract hardware facts to files.** Extract only the facts the artifact needs (memory sizes, pin and alternate-function table, clock limits, interrupt numbers) into `.agile-v/REQUIREMENTS.md` or a constraints file it references, and read them from disk.
3. **One driver or module per sub-agent context.** Pass the trait interface and the relevant REQs, not the whole firmware.
4. **Reference crates by name and version** (docs.rs) instead of pasting dependency documentation. Read `Cargo.toml` from disk; never load the full `Cargo.lock`.

---

## Output Format

Emit a Build Manifest with every delivery. List paths, never file contents.

**C1 format:** `ART-XXXX | REQ-XXXX@revision | baseline-id | implements | path | notes`

**C2+ format:** `ART-XXXX.N | REQ-XXXX@revision | baseline-id | implements | path | CYCLE | CR | notes`

Example:

```text
BUILD_MANIFEST
Cycle: C1
Task: TASK-0012 TMP117 sampling firmware
Risk class: L2
Toolchain: rustc 1.96.0 (pinned); edition 2024; MSRV (rust-version) 1.88; nightly: none
Target: thumbv8m.main-none-eabihf; MCU STM32H563ZI; RAM 640 KiB; flash 2 MiB

ART-0001 | REQ-0040@r1 | BASELINE-0003 | implements | firmware/.cargo/config.toml | runner, flip-link
ART-0002 | REQ-0041@r1 | BASELINE-0003 | implements | firmware/app/memory.x | values from REQ-0041
ART-0003 | REQ-0044@r1 | BASELINE-0003 | implements | sensor-core/src/lib.rs | embedded-hal 1.0 I2c
ART-0004 | REQ-0042@r2 | BASELINE-0003 | implements | firmware/app/src/tasks.rs | channel cap 4
ART-0005 | REQ-0046@r1 | BASELINE-0003 | implements | firmware/bsp/src/startup.rs | unsafe: DEC-0007

Evidence summary:
- cargo fmt --all --check: exit 0
- cargo clippy --target thumbv8m.main-none-eabihf -- -D warnings: exit 0
- cargo clippy -p sensor-core --all-targets -- -D warnings: exit 0
- cargo build --release (target rustflags -D warnings via --config): exit 0
- cargo test -p sensor-core: exit 0 (2 passed)
- cargo size --release: flash 9.1 KiB / 2048 KiB; static RAM 1.4 KiB / 640 KiB
- stack paint: <peak> of <region> bytes used; margin <n>% (threshold 20%)
- on-target/HIL: <n>/<n> designated tests pass
- cargo deny check: exit 0
- unsafe inventory: 1 block listed, 0 unlisted
- cargo llvm-cov -p sensor-core --fail-under-lines 80: exit 0
- SBOM: <path to the CycloneDX file>
- cargo +1.88 check --release --target thumbv8m.main-none-eabihf: exit 0
- Version table comparison: no mismatches
AI_RUN_MANIFEST: .agile-v/aibom/TASK-0012/AI_RUN_MANIFEST.yaml
```

Header fields are mandatory:

- cycle, task, and risk class;
- toolchain: rustc version, edition, MSRV, and any nightly used plus its purpose;
- target: triple, MCU, RAM, and flash.

The evidence summary lists each evidence command that was run, with its result, and the AI_RUN_MANIFEST path. Each artifact file starts with the traceability header in its own comment syntax: `// Implements:` (Rust), `# Implements:` (TOML), `/* Implements: */` (linker scripts).

---

## When to Use

**Project types:**

- Bare-metal MCU firmware in Rust: Cortex-M (`thumbv6m`, `thumbv7m`, `thumbv7em`, `thumbv8m`) and RISC-V (`riscv32*`, including ESP32-C and ESP32-H series).
- Embassy-based async firmware, RTIC applications, and BSP and startup crates.
- Host-testable crates, drivers, and protocol crates that belong to a firmware project under the routing rule below.
- Risk classes L0 to L2 only.

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

### Auto-Trigger Hints

**File patterns:**

- `**/memory.x`
- `**/Embed.toml`
- `**/.cargo/config.toml` whose `[build] target` is a bare-metal triple
- `**/build.rs` that emits linker arguments referencing `link.x`, `memory.x`, or `defmt.x`

`#![no_main]` alone is not a trigger (see the routing rule).

**Task keywords:**

- "Embassy", "RTIC", "embedded-hal", "probe-rs", "defmt", "no_main"
- "firmware", "bare-metal", "MCU", "microcontroller", "BSP", "HAL", "interrupt", "watchdog"
- MCU families: "STM32", "nRF52", "nRF53", "RP2040", "RP2350", "SAMD", "ATSAME", "LPC", "i.MX RT", "ESP32-C3", "ESP32-C6", "ESP32-H2", "CH32V", "GD32"
