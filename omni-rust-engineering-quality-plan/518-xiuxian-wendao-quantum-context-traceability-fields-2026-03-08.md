# 518. Xiuxian Wendao Quantum Context Traceability Fields

Date: 2026-03-08

## Scope

This shard records the traceability expansion of `QuantumContext` in
`xiuxian-wendao`.

## Why This Change Was Needed

Even after the pipeline was consolidated around `HierarchicalHit`, the public
`QuantumContext` contract still exposed only:

- `anchor_id`,
- `semantic_path`,
- score fields.

That meant downstream consumers still lacked two stable traceability fields on
this public result object:

- the owning canonical `doc_id`,
- the physical relative `path`.

Without those fields, a caller that wanted provenance-rich rendering still had
to perform extra lookups or heuristics outside the retrieval contract.

## What Changed

### Public Result Contract

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
so `QuantumContext` now includes:

- `doc_id: String`
- `path: String`

These values are sourced directly from the already-resolved `HierarchicalHit`.

### Constructor Alignment

Updated `QuantumContext::from_hierarchical_hit(...)` so the public result now
materializes:

- compatibility-preserving outward `anchor_id`,
- canonical `doc_id`,
- physical `path`,
- `semantic_path`,
- topology and saliency scores.

### Fixture Locking

Updated
`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`
and the hybrid expected fixtures so the public contract now snapshots these new
traceability fields across:

- quantum-anchor-batch coverage,
- quantum-fusion coverage,
- semantic-ignition coverage.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast`
  passed (`87 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- A public retrieval result should carry enough provenance to be rendered or
  audited without forcing another index lookup round-trip.
- Internal contract consolidation is not enough if the public result object
  still drops important ownership and path metadata.
- Fixture snapshots are the right mechanism for locking down additive public
  result fields across multiple retrieval entrypoints.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/custom_columns.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/doc_fallback.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/duplicate_rows.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/contexts_from_anchors.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/duplicate_anchor_rows.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/page_index_doc_fallback.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/delegates_and_recovers_trace.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/semantic_ignition/respects_min_vector_score.json`
