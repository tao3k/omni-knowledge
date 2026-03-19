# 391. Interface-only modularization wave and daochang package rename (2026-03-05)

## Scope

- Continue structural sink-down: keep crate root modules as interface layers.
- Apply the same rule to additional crates beyond the logging rollout.
- Rename Rust runtime package from `omni-agent` to `xiuxian-daochang`.

## Implementation

1. `xiuxian-event` interface-only split:
   - `src/lib.rs` now declares modules and re-exports only.
   - Moved event model into `src/event.rs`.
   - Moved bus implementation into `src/bus.rs`.
   - Moved global singleton helpers into `src/global.rs`.
   - Moved source constants into `src/sources.rs`.
   - Public API preserved via re-exports (`OmniEvent`, `EventBus`, `GLOBAL_BUS`, `publish`, `emit`, `subscribe`).

2. `omni-security` interface-only split:
   - `src/lib.rs` now only wires modules and exports.
   - Secret scanner model and logic moved to `src/scanner.rs`.
   - Permission gatekeeper moved to `src/permissions.rs`.
   - Existing sandbox module remains as `src/sandbox.rs`.
   - Public API preserved (`SecurityViolation`, `SecretScanner`, `PermissionGatekeeper`, sandbox exports).

3. Package rename (`omni-agent` -> `xiuxian-daochang`):
   - `packages/rust/crates/omni-agent/Cargo.toml`
     - `package.name = "xiuxian-daochang"`
     - `default-run = "xiuxian-daochang"`
     - added `[lib] name = "omni_agent"` to keep crate import path stable during migration.
   - CLI command surface updated:
     - `packages/rust/crates/omni-agent/src/cli.rs`
       - `#[command(name = "xiuxian-daochang")]`
   - Logging target alignment:
     - `packages/rust/crates/omni-agent/src/main.rs`
       - `init_from_cli("xiuxian_daochang", ...)`
     - `packages/rust/crates/omni-agent/src/bin/webhook_dedup_probe.rs`
       - `init_from_cli("xiuxian_daochang", ...)`
       - default dedup prefix changed to `xiuxian-daochang:test:dedup:probe`.

## Verification

- `xiuxian-event`:
  - `cargo check -p xiuxian-event` -> pass
  - `cargo clippy -p xiuxian-event -- -W clippy::too_many_lines` -> pass
  - `cargo nextest run -p xiuxian-event` -> `5 passed`, `0 failed`

- `omni-security`:
  - `cargo check -p omni-security` -> pass
  - `cargo clippy -p omni-security -- -W clippy::too_many_lines` -> pass
  - `cargo nextest run -p omni-security` -> `15 passed`, `0 failed`

- `xiuxian-daochang` rename validity:
  - `cargo check -p xiuxian-daochang` -> pass
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` -> pass
  - `cargo nextest run -p xiuxian-daochang --test gateway_validation` -> `3 passed`, `0 failed`
  - `cargo pkgid -p xiuxian-daochang` -> pass
  - `cargo pkgid -p omni-agent` -> fails as expected (package no longer exists)
  - `Cargo.lock` package record now contains only `name = "xiuxian-daochang"` for this crate.

## Outcome

- Additional crates now follow the same root-module discipline (`lib.rs` as API surface, implementation in child modules).
- The runtime package naming has moved to `xiuxian-daochang` while preserving short-term library import compatibility.
- The migration remains buildable and test-validated for touched lanes.
