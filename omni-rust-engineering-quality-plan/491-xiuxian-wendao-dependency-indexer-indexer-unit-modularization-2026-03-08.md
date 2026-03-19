# 491. Xiuxian Wendao Dependency Indexer Indexer Unit Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`dependency_indexer_indexer_unit.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original file mixed separate core-indexer concerns into one entrypoint:

- indexer construction,
- default config assertions,
- performance budget validation.

These are related, but the benchmark budget helpers and performance contract
should not live beside basic constructor checks in one implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for benchmark slack calculation, workspace-root discovery, and
  temporary config writing,
- `config.rs` for constructor and default-config assertions,
- `performance.rs` for build budget validation.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test dependency_indexer_indexer_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test dependency_indexer_indexer_unit --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Performance-budget logic should be isolated from constructor checks so the
  benchmark contract remains visible and self-contained.
- Environment-sensitive timing helpers belong in a dedicated support module.
- Thin entrypoints keep core-indexer integration tests aligned with the rest of
  the package test structure.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/config.rs`
- `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/performance.rs`
