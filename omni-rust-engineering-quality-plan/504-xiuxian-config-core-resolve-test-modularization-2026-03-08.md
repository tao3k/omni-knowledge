# 504. Xiuxian Config Core Resolve Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_resolve.rs`
integration test in `xiuxian-config-core`.

## Why This Change Was Needed

The original file bundled several distinct resolver contracts into one
entrypoint:

- orphan-file handling,
- array merge strategies,
- dotted namespace projection,
- empty-namespace root merging.

These concerns belong to the same resolver feature area, but they should not
remain mixed in one top-level file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-config-core/tests/test_resolve.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-config-core/tests/test_resolve/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for temp-workspace creation, fixture writing, and merge helpers,
- `orphan.rs` for orphan-file behavior,
- `arrays.rs` for array merge strategy coverage,
- `namespace.rs` for dotted and root-namespace merge behavior.

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

- Resolver tests should isolate orphan behavior, merge strategies, and
  namespace projection into separate modules.
- Temp-workspace and fixture-writing logic belong in dedicated support helpers,
  not inline in every test body.
- Thin launchers keep configuration-resolution suites aligned with the same test
  structure used across the Rust workspace.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-config-core/tests/test_resolve.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_resolve/mod.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_resolve/support.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_resolve/orphan.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_resolve/arrays.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_resolve/namespace.rs`
