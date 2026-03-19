# 488. Xiuxian Wendao Dependency Debug Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_dependency_debug.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original file mixed separate dependency-indexer integration behaviors in one
entrypoint:

- indexer creation from explicit config,
- config loading assertions,
- build result structure validation,
- empty-index search behavior.

These contracts are related, but they exercise different integration surfaces
and should not remain bundled in one implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for temp-root creation, config writing, manifest writing, and
  empty-indexer setup,
- `config.rs` for config creation and loading coverage,
- `build.rs` for dependency index build result behavior,
- `search.rs` for empty-index search assertions.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_dependency_debug --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_dependency_debug --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Config, build, and search integration checks should live in separate modules
  even when they share the same temp-root fixture machinery.
- Support helpers should own the repetitive project-root and config-file setup
  so the test bodies stay focused on contract assertions.
- Thin entrypoints keep dependency-debug coverage aligned with the rest of the
  package test structure.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/config.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/build.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_dependency_debug/search.rs`
