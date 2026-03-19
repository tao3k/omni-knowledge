# 459. Xiuxian Wendao Registry Contract Deduplication

Date: 2026-03-07

## Scope

This shard records the removal of duplicate top-level test binaries for the
embedded Wendao registry behavior family.

## Why This Change Was Needed

The repository still kept parallel non-contract test entries for two behaviors
that were already fully represented by fixture-backed contract binaries:

- embedded dynamic semantic URI discovery,
- embedded resource registry indexing and validation.

The duplicate files were:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry.rs`

Their assertions were strict subsets of the existing contract surfaces:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_contracts.rs`

Keeping both sets of binaries added maintenance cost without adding new
coverage.

## What Changed

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry.rs`

No contract expansion was required.

The remaining contract binaries already covered the removed assertions and more:

- empty and prefixed dynamic-discovery queries,
- exact config-id lookup,
- semantic carryover lookup,
- alias query forms,
- valid embedded registry indexing,
- semantic URI lifting for relative wikilinks,
- missing linked-resource validation.

## Architectural Takeaways

- For registry-style APIs, fixture-backed contract binaries are a better single
  source of truth than several narrow top-level assertion files.
- Exact-match, alias, and error-path semantics belong together in one explicit
  contract payload.
- Deleting duplicate binaries is correct when the contract surface already
  captures a superset of the removed behavior.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery.rs` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_wendao_dynamic_discovery_contracts --test test_wendao_resource_registry_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ... --tests` completed cleanly.
- The targeted `cargo nextest run ...` passed (`2 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Authoritative contract binaries:
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery_contracts.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_contracts.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/459-xiuxian-wendao-registry-contract-deduplication-2026-03-07.md`
