# 454. Xiuxian Wendao Refresh Mode Fixture Deduplication

Date: 2026-03-07

## Scope

This shard records the consolidation of link-graph refresh mode coverage onto
one fixture-backed lane in `xiuxian-wendao/tests`.

## Why This Change Was Needed

The repository had two parallel refresh-mode test surfaces:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/link_graph_refresh_unit.rs`

They covered overlapping incremental refresh behavior, but only one of them
used the modern fixture-first test structure. Keeping both introduced three
problems:

- duplicated behavior coverage,
- two different testing styles for the same feature,
- reduced signal density because the legacy unit file asserted mode enums only
  and did not preserve the richer result and stats contract.

## What Changed

### 1) Expanded the fixture-backed refresh threshold contract

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/expected/result.json`

The `threshold_modes` scenario now captures all three refresh outcomes through a
single fixture-backed contract:

- `noop`
- `full`
- `delta`

### 2) Removed the redundant legacy test binary

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/link_graph_refresh_unit.rs`

Refresh-mode semantics now live in exactly one lane: the `test_link_graph`
fixture suite.

### 3) Preserved a meaningful behavioral distinction

During migration, the updated fixture contract exposed a real behavioral
difference:

- `full` refresh produced `links_in_graph = 2`
- `delta` refresh produced `links_in_graph = 1`

That distinction would have remained invisible in the removed legacy test file,
which only asserted the enum mode and not the resulting graph state.

## Architectural Takeaways

- When a modern fixture lane and a legacy hand-built lane cover the same
  feature, the correct move is usually consolidation, not coexistence.
- Fixture contracts preserve behavioral details that plain enum assertions do
  not. Here, the fixture made the `delta` versus `full` graph-state difference
  explicit.
- Support modules should export reusable projection helpers instead of forcing
  individual tests to restate match logic locally.
- Deleting redundant tests is a quality improvement when coverage becomes more
  expressive, not less.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/refresh/threshold_modes/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/link_graph_refresh_unit.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_link_graph test_link_graph_refresh_incremental_updates_and_deletes_notes test_link_graph_refresh_incremental_with_threshold_modes --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The targeted `cargo nextest run ...` passed (`2 passed, 82 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Related fixture support:
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh_fixture_support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/454-xiuxian-wendao-refresh-mode-fixture-deduplication-2026-03-07.md`
