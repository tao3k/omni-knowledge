# 492. Xiuxian Wendao Dep Indexer Py Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `dep_indexer_py.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original file mixed several support-type contracts into one entrypoint:

- config loading from workspace resources,
- external dependency model construction,
- symbol-index search behavior.

These behaviors should evolve independently and therefore should not stay in one
mixed top-level test file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for workspace-root resolution and reusable symbol-index setup,
- `config.rs` for config loading assertions,
- `dependency.rs` for external dependency support-type behavior,
- `symbol_index.rs` for symbol-index search assertions.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test dep_indexer_py --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test dep_indexer_py --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Even support-type compatibility tests should follow the same modularization
  rules as the primary Rust-facing suites.
- Shared workspace-root and symbol-index setup belongs in local support modules
  instead of repeating the same scaffolding in one entrypoint.
- Thin entrypoints keep Python-support integration tests consistent with the
  rest of the crate's test layout.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/config.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/dependency.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dep_indexer_py/symbol_index.rs`
