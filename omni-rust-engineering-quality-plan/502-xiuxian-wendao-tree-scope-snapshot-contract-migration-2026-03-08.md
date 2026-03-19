# 502. Xiuxian Wendao Tree Scope Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` tree-scope filter suite in `xiuxian-wendao` from direct assertion tests to fixture-backed snapshot contracts.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_fixture_support.rs`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_tree_scope_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_tree_scope_contract_support.rs`

## Why This Change Was Needed

After the parser-wave audit, tree-scope filters were the next strongest migration target because they already behaved like a contract suite in practice:

- scenario-specific fixture inputs already existed,
- most scenarios already stored expected JSON under `tests/fixtures/link_graph/tree_scope_filters/`,
- the remaining work was mainly structural: move the suite into an external contract binary and replace the last inline assertions.

## What Changed

### 1) Added a dedicated tree-scope contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_tree_scope_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_tree_scope_contract_support.rs`

The new binary covers the full tree-scope behavior family:

- filter deserialization,
- invalid option validation,
- edge-type-restricted graph filtering,
- mixed-scope collapse behavior,
- per-doc caps,
- tree-hop-limited section expansion,
- heading-depth and minimum-word filtering.

Why this matters:

- the whole tree-scope surface is now reviewed through one cohesive contract binary,
- behavior and schema validation live side by side,
- source modules no longer carry a scattered test tree for the same feature family.

### 2) Reused existing expected fixtures and added missing ones

Existing expected fixtures were preserved for the search-behavior scenarios under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/*/expected/result.json`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/deserialize_accepts_tree_filters/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/validate_rejects_invalid_tree_filters/expected/result.json`

Why this matters:

- the new contract binary does not discard earlier fixture work,
- option-level cases now have the same snapshot discipline as search-behavior cases,
- the suite remains easy to diff and extend.

### 3) Removed the superseded internal module tree

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- the migrated tree-scope suite no longer coexists with its older direct-assertion form,
- support helpers now live under `tests/support/`, which matches the external-contract layout,
- the `test_link_graph` module graph is simpler.

## Architectural Takeaways

### Reusing old fixtures is better than re-inventing them

The tree-scope suite already had good fixture roots. The migration succeeded by externalizing structure, not by replacing stable assets unnecessarily.

### Validation-only cases deserve snapshot contracts too

Deserialization and validation cases are often left behind in direct-assertion style. Moving them into the same fixture namespace keeps the suite conceptually whole.

### Feature-family migrations scale better than single-file churn

Tree-scope filters were migrated as one coherent capability family. That is easier to audit and sign off than a long sequence of tiny test edits.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_tree_scope_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_tree_scope_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/deserialize_accepts_tree_filters/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/tree_scope_filters/validate_rejects_invalid_tree_filters/expected/result.json`
- `.cache/codex/execplans/wendao-test-snapshot-migration-wave-2-tree-scope.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_allows_verified_for_graph_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_semantic_graph_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_edge_type_filter_restricts_structural_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_mixed_scope_collapse_toggle_changes_output_shape.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_options_deserialize_accepts_tree_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_options_validate_rejects_invalid_tree_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_section_scope_respects_per_doc_cap.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_hops_limit_section_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/link_graph_search_tree_level_and_min_words_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-tree-scope-contracts cargo check -p xiuxian-wendao --test test_link_graph_tree_scope_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-tree-scope-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_tree_scope_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-tree-scope-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_tree_scope_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_tree_scope_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_tree_scope_contracts` passed (`9 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_tree_scope_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_tree_scope_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_tree_scope_contract_support.rs`
- Wave plan: `.cache/codex/execplans/wendao-test-snapshot-migration-wave-2-tree-scope.md`
- Next cohesive target: `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/` and `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/`
