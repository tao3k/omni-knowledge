# 514. Xiuxian Wendao Link-Graph Refs Contract Migration

Date: 2026-03-08

## Scope

This shard records the Wave 14 completion for the link-graph entity-reference surface in `xiuxian-wendao`.

The migrated external binary is:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs.rs`

The retired wrapper tree is:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/`

The committed fixture tree used as the source of truth is:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph_refs/`

## What Changed

- Completed the standalone snapshot contract coverage for extraction, formatting, parsing, search, and statistics.
- Corrected the stats fixture shape so `LinkGraphRefStats.by_type` matches real serde output.
- Kept the expected snapshots in committed JSON fixtures under `tests/fixtures/link_graph_refs/`.

## Key Discovery

`LinkGraphRefStats.by_type` serializes as an ordered array of `(String, usize)` tuples, not as a JSON object map. The expected stats fixture was updated accordingly.

Updated fixture:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph_refs/stats/expected/result.json`

## Validation

Executed and passed:

- `CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-refs cargo check -p xiuxian-wendao --test test_link_graph_refs --message-format short`
- `CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-refs NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_refs`
- `CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-refs cargo clippy -p xiuxian-wendao --test test_link_graph_refs -- -W clippy::too_many_lines`

Observed non-blocking warning noise remained limited to pre-existing runtime-config dead-code warnings in:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`

## Result

The link-graph refs surface now runs as a direct snapshot-contract binary, and the expected stats serialization is aligned with the real contract emitted by the current code.
