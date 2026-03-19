# 460. Xiuxian Wendao Link Graph Saliency Support Module Boundary

Date: 2026-03-07

## Scope

This shard records the modular cleanup of the `test_link_graph_saliency` test
family so that its module root returns to interface-only responsibility.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`
previously mixed two concerns:

- declaring the test modules,
- hosting shared Valkey helpers, constants, and imports for all child tests.

That structure violated the repository rule that `mod.rs` should act as an
interface boundary rather than an ambient implementation bucket.

## What Changed

### 1) Restored `mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`

The module root now only declares child modules.

### 2) Extracted shared helpers into a dedicated support module

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/support.rs`

This file now owns:

- `TEST_VALKEY_URL`,
- `unique_prefix()`,
- `valkey_connection()`,
- `clear_prefix()`.

### 3) Localized test dependencies inside each child file

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_activation_boosts_score.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_clamps_bounds.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_store_auto_repairs_invalid_payload.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_and_get_with_valkey.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_updates_inbound_edge_zset.rs`

Each test file now explicitly imports the `xiuxian_wendao` items it uses and,
when needed, imports shared helpers from `super::support`.

## Architectural Takeaways

- `mod.rs` should not become an import sink or helper bag just because several
  sibling tests need the same setup logic.
- A dedicated `support.rs` is the right place for shared test infrastructure in
  a growing test module family.
- Explicit per-file imports make dependencies visible and reduce hidden coupling
  between sibling test modules.
- This pattern is a reusable modernization step for other test families still
  relying on `use super::*;` plus helper-heavy `mod.rs` files.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_activation_boosts_score.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_clamps_bounds.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_store_auto_repairs_invalid_payload.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_and_get_with_valkey.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_updates_inbound_edge_zset.rs`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_link_graph_saliency --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The full `test_link_graph_saliency` binary passed (`5 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New reusable support module:
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/460-xiuxian-wendao-link-graph-saliency-support-module-boundary-2026-03-07.md`
