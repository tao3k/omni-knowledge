# 490. Xiuxian Wendao Dependency Indexer Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_dependency_indexer.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file mixed several distinct dependency-indexer contracts in
one entrypoint:

- symbol-index search behavior,
- dependency config loading from workspace resources,
- external dependency model construction.

Those concerns are related but should not remain bundled in one top-level file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for workspace-root resolution and reusable symbol-index setup,
- `symbol_index.rs` for symbol search behavior,
- `config.rs` for config loading assertions,
- `dependency.rs` for external dependency model construction.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_dependency_indexer --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_dependency_indexer --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Search, config, and model-construction checks should live in separate modules
  even when they target the same dependency-indexer feature area.
- Workspace-root and symbol-index fixtures belong in a support boundary rather
  than being redefined inside one top-level test file.
- Thin entrypoints preserve a stable test binary while the dependency-indexer
  coverage grows in focused modules.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/symbol_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/config.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_indexer/dependency.rs`
