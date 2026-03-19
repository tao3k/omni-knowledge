# 499. Xiuxian Wendao Live Saliency Rerank External Contract Migration

Date: 2026-03-08

## Scope

This shard records the final externalization of the live saliency rerank regression in `xiuxian-wendao`.

The earlier migration moved the behavior into a snapshot lane, but that lane still lived under `src/` as an internal test module. This change completes the migration by moving the regression into the repository's integration-test contract layout, adding explicit input fixtures, and deleting the superseded source test.

## Why This Change Was Needed

The user requested that Wendao tests be migrated to snapshot style and that the original migrated tests be removed after replacement.

The remaining rerank lane still violated that goal in two ways:

- it lived inside `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/tests.rs`,
- it did not use fixture input materialization like the crate's other contract binaries.

That left the migration only partially complete.

## What Changed

### 1) Added a dedicated external contract binary for live saliency reranking

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_live_saliency_rerank_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_live_saliency_rerank_contract_support.rs`

What the new contract does:

- materializes fixture input docs into a temp tree,
- writes a per-test Wendao config pointing search runtime at a unique Valkey prefix,
- runs a baseline `search_planned(...)`,
- writes learned saliency for `notes/zeta`,
- runs the boosted `search_planned(...)`,
- snapshots both baseline and boosted result sets into JSON.

Why this matters:

- the regression now exercises the public search path rather than private helper functions,
- the test follows the same contract layout used across other Wendao fixture lanes,
- runtime isolation is explicit via a per-test Valkey prefix.

### 2) Added fixture input materialization for the rerank scenario

Added:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/input/notes/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/input/notes/zeta.md`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/expected/result.json`

The expected snapshot now captures two stable states:

- baseline ordering with equal scores and path-order tie-break,
- boosted ordering after learned saliency moves `notes/zeta.md` ahead of `notes/alpha.md`.

Why this matters:

- the contract now proves the reorder delta directly,
- fixture inputs make the test reproducible and inspectable,
- the result shape is aligned with the repository's input/expected snapshot convention.

### 3) Removed the superseded internal source test

Removed:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/tests.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/mod.rs`

Why this matters:

- the migrated regression no longer coexists with its obsolete internal predecessor,
- the search pipeline source module stays focused on production code,
- the user-requested delete-after-migration policy is now satisfied.

## Architectural Takeaways

### Public-path contracts are stronger than private helper tests

The new contract validates the behavior through `LinkGraphIndex::build(...)` and `search_planned(...)`. That gives better coverage than asserting private ranking helpers in isolation.

### Snapshot contracts can encode state transitions, not just final outputs

By snapshotting both baseline and boosted hits, the fixture captures the important behavioral delta while remaining deterministic and easy to update intentionally.

### Runtime isolation is critical for Valkey-backed integration tests

Using a unique key prefix per test keeps the contract safe under parallel execution and avoids cross-test contamination.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_live_saliency_rerank_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_live_saliency_rerank_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/input/notes/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/input/notes/zeta.md`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/expected/result.json`
- `.cache/codex/execplans/wendao-live-saliency-rerank.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/tests.rs`

Updated to remove the deleted module reference:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency-rerank cargo check -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency-rerank NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency-rerank cargo clippy -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts` passed (`1 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_live_saliency_rerank_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_live_saliency_rerank_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_live_saliency_rerank_contract_support.rs`
- Fixture root: `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/`
