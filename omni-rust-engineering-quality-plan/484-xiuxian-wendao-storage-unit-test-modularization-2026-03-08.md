# 484. Xiuxian Wendao Storage Unit Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `storage_unit.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original storage integration test file mixed multiple behavior surfaces into
one entrypoint:

- storage construction,
- CRUD lifecycle behavior,
- text search and stats aggregation,
- vector search ranking,
- shared Valkey gating and test vectorization support.

These concerns share a storage boundary, but they should not remain bundled in
one implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/storage_unit.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/storage_unit/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for Valkey gating, storage construction, and vector helper logic,
- `creation.rs` for storage constructor coverage,
- `lifecycle.rs` for upsert/delete/clear behavior,
- `search.rs` for text search and stats aggregation,
- `vector.rs` for semantic vector ranking behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test storage_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test storage_unit --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Storage integration tests should separate constructor, lifecycle, search, and
  ranking behavior into focused modules.
- Shared helpers such as Valkey detection, storage creation, and deterministic
  vector generation belong in a local support module.
- Even relatively small async integration suites benefit from a stable thin
  entrypoint plus feature-local modules.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/storage_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/creation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/lifecycle.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/search.rs`
- `packages/rust/crates/xiuxian-wendao/tests/storage_unit/vector.rs`
