# 503. Xiuxian Wendao Quantum Retrieval Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` quantum retrieval suite in `xiuxian-wendao` from internal direct tests to one fixture-backed snapshot contract binary.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_quantum_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_quantum_contract_support.rs`

## Why This Change Was Needed

After the parser and tree-scope waves, the next strongest migration target was the quantum retrieval family because it was already a single capability cluster in practice:

- all scenarios were about quantum-context construction and fusion,
- the success-path tests already shared the hybrid fixture namespace,
- the remaining non-snapshot assertions were mostly Arrow-batch and validation-error cases.

Migrating the entire family together kept the audit surface coherent and avoided the piecemeal churn the user explicitly wanted to avoid.

## What Changed

### 1) Added a dedicated quantum retrieval contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_quantum_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_quantum_contract_support.rs`

The new binary covers the full quantum retrieval surface:

- Arrow-native saliency scoring contracts,
- quantum contexts from precomputed anchors,
- quantum contexts from prepared Arrow batches,
- semantic ignition delegation and error propagation.

Why this matters:

- one binary now expresses the entire quantum retrieval contract surface,
- batch scoring and orchestration failures are reviewed through stable JSON snapshots,
- support code moved into `tests/support/`, matching the external-contract pattern from earlier waves.

### 2) Preserved existing hybrid snapshots and added missing error fixtures

Preserved existing expected fixtures under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/appends_fused_saliency_column.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/rejects_null_identifier_values.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/rejects_wrong_similarity_column_type.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_missing_identifier_column.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_null_identifier_values.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_wrong_identifier_column_type.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_wrong_score_column_type.json`

Why this matters:

- the already-good hybrid fixture tree stays canonical,
- validation failures now use the same snapshot discipline as success paths,
- the contract binary remains easy to diff and extend.

### 3) Removed the superseded internal module tree

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- the migrated suite no longer coexists with the older direct-test form,
- the internal `test_link_graph` module graph is smaller,
- this wave satisfies the user's request to delete migrated direct tests immediately.

## Architectural Takeaways

### Cohesive fixture roots are strong migration anchors

The quantum family already shared one hybrid fixture namespace. That made it possible to externalize structure without rewriting stable expected outputs.

### Validation errors deserve snapshot contracts too

Arrow batch schemas and type/nullable failures are part of the retrieval contract, not incidental test setup details. Capturing them as snapshots makes regressions clearer.

### Semantic policy should remain a separate wave

`semantic_policy.rs` sits closer to query parsing and retrieval planning than to runtime quantum-context orchestration. Keeping it out of this wave preserves batch cohesion.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_quantum_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_quantum_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/appends_fused_saliency_column.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/rejects_null_identifier_values.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/batch_quantum_scorer/rejects_wrong_similarity_column_type.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_missing_identifier_column.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_null_identifier_values.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_wrong_identifier_column_type.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/rejects_wrong_score_column_type.json`
- `.cache/codex/execplans/wendao-test-snapshot-migration-wave-3-quantum.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-contracts cargo check -p xiuxian-wendao --test test_link_graph_quantum_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_quantum_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_quantum_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_quantum_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_quantum_contracts` passed (`17 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_quantum_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_quantum_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_quantum_contract_support.rs`
- Wave plan: `.cache/codex/execplans/wendao-test-snapshot-migration-wave-3-quantum.md`
- Deferred next candidate: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
