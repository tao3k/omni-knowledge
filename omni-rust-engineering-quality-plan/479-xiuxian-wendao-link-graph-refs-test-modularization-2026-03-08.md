# 479. Xiuxian Wendao Link Graph Refs Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_link_graph_refs.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test binary grouped several distinct reference-surface behaviors in
one top-level file:

- entity reference extraction,
- formatting helpers (`to_wikilink`, `to_tag`),
- reference parsing and validation,
- note-to-entity search,
- reference statistics.

Those concerns belong to the same feature area, but they are different enough
that keeping them in one file made the suite harder to scan and extend.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/`
with focused modules:

- `mod.rs` for the module graph only,
- `extraction.rs` for entity reference extraction coverage,
- `formatting.rs` for `LinkGraphEntityRef` rendering behavior,
- `parsing.rs` for entity reference parsing and validation,
- `search.rs` for note lookup by entity reference,
- `stats.rs` for aggregate reference statistics.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph_refs --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_refs --no-fail-fast`
  passed (`15 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Even small utility-facing integration suites should be split when they cover
  multiple contract surfaces.
- Pure formatting and parsing checks should not live beside search and aggregate
  statistics behavior in the same implementation file.
- Thin entrypoints keep the crate-level test binary stable while allowing
  feature-local modules to evolve independently.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/extraction.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/formatting.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/parsing.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/search.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_refs/stats.rs`
