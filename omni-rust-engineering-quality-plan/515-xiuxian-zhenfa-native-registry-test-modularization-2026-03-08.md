# 515. Xiuxian Zhenfa Native Registry Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the large mixed-concern
`test_native_registry.rs` integration test in `xiuxian-zhenfa`.

## Why This Change Was Needed

The original file bundled support fixtures and multiple orchestrator behaviors in
one 540-line entrypoint:

- plain dispatch,
- registry snapshot behavior,
- cache interactions,
- mutation-lock behavior,
- audit events,
- signal routing.

That violated the workspace modularization standard and made the test surface
hard to extend without further file growth.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry.rs` so
it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for shared test tools, cache fixtures, lock fixtures, and sinks,
- `dispatch.rs` for direct dispatch behavior,
- `registry.rs` for definition snapshot behavior,
- `cache.rs` for read-through and write-back cache behavior,
- `mutation.rs` for mutation-lock coverage,
- `audit.rs` for failure audit emission,
- `signals.rs` for signal sink routing.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-zhenfa --tests
cargo nextest run -p xiuxian-zhenfa --no-fail-fast
cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-zhenfa --tests` passed.
- `cargo nextest run -p xiuxian-zhenfa --no-fail-fast` passed (`32 passed, 0 skipped`).
- `cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines` passed.

Notes:

- `cargo check` emitted unrelated `missing-docs` warnings for
  `packages/rust/crates/xiuxian-zhenfa/tests/test_client.rs` and
  `packages/rust/crates/xiuxian-zhenfa/tests/test_gateway.rs`; both files were
  intentionally left untouched because they already contain in-flight user
  edits.

## Architectural Takeaways

- Shared fixtures belong in a support module rather than inside the launcher.
- Registry, cache, mutation, audit, and signal behaviors should each have their
  own module so future changes stay localized.
- Large integration suites should be decomposed by behavior, not only by file
  length.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/mod.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/support.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/dispatch.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/registry.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/cache.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/mutation.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/audit.rs`
- `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry/signals.rs`
