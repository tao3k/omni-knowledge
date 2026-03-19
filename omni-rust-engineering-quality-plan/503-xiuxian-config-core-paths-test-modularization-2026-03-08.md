# 503. Xiuxian Config Core Paths Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the compact but mixed `paths_unit.rs`
integration test in `xiuxian-config-core`.

## Why This Change Was Needed

Even though the file was small, it mixed two separate path-resolution surfaces in
one top-level file:

- default project-home resolution,
- environment-driven relative and absolute path resolution.

The repository rule is to split by concern rather than line count, so this file
still qualified for cleanup.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-config-core/tests/paths_unit.rs` so it now
acts as an explicit integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-config-core/tests/paths_unit/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for the shared project-root fixture,
- `defaults.rs` for default home-path behavior,
- `env_resolution.rs` for relative and absolute environment resolution.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-config-core --tests
cargo nextest run -p xiuxian-config-core --no-fail-fast
cargo clippy -p xiuxian-config-core -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-config-core --tests` passed.
- `cargo nextest run -p xiuxian-config-core --no-fail-fast` passed (`12 passed, 0 skipped`).
- `cargo clippy -p xiuxian-config-core -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Path helper regression tests still benefit from separating default behavior
  from environment-driven behavior.
- Small shared fixtures belong in a local support module instead of repeating
  literal paths in multiple test bodies.
- Compact test files should still follow the same explicit-launcher structure as
  larger suites.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-config-core/tests/paths_unit.rs`
- `packages/rust/crates/xiuxian-config-core/tests/paths_unit/mod.rs`
- `packages/rust/crates/xiuxian-config-core/tests/paths_unit/support.rs`
- `packages/rust/crates/xiuxian-config-core/tests/paths_unit/defaults.rs`
- `packages/rust/crates/xiuxian-config-core/tests/paths_unit/env_resolution.rs`
