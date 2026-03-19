# 483. Xiuxian Wendao KG Cache Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_kg_cache.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original file mixed several cache-oriented behaviors into one top-level
entrypoint:

- shared cache setup and serialization fixture creation,
- cache miss/hit lifecycle behavior,
- explicit invalidation behavior,
- nonexistent scope handling,
- normalized path reuse.

These behaviors all belong to the KG cache surface, but they should be split so
lifecycle assertions do not sit beside path-behavior contracts in one file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for the shared lock, Valkey gate, fixture creation, and cached
  load helper,
- `lifecycle.rs` for miss/hit and invalidation coverage,
- `path_behavior.rs` for nonexistent scope and path normalization behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_kg_cache --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_kg_cache --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Stateful cache integration tests should isolate lifecycle and path contracts
  into separate modules even when they share common setup logic.
- Shared serialization fixtures and synchronization primitives should stay in a
  dedicated support boundary rather than remaining embedded in the test binary.
- Thin entrypoints keep Valkey-gated test suites maintainable without changing
  the public test binary name.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache/lifecycle.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache/path_behavior.rs`
