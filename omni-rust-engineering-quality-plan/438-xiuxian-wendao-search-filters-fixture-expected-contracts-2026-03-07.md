# 438. Xiuxian Wendao Search-Filters Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the Wendao LinkGraph test-architecture slice that migrates
the inline-corpus `search_filters` scenarios to `tests/fixtures/.../input` plus
`tests/fixtures/.../expected` contracts.

## Why This Change Was Needed

The `search_filters` directory still carried several older tests that built
corpora inline with `TempDir` plus `write_file(...)`. Those tests protected
useful behavior, but the setup and the contract were split apart.

## What Changed

### 1) Added a search-filters fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters_fixture_support.rs`

This module owns:

- search-filter scenario materialization,
- stable ordered path projection for filter results,
- expected-fixture assertion dispatch.

### 2) Migrated the inline-corpus search-filter scenarios

Updated files:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_link_to_and_linked_by.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_mentions_orphan_tagless_and_missing_backlink.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_accepts_ppr_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_with_distance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_temporal_filters_and_sorting.rs`

New scenarios now live under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/link_to_and_linked_by/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/mentions_orphan_tagless_missing_backlink/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/related_accepts_ppr_options/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/related_with_distance/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/temporal_filters_and_sorting/...`

What the contracts now cover:

- `link_to` vs `linked_by` filtering,
- mention-based filtering,
- orphan / tagless / missing-backlink filtering,
- related filtering with and without PPR tuning,
- temporal filtering and deterministic sort order.

## Architectural Takeaways

- Path lists are the right contract surface for these filter tests; exact hit
  scores add noise and no additional confidence.
- Directory-style tests still benefit from fixture support modules rooted in the
  parent lane rather than generic helpers.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_link_to_and_linked_by.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_mentions_orphan_tagless_and_missing_backlink.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_accepts_ppr_options.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_filters_related_with_distance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_filters/link_graph_search_temporal_filters_and_sorting.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_filters/*/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph search_filters
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph search_filters`
  passed (`7 passed, 77 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/437-xiuxian-wendao-semantic-policy-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/438-xiuxian-wendao-search-filters-fixture-expected-contracts-2026-03-07.md`
