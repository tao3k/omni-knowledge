# 461. Xiuxian Wendao Link Graph Agentic Support Module Boundary

Date: 2026-03-07

## Scope

This shard records the modular cleanup of the `test_link_graph_agentic` test
family so that its module root no longer acts as a helper bucket.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/mod.rs`
previously mixed two concerns:

- declaring child test modules,
- hosting shared Valkey helpers, constants, and imports.

That structure hid dependencies behind `use super::*;` and violated the project
rule that `mod.rs` should remain interface-only.

## What Changed

### 1) Restored `mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/mod.rs`

The module root now only declares child modules.

### 2) Extracted shared Valkey helpers into `support.rs`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/support.rs`

This file now owns:

- `TEST_VALKEY_URL`,
- `unique_prefix()`,
- `valkey_connection()`,
- `clear_prefix()`.

### 3) Localized dependencies inside each child test file

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_promoted_with_audit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_rejects_invalid_transition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_rejects_invalid_payload.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_roundtrip.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_trims_stream_by_max_entries.rs`

Each child now imports the exact `xiuxian_wendao` items it uses and, when
needed, imports shared helpers from `super::support`.

## Architectural Takeaways

- `support.rs` is the correct home for shared test infrastructure in a growing
  module family.
- `use super::*;` usually signals hidden coupling between sibling tests and an
  overloaded module root.
- Localized imports make each test file independently readable and reduce the
  chance of accidental dependency drift.
- This is the same modernization pattern already applied to
  `test_link_graph_saliency` and should be repeated elsewhere.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_promoted_with_audit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_decide_rejects_invalid_transition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_rejects_invalid_payload.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_roundtrip.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/suggested_link_log_trims_stream_by_max_entries.rs`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_link_graph_agentic --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The full `test_link_graph_agentic` binary passed (`5 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New reusable support module:
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/461-xiuxian-wendao-link-graph-agentic-support-module-boundary-2026-03-07.md`
