# 513. Xiuxian Wendao Link-Graph Agentic Expansion and Seed/Priors Contract Migration

Date: 2026-03-08

## Scope

This shard records the Wave 13 migration of two remaining small wrapper-based link-graph fixture contract families into one explicit external snapshot contract binary.

The retired wrappers were:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors.rs`

The retired wrapper trees were:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/`

They have now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_seed_priors_contracts.rs`

## Why This Change Was Needed

Both families already used committed fixtures and committed expected snapshots, but they still lived behind thin wrapper binaries and local module trees.

This wave removed that last indirection for these small link-graph fixture suites while preserving their existing fixture roots unchanged.

## What Changed

### 1) Added one external binary for both link-graph fixture families

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_seed_priors_contracts.rs`

The new binary covers:

- bounded agentic expansion planning budgets,
- query-narrowed agentic expansion planning,
- worker telemetry for bounded agentic expansion execution,
- seed-grounded related retrieval cluster accuracy,
- journal-seeded semantic retrieval surfacing agenda tasks,
- structural priors promoting an architecture hub.

### 2) Extracted the local seed/prior helper into shared support

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_seed_and_priors_contract_support.rs`

This replaces the old local helper module from the retired wrapper tree and keeps the new external binary independent from deleted local modules.

### 3) Deleted the superseded wrappers immediately

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion/`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-agentic-seed-priors cargo check -p xiuxian-wendao --test test_link_graph_agentic_seed_priors_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-agentic-seed-priors NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_agentic_seed_priors_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-link-graph-agentic-seed-priors cargo clippy -p xiuxian-wendao --test test_link_graph_agentic_seed_priors_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed for `test_link_graph_agentic_seed_priors_contracts`.
- `cargo nextest run` passed for `test_link_graph_agentic_seed_priors_contracts` (`6 passed, 0 skipped`).
- `cargo clippy` completed for the new binary.
- Remaining warnings observed during validation came from pre-existing library code in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs`, `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`, and an existing `too_many_lines` warning in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/policy.rs`.
