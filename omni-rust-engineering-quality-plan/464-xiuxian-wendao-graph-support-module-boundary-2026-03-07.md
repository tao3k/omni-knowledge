# 464. Xiuxian Wendao Graph Support Module Boundary

Date: 2026-03-07

## Scope

This shard records the cleanup of the `test_graph` family so its module root is
interface-only and shared environment checks move into a support module.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_graph/mod.rs` previously mixed
module declarations with ambient imports and a `has_valkey()` helper. Child
files depended on those via `use super::*;`, which hid each test file's real
dependencies.

## What Changed

### 1) Restored `mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_graph/mod.rs`

The module root now only declares child modules.

### 2) Extracted the shared Valkey gate into `support.rs`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_graph/support.rs`

This support module now owns `has_valkey()`.

### 3) Localized domain imports inside each child test file

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_graph/entity_relation_crud.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/entity_search_scoring.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/graph_persistence.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/graph_traversal.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/skill_registration.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/tool_relevance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/valkey_persistence.rs`

Each child now explicitly imports the `xiuxian_wendao` graph types it uses.
Only the Valkey persistence tests import `super::support::has_valkey`.

## Architectural Takeaways

- Environment-gating helpers such as `has_valkey()` belong in `support.rs`, not
  in `mod.rs`.
- Data-model and graph tests benefit from explicit per-file imports just as much
  as CLI and benchmark suites do.
- A support module can be very small; its value is keeping the module boundary
  clean, not the helper count.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/entity_relation_crud.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/entity_search_scoring.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/graph_persistence.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/graph_traversal.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/skill_registration.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/tool_relevance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_graph/valkey_persistence.rs`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_graph --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The full `test_graph` binary passed (`25 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New support module:
  - `packages/rust/crates/xiuxian-wendao/tests/test_graph/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/464-xiuxian-wendao-graph-support-module-boundary-2026-03-07.md`
