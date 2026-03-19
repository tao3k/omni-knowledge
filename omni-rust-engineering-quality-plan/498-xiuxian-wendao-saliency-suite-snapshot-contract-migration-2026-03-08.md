# 498. Xiuxian Wendao Saliency Suite Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the migration of the remaining `xiuxian-wendao` saliency test suite from a split direct-assertion layout into a fixture-backed snapshot contract test binary.

## Why This Change Was Needed

The repository is steadily standardizing Wendao tests around fixture-backed contract coverage. After the live saliency rerank regression was migrated, the legacy `test_link_graph_saliency` suite still used a split directory of direct assertion tests:

- two pure saliency formula checks,
- four Valkey-backed persistence/update checks,
- a dedicated support module plus a separate entry binary.

That structure was functionally correct, but inconsistent with the newer contract-first testing layout already used across Wendao.

## What Changed

### 1) Replaced the split saliency suite with one contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency_contracts.rs`

This new binary covers the same six saliency behaviors as fixture-backed JSON contracts:

- activation boosts score,
- clamped bounds,
- touch + get roundtrip,
- invalid payload auto-repair,
- inbound edge ZSET score update,
- batched Valkey reads.

### 2) Added dedicated saliency contract support helpers

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_saliency_contract_support.rs`

This helper now owns:

- fixture assertion routing,
- Valkey availability checks,
- unique prefix generation,
- Valkey cleanup helpers,
- shared snapshot formatting helpers.

A monotonic counter was added to unique prefixes so the contract tests remain isolated under parallel execution.

### 3) Added fixture-backed expected outputs for all six saliency scenarios

Added fixture roots:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/compute_activation_boosts_score/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/compute_clamps_bounds/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/touch_and_get_with_valkey/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/auto_repairs_invalid_payload/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/touch_updates_inbound_edge_zset/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/get_many_with_valkey/expected/result.json`

Why this matters:

- expected behavior is now stored as stable JSON contracts,
- the saliency suite matches Wendao’s broader fixture-first testing conventions,
- future refactors can update explicit contracts rather than scattered inline assertions.

### 4) Removed the superseded split suite

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/`

Why this matters:

- the migrated tests do not coexist with their direct-assertion predecessors,
- the suite remains easier to navigate,
- repository test structure continues converging on one contract pattern.

## Architectural Takeaways

### Snapshot contracts also fit stateful Valkey-backed tests

Contract fixtures are not limited to pure parser or CLI lanes. They also work well for stateful persistence tests when the result surface is normalized into deterministic JSON.

### Parallel test isolation needs stronger prefix discipline

The original nanosecond-only prefix strategy was acceptable in a smaller suite, but the fixture-contract binary benefits from a counter-extended prefix to prevent cross-test contamination under parallel execution.

### Delete superseded tests immediately after migration

Removing the split suite at the same time as the contract replacement keeps intent clear and avoids duplicate maintenance.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_saliency_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/compute_activation_boosts_score/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/compute_clamps_bounds/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/touch_and_get_with_valkey/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/auto_repairs_invalid_payload/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/touch_updates_inbound_edge_zset/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/saliency/get_many_with_valkey/expected/result.json`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_activation_boosts_score.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/compute_link_graph_saliency_clamps_bounds.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_get_many_with_valkey.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_store_auto_repairs_invalid_payload.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_and_get_with_valkey.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_touch_updates_inbound_edge_zset.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-saliency-contracts cargo check -p xiuxian-wendao --test test_link_graph_saliency_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-saliency-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_saliency_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-saliency-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_saliency_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_saliency_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_saliency_contracts` passed (`6 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_saliency_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency_contracts.rs`
- Shared helpers: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_saliency_contract_support.rs`
