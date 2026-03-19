# 487. Xiuxian Wendao Dependency Indexer Symbols Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`dependency_indexer_symbols_unit.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file bundled two distinct contract surfaces into one
entrypoint:

- symbol extraction from fixture source files,
- symbol index search and serialization behavior.

These behaviors belong to the same dependency-indexer area, but they should not
remain mixed in one top-level implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for fixture-file writing and reusable symbol-index assembly,
- `extraction.rs` for Rust/Python symbol extraction coverage,
- `index.rs` for index search and serialize/deserialize behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test dependency_indexer_symbols_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test dependency_indexer_symbols_unit --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Extraction fixtures and index-state assertions should not share one top-level
  test file even when they target the same subsystem.
- Reusable fixture writing belongs in a support module instead of repeated
  temporary-file setup inside each test body.
- Thin entrypoints keep dependency-indexer test binaries stable while the
  behavior coverage grows in focused modules.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit/extraction.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_symbols_unit/index.rs`
