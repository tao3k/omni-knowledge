# 505. Xiuxian Wendao Search Behavior Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` search-behavior suite in `xiuxian-wendao` from internal direct tests to one fixture-backed snapshot contract binary.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters_fixture_support.rs`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_behavior_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_behavior_contract_support.rs`

## Why This Change Was Needed

After planning was externalized, the next strong migration target was search behavior because match strategies and filter operators are both user-visible query-surface contracts.

The family already had stable fixture roots for:

- `search_match_strategies`
- `search_filters`

The remaining work was structural plus one missing validation snapshot.

## What Changed

### 1) Added one search-behavior contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_behavior_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_behavior_contract_support.rs`

The new binary covers:

- path-fuzzy parsing and retrieval,
- exact and regex retrieval,
- link/mention/orphan/tagless/backlink filters,
- related-distance and related-PPR behavior,
- temporal sorting,
- invalid related-PPR alpha validation.

### 2) Added the missing validation fixture and updated it to current error text

Added and finalized:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/validate_rejects_invalid_related_ppr_alpha/expected/result.json`

This snapshot now reflects the current schema-validation prefix emitted by Wendao.

### 3) Removed the superseded internal module tree for search behavior

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-search-behavior-contracts cargo check -p xiuxian-wendao --test test_link_graph_search_behavior_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-search-behavior-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_search_behavior_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-search-behavior-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_search_behavior_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed cleanly.
- `cargo nextest run` passed (`12 passed, 0 skipped`).
- `cargo clippy` completed cleanly.
