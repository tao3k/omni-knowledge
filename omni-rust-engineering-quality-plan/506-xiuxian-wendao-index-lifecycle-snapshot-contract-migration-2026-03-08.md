# 506. Xiuxian Wendao Index Lifecycle Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` index-lifecycle suite in `xiuxian-wendao` from internal direct tests to one fixture-backed snapshot contract binary.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_index_lifecycle_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_index_lifecycle_contract_support.rs`

## Why This Change Was Needed

This family all described how the index is created, cached, refreshed, and structurally exposed over time. Keeping them inside separate internal modules scattered one lifecycle story across eight files.

The migration unified that story into one contract binary and added the only missing snapshot surface: page-index parent ids.

## What Changed

### 1) Added one lifecycle contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_index_lifecycle_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_index_lifecycle_contract_support.rs`

The new binary covers:

- directory scoping and skill metadata promotion,
- cache reuse, invalidation, and saliency seeding,
- page-index hierarchy, thinning, semantic documents, refresh, and parent ids,
- incremental refresh sequencing and threshold modes.

### 2) Added the missing page-index parent-id snapshot

Added:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/parent_ids.json`

This captured the actual anchor-parent topology produced by the page-index tree.

### 3) Removed the superseded internal lifecycle modules

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-index-lifecycle-contracts cargo check -p xiuxian-wendao --test test_link_graph_index_lifecycle_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-index-lifecycle-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_index_lifecycle_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-index-lifecycle-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_index_lifecycle_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed cleanly.
- `cargo nextest run` passed (`17 passed, 0 skipped`).
- `cargo clippy` completed cleanly.
