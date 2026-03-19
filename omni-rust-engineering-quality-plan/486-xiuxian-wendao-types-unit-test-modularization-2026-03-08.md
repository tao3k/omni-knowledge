# 486. Xiuxian Wendao Types Unit Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `types_unit.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file mixed two separate type-surface contracts in one
entrypoint:

- `KnowledgeEntry` construction and builder behavior,
- `KnowledgeSearchQuery` builder behavior.

These contracts are small, but they evolve independently and should not remain
packed into the same top-level implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/types_unit.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/types_unit/` with focused
modules:

- `mod.rs` for the module graph only,
- `knowledge_entry.rs` for `KnowledgeEntry` construction and tag/source helpers,
- `query.rs` for `KnowledgeSearchQuery` builder behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test types_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test types_unit --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Even compact type-surface suites should separate entry and query contracts
  when they exercise different builder APIs.
- Thin entrypoints keep type tests aligned with the same package-level test
  structure used across the rest of `xiuxian-wendao`.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/types_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/types_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/types_unit/knowledge_entry.rs`
- `packages/rust/crates/xiuxian-wendao/tests/types_unit/query.rs`
