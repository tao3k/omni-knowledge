# 485. Xiuxian Wendao Entity Unit Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `entity_unit.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original entity test file mixed several distinct model contracts into one
entrypoint:

- entity construction defaults,
- entity builder methods,
- relation identifier construction,
- enum display formatting.

These behaviors all belong to the entity model surface, but they should not
remain bundled in one top-level file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/entity_unit.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/entity_unit/` with focused
modules:

- `mod.rs` for the module graph only,
- `entity.rs` for entity construction and builder behavior,
- `relation.rs` for relation id construction,
- `enums.rs` for `EntityType` and `RelationType` display contracts.

### Contract Correction

Adjusted the relation-id assertion to match the current `Relation::new`
implementation contract. The constructor lowercases values and replaces spaces
with underscores, but it does not rewrite `Omni-Dev-Fusion` into an unrelated
legacy brand slug. The test now asserts the actual normalized target fragment:
`omni-dev-fusion`.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test entity_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test entity_unit --no-fail-fast`
  passed (`5 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Small model-focused suites still benefit from separating entity, relation,
  and enum contracts into distinct modules.
- Tests should assert the live constructor contract, not stale branding or
  pre-rename assumptions.
- Thin entrypoints keep model-surface tests easy to extend without reopening a
  mixed file.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/entity_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/entity_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/entity_unit/entity.rs`
- `packages/rust/crates/xiuxian-wendao/tests/entity_unit/relation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/entity_unit/enums.rs`
