# Wendao PageIndex Rust Core Implementation

## Summary

Implemented the first Rust-native `PageIndex` path inside `xiuxian-wendao` and integrated it into the existing `LinkGraphIndex` lifecycle.

This change adds:
- typed `PageIndex` records with `Arc<str>` text ownership,
- a dedicated `link_graph/page_index/` algorithm module for stack-based folding and deterministic thinning,
- `trees_by_doc` storage inside `LinkGraphIndex`,
- cache snapshot persistence for page trees,
- incremental refresh maintenance for page trees,
- focused integration tests for hierarchy building, headingless documents, thinning, and refresh updates.

## Design Notes

### Why this shape

The implementation keeps responsibilities separated:
- `link_graph/page_index/` contains the pure tree-building and thinning logic.
- `link_graph/index/page_indices.rs` owns index-local orchestration and accessors.
- `link_graph/parser/sections.rs` now emits the extra metadata required for stable page trees (`heading_title`, `line_start`, `line_end`).

This avoids bloating `LinkGraphIndex` with fold logic and keeps `mod.rs` interface-only.

### Storage choice

`PageIndexNode` uses `Arc<str>` for in-memory text ownership, but cache snapshots use explicit snapshot conversion structs instead of enabling global serde `rc` support. This keeps the cache format explicit and decouples in-memory representation from serialized payload shape.

### Thinning policy

The default thinning threshold was set to `12` tokens after test feedback. A larger default collapsed too many realistic parent headings. The lower threshold still folds truly tiny parent sections while preserving ordinary heading structure.

## Files Added

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/page_index/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/page_index/builder.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/page_index/thinning.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `.agent/execplans/wendao-page-index-rust-core.md`

## Files Updated

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/assemble.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/cache/snapshot.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/refresh/mutate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/parser/sections.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/resources/xiuxian_wendao.link_graph.valkey_cache_snapshot.v1.schema.json`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

### Tier 2

1. `cargo check -p xiuxian-wendao`
   - Result: passed.

### Tier 3

1. `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
   - Result: passed.
   - Fixes required during the run:
     - converted `PageIndex` doc comments to use backticks,
     - removed an `if-not-else` pedantic warning in `page_index/builder.rs`,
     - replaced a too-many-arguments helper in `parser/sections.rs` with a small `SectionCursor` struct,
     - derived `Copy` for `SectionCursor` to satisfy `needless_pass_by_value`.

2. `cargo nextest run -p xiuxian-wendao --test test_link_graph page_index`
   - Result: passed.
   - Coverage from this run:
     - hierarchy construction,
     - headingless document fallback,
     - deterministic thinning,
     - incremental refresh maintenance.

## Follow-Up Opportunities

1. Add a CLI or API surface that can emit `PageIndex` trees for debugging and Wendao UI consumers.
2. Revisit thinning strategy once retrieval consumers start using `PageIndex` directly; token threshold and merge formatting may need domain tuning.
3. Consider whether search/ranking should consume `PageIndex` nodes directly instead of rebuilding passage-level heuristics from flat sections.
