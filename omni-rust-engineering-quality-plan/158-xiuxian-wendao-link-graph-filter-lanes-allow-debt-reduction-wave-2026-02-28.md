# 158. Xiuxian-Wendao Link-Graph Filter Lanes Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph/search_filters/*`
  - `tests/test_link_graph/tree_scope_filters/*`

## Why This Wave

After wave `157`, the next dense suppression cluster was the LinkGraph filter
test lanes. These files were medium-to-small and shared common setup patterns,
making them suitable for fast, suppression-free convergence.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from `search_filters` lane:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_link_to_and_linked_by.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_mentions_orphan_tagless_and_missing_backlink.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_accepts_ppr_options.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_with_distance.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_options_validate_rejects_invalid_related_ppr_alpha.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_temporal_filters_and_sorting.rs`

2. Removed file-level `#![allow(...)]` from `tree_scope_filters` lane:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_allows_verified_for_graph_filters.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_semantic_graph_filters.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_structural_scope.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_mixed_scope_collapse_toggle_changes_output_shape.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_options_deserialize_accepts_tree_filters.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_options_validate_rejects_invalid_tree_filters.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_section_scope_respects_per_doc_cap.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_hops_limit_section_expansion.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_level_and_min_words_filters.rs`

3. Fixed newly exposed `clippy::implicit_clone` warnings in filter tests:
   - replaced `map_err(|e| e.to_string())?` with `map_err(|e| e.clone())?`
   - this kept behavior identical while satisfying pedantic lint rules.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`
- Nextest run ID: `52fb7078-c7e8-4492-839b-d9f368165f74`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `116`
  - After this wave: `99`
  - Net reduction: `17` files

## Engineering Outcome

- Both LinkGraph filter test lanes now run under strict clippy without
  file-level lint suppression.
- Remaining suppression debt is pushed deeper into fewer areas, improving
  subsequent-wave targeting precision.

## Next Slice

- Continue with `test_link_graph_agentic/*` and `test_link_graph_saliency/*`
  leaf test files where suppression markers remain concentrated.
