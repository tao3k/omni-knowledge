# 439. Xiuxian Wendao Tree-Scope Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the Wendao LinkGraph test-architecture slice that migrates
the remaining inline-corpus `tree_scope_filters` scenarios onto fixture-backed
expected JSON contracts.

## Why This Change Was Needed

The `tree_scope_filters` directory still contained the densest concentration of
section-scope inline corpus setup after the earlier migration wave. Those tests
covered high-value behavior, but their contracts were still hidden inside local
assertion logic.

## What Changed

### 1) Added a tree-scope fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_fixture_support.rs`

This module owns:

- tree-scope scenario materialization,
- section-hit outline projection,
- per-path count projection,
- ordered section-label projection,
- expected-fixture assertion dispatch.

### 2) Migrated the inline-corpus tree-scope scenarios

Updated files:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_allows_verified_for_graph_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_semantic_graph_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_structural_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_mixed_scope_collapse_toggle_changes_output_shape.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_section_scope_respects_per_doc_cap.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_hops_limit_section_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_level_and_min_words_filters.rs`

New scenarios now live under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/verified_edges_keep_graph_filters/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/structural_edges_restrict_graph_filters/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/semantic_edges_restrict_section_scope/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/mixed_scope_collapse_toggle/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/section_scope_per_doc_cap/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/tree_hops_limit_section_expansion/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/tree_level_and_min_words/...`

What the contracts now cover:

- edge-type gating for graph filters,
- edge-type gating for section scope,
- mixed-scope collapse behavior,
- per-document section caps,
- tree-hop-limited section expansion,
- heading-depth and minimum-word filters.

## Architectural Takeaways

- Tree-scope behavior is a contract surface, not just internal ranking detail.
- Section-oriented tests need stable, readable output shapes such as section
  labels and per-path counts rather than procedural assertions.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/*.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/*/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph tree_scope_filters
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph tree_scope_filters`
  passed (`9 passed, 75 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/438-xiuxian-wendao-search-filters-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/439-xiuxian-wendao-tree-scope-fixture-expected-contracts-2026-03-07.md`
