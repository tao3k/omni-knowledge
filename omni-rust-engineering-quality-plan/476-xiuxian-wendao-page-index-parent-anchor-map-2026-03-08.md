# 476. Xiuxian Wendao Page Index Parent Anchor Map

Date: 2026-03-08

## Scope

This shard records the page-index anchor enhancement that adds explicit parent
anchors to `PageIndexNode` and a parent lookup index to `LinkGraphIndex`.

## Why This Change Was Needed

The page-index tree previously encoded hierarchy only through nested child
vectors. That was sufficient for tree rendering, but it left two gaps:

- page-index nodes did not carry a physical parent anchor on the node itself;
- `LinkGraphIndex` had no direct parent lookup map for anchor-based semantic
  uplink and trace reconstruction.

This made hierarchical anchor traversal dependent on recursive tree searches
instead of an explicit parent topology.

## What Changed

### Page Index Node Model

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/page_index.rs`
so `PageIndexNode` now includes:

- `pub parent_id: Option<String>`

### Builder Injection

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/page_index/builder.rs`
so parent anchors are assigned when a node closes into its parent.

### Index Parent Map

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs` to add:

- `node_parent_map: HashMap<String, Option<String>>`

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
so the index now:

- synchronizes `parent_id` fields across stored trees,
- rebuilds `node_parent_map` when trees are built or restored,
- removes stale parent entries during incremental page-index replacement,
- reconstructs semantic anchor paths from the parent map instead of relying
  solely on recursive path search.

### Snapshot Compatibility

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/cache/snapshot.rs`
so page-index snapshots now persist `parent_id`, while keeping backward
compatibility with `#[serde(default)]` for older snapshots.

### Build Constructor Alignment

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/assemble.rs`
to initialize the new `node_parent_map` field.

## Test Coverage

Added a direct structural assertion in:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`

The new test verifies:

- root nodes have `parent_id == None`;
- child nodes carry their immediate parent anchor id.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast`
  passed (`85 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed after
  fixing a small `clippy::assigning_clones` warning in the new parent-map sync
  path.

## Architectural Takeaways

- Hierarchical document trees should expose explicit parent anchors in the node
  model, not only implicit nesting.
- Anchor-oriented traversal benefits from an index-side parent map even when the
  canonical source of truth remains the tree.
- Snapshot restoration must rebuild or preserve topology metadata explicitly;
  otherwise cached trees drift from freshly built trees.
- Incremental page-index rebuilds must remove stale topology entries before
  inserting refreshed trees.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/page_index/builder.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/assemble.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/cache/snapshot.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
