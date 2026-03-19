# 507. Xiuxian Wendao Graph Surface Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the remaining `LinkGraph` graph-surface suite in `xiuxian-wendao` from internal direct tests to one fixture-backed snapshot contract binary, and the deletion of the obsolete internal `test_link_graph` tree.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/`

The final remaining modules inside that tree were:

- `graph_navigation.rs`
- `graph_navigation_fixture_support.rs`
- `markdown_attachments.rs`
- `markdown_attachments_fixture_support.rs`

They have now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_graph_surface_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_graph_surface_contract_support.rs`

## Why This Change Was Needed

Once search, planning, lifecycle, and quantum suites were externalized, the old `test_link_graph` module tree existed only to host the last graph-navigation and attachment tests. Keeping the tree around after that point added no value.

Externalizing the final graph-surface batch allowed the entire internal `test_link_graph` harness to be removed.

## What Changed

### 1) Added one graph-surface contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_graph_surface_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_graph_surface_contract_support.rs`

The new binary covers:

- neighbors, related results, metadata, and TOC snapshots,
- PPR diagnostics for single-seed and multi-seed related searches,
- Markdown link extraction behavior,
- attachment search filtering by kind and extension.

### 2) Removed the remaining internal `test_link_graph` tree

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/`

Why this matters:

- there is no longer a duplicated internal-vs-external structure for `LinkGraph` tests,
- all migrated `LinkGraph` behavior now lives in explicit external contract binaries,
- the repository reflects the user's requirement to delete superseded tests immediately after migration.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-graph-surface-contracts cargo check -p xiuxian-wendao --test test_link_graph_graph_surface_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-graph-surface-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_graph_surface_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-graph-surface-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_graph_surface_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed cleanly.
- `cargo nextest run` passed (`8 passed, 0 skipped`).
- `cargo clippy` completed cleanly.
