# 462. Xiuxian Wendao Knowledge Mod Import Bucket Removal

Date: 2026-03-07

## Scope

This shard records the cleanup of the `test_knowledge` suite so that its
`mod.rs` no longer acts as an ambient import bucket.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_knowledge/mod.rs` previously
imported multiple domain types and then relied on child files pulling them via
`use super::*;`.

That structure obscured which types each child test actually depended on and
made `mod.rs` carry implementation responsibility instead of just module
boundary responsibility.

## What Changed

### 1) Restored `mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/mod.rs`

The module root now only declares child modules.

### 2) Replaced `use super::*;` with explicit per-file imports

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_category_equality.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_category_variants.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_clone.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_creation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_default_category.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_equality.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_tag_operations.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_with_metadata.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_with_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_stats_default.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_builder.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_creation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_default.rs`

Each child file now imports the exact knowledge-model types it needs from
`xiuxian_wendao`.

## Architectural Takeaways

- Even small, data-model-focused test suites benefit from explicit imports.
- Import buckets in `mod.rs` create hidden coupling even when there are no
  shared helper functions.
- The absence of shared helpers does not justify keeping `mod.rs` as a context
  bag. Module roots should still stay interface-only.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_category_equality.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_category_variants.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_clone.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_creation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_default_category.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_equality.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_tag_operations.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_with_metadata.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_entry_with_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/knowledge_stats_default.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_builder.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_creation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/search_query_default.rs`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_knowledge --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The full `test_knowledge` binary passed (`13 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/462-xiuxian-wendao-knowledge-mod-import-bucket-removal-2026-03-07.md`
