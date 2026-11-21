# Change Log

## CR-0001: REQ-0209 dotted-triple quoting rule is wrong

| Field | Value |
|---|---|
| Cycle | C1 |
| Affected | REQ-0209@r5 (BASELINE-0001) |
| Raised | 2026-11-21, by the build agent (ART-0002 halt) |
| Reproduced by | The orchestrator, on cargo 1.96.0, in a scratch crate with an injected unused variable |
| Status | APPROVED 2026-11-21 (Wes Cravens); baselined in BASELINE-0002 |

**Defect.** REQ-0209@r5 requires dotted triples to be quoted in the `--config` key (`target."thumbv8m.main-none-eabihf".rustflags`). Cargo 1.96.0 silently ignores the quoted key. The unquoted form (`target.thumbv8m.main-none-eabihf.rustflags`) applies the flags. `.cargo/config.toml` behaves the same way: a `[target."…"]` table is ignored, and `[target.thumbv8m.main-none-eabihf]` applies. As baselined, the L1 "build with `-D warnings`" evidence would pass vacuously on dotted triples.

**Origin.** The r5 resolution of Logic Gatekeeper finding LG-301 adopted the recommendation without an empirical check.

**Proposed change (REQ-0209 r6).** Replace "Triples containing a dot must be quoted in the key (e.g. `target.\"thumbv8m.main-none-eabihf\".rustflags`)" with: "Triples containing a dot are written **unquoted** in the key (e.g. `target.thumbv8m.main-none-eabihf.rustflags`), and the same applies to `[target.<triple>]` tables in `.cargo/config.toml`. Cargo silently ignores quoted dotted triples. A `cfg(...)` target table is an accepted alternative." Lineage adds CR-0001.

**Impact.**
- REQ-0209 → r6. All other REQs are unchanged, so they are carried forward at r5.
- New baseline: BASELINE-0002.
- ART-0002 (`build-agent-embedded-rust/SKILL.md`) gets a one-line correction, becoming ART-0002.2.
- ART-0001 is not affected, because its examples use triples without dots.
- No test or verification has been executed yet.

| Evidence | Result |
|---|---|
| quoted `--config` key | build passes (flags ignored) |
| unquoted `--config` key | build fails (flags applied) |
| quoted `[target."…"]` table | build passes (ignored) |
| unquoted `[target.…]` table | build fails (applied) |
