# 497. Xiuxian Wendao Unified Symbol Py Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `unified_symbol_py.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original support-oriented file mixed several distinct unified-symbol
contracts in one entrypoint:

- unified search behavior,
- external usage tracking,
- stats aggregation.

These contracts are separate enough that they should not stay bundled in one
small top-level file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py/` with
focused modules:

- `mod.rs` for the module graph only,
- `search.rs` for unified search behavior,
- `usage.rs` for external usage tracking,
- `stats.rs` for aggregated stats behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test unified_symbol_py --no-fail-fast
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test unified_symbol_py --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Even compact support-oriented suites should separate search, usage, and stats
  contracts into focused modules.
- Thin entrypoints keep Python-facing support tests aligned with the broader
  package test structure.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py/search.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py/usage.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_py/stats.rs`
