# 493. Xiuxian Wendao Unified Symbol Unit Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`unified_symbol_unit.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original unified-symbol test file mixed several separate contracts in one
entrypoint:

- unified symbol construction,
- unified/project/external search behavior,
- external usage tracking,
- stats aggregation.

These contracts are distinct enough that they should not remain packed into one
implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/` with
focused modules:

- `mod.rs` for the module graph only,
- `symbol.rs` for symbol construction behavior,
- `search.rs` for unified/project/external search behavior,
- `usage.rs` for external usage tracking,
- `stats.rs` for aggregated index statistics.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test unified_symbol_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test unified_symbol_unit --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Model construction, search behavior, usage tracking, and stats aggregation
  should each have their own module even in compact unit-style suites.
- Thin entrypoints keep unified-symbol coverage aligned with the package-wide
  test structure and make future additions cheaper.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/symbol.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/search.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/usage.rs`
- `packages/rust/crates/xiuxian-wendao/tests/unified_symbol_unit/stats.rs`
