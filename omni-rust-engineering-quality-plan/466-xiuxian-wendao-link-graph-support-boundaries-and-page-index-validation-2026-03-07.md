# 466. Xiuxian Wendao Link Graph Support Boundaries And Page Index Validation

Date: 2026-03-07

## Scope

This shard records the structural cleanup of the remaining `test_link_graph`
family and the two link-graph benchmark binaries, with special attention to
keeping `mod.rs` interface-only and moving shared helpers into support modules.

## Why This Change Was Needed

The remaining link-graph test families still had two kinds of structural debt:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/mod.rs`

First, those module roots mixed declarations with helpers, constants, or ambient
imports. Second, many child files still depended on `use super::*;`, which hid
real dependencies behind a parent-module bucket.

## What Changed

### 1) Restored link-graph and benchmark module roots to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/mod.rs`

These files now declare child modules only.

### 2) Added focused support modules with narrow responsibilities

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/support.rs`

The support split follows three rules:

- root support owns only local helper functions such as `write_file`,
  `sort_term`, and Valkey cache-key cleanup;
- lane support modules re-export only stable domain types or local helpers;
- fixture-contract helpers remain imported directly from their sibling fixture
  modules instead of being re-exported through another support layer.

### 3) Localized root-level link-graph imports

Updated representative root-level test files:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`

Each file now imports fixture helpers and domain types explicitly.

### 4) Localized nested lane imports down to the file level

Updated all child files under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/`

The final state is zero-warning `cargo check` on the crate test target, with no
remaining `use super::*;` in these families.

### 5) Modularized link-graph benchmark support without polluting `mod.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/link_graph_hybrid_batch_latency_on_2k_fixture.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/link_graph_related_ppr_latency_on_10k_fixture.rs`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/support.rs`

The benchmark fixture generators, latency-budget helpers, percentile helpers,
shared constants, and env parsing now live in support modules.

## Architectural Takeaways

- A support module should not become a second `mod.rs` bucket. Keep it focused
  on stable shared helpers and local domain types.
- Re-exporting sibling fixture-contract helpers through nested support layers is
  brittle. Import fixture helpers directly from their defining module instead.
- Zero-warning `cargo check` is a useful structural convergence signal before
  running broader nextest or clippy gates.
- Ignored benchmark binaries still benefit from explicit support modules because
  compile-time boundaries matter even when runtime execution is opt-in.

## Validation Evidence

Executed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast
cargo nextest run -p xiuxian-wendao --test test_link_graph_hybrid_benchmark --test test_link_graph_ppr_benchmark --no-tests pass --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` completed with zero warnings.
- The focused benchmark binaries compiled and reported `0 passed, 2 skipped`.
- `cargo clippy ...` completed cleanly.
- `test_link_graph` converged structurally but still has four failing page-index
  snapshot tests on the current branch (`80 passed, 4 failed`):
  - `test_link_graph::page_index::test_link_graph_page_index_builds_hierarchy_and_line_ranges`
  - `test_link_graph::page_index::test_link_graph_page_index_exports_semantic_documents`
  - `test_link_graph::page_index::test_link_graph_page_index_thins_small_parent_sections`
  - `test_link_graph::page_index::test_link_graph_page_index_refresh_updates_incremental_tree`

## Validation Boundary Note

These four failures are content-contract mismatches in the page-index snapshot
lane. This cleanup changed module boundaries and imports, but it did not attempt
to rebaseline page-index fixtures or alter page-index runtime semantics.

## Artifacts and Notes

- New support modules:
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/466-xiuxian-wendao-link-graph-support-boundaries-and-page-index-validation-2026-03-07.md`
