# 505. Xiuxian Config Core Cache Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of `test_cache.rs` in `xiuxian-config-core`.

## Why This Change Was Needed

The file had only two tests, but it still mixed two distinct cache surfaces in
one top-level implementation file:

- cache invalidation after underlying file changes,
- concurrent read stability.

To avoid mixed old/new structure inside the crate, the cache suite was normalized
at the same time as the other config-core test entrypoints.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-config-core/tests/test_cache.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-config-core/tests/test_cache/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for temp-workspace creation, fixture writing, spec construction,
  and strict-mode projection,
- `invalidation.rs` for cache invalidation behavior,
- `concurrency.rs` for concurrent read stability.

### Signature Fix

The new shared `skills_spec()` helper now returns `ConfigCascadeSpec<'static>`
explicitly, which matches the borrowed nature of the spec and avoids a missing-
lifetime compile error.

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

- Small suites should still be normalized when they would otherwise leave a
  crate with mixed structural conventions.
- Shared cached-config setup should live behind one support boundary, especially
  when concurrency and invalidation tests need the same fixture logic.
- Lifetime-bearing helper return types should be made explicit instead of
  relying on impossible elision.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-config-core/tests/test_cache.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_cache/mod.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_cache/support.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_cache/invalidation.rs`
- `packages/rust/crates/xiuxian-config-core/tests/test_cache/concurrency.rs`
