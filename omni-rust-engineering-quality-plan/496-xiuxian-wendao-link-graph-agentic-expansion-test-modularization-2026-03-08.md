# 496. Xiuxian Wendao Link Graph Agentic Expansion Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_link_graph_agentic_expansion.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original fixture-backed file mixed distinct plan and execution contracts in
one top-level implementation file:

- worker and pair budget planning,
- query-constrained planning,
- execution telemetry without persistence.

Those contracts share one feature area, but plan-only and execution behavior
should not remain mixed in one implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`
so it now acts as a thin integration-test launcher while preserving the
crate-level shared support modules required by the fixture helpers.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for local re-exports of shared fixture assertions and snapshots,
- `plan.rs` for planning-only fixture contracts,
- `execution.rs` for execution telemetry behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph_agentic_expansion --no-fail-fast
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_agentic_expansion --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Fixture-backed plan and execution contracts should be split into separate
  modules even when they share one support stack.
- Local support re-exports are the cleanest way to consume crate-level shared
  fixture helpers from directory-module test suites.
- Thin entrypoints preserve a stable test binary while making the planning and
  execution surfaces easier to grow independently.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/plan.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/execution.rs`
