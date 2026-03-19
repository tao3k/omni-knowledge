# 516. Xiuxian Wendao Hierarchical Uplink Activation

Date: 2026-03-08

## Scope

This shard records the activation of a first-class hierarchical uplink contract
inside `xiuxian-wendao`.

## Why This Change Was Needed

The page-index parent topology had already been landed through `parent_id` and
`node_parent_map`, but the retrieval layer still lacked a dedicated semantic
uplink API.

That left two quality gaps:

- lineage recovery lived behind page-index helper methods instead of a stable
  retrieval-facing model,
- hybrid retrieval still reconstructed semantic path information without an
  explicit hierarchical record type.

## What Changed

### Hierarchical Hit Model

Added `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/hierarchical_hit.rs`
with a new `HierarchicalHit` record that captures:

- canonical `anchor_id`,
- owning `doc_id`,
- physical document `path`,
- recovered `semantic_path`.

The model also exposes a `trace_label()` helper for downstream traceability.

### Dedicated Hierarchical Resolver

Added `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/hierarchical.rs`.

This module now centralizes lineage extraction by:

- canonicalizing anchor ids,
- following `node_parent_map` to reconstruct ancestry,
- resolving root-level document fallback paths,
- producing `HierarchicalHit` as a stable retrieval contract.

### PageIndex API Reuse

Updated `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
so:

- `page_index_semantic_path()` now delegates to the hierarchical lineage
  extractor,
- `page_index_trace_label()` now delegates to `HierarchicalHit::trace_label()`.

This removes duplicate ancestry-walk logic from the page-index helper module.

### Quantum Retrieval Integration

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`
so quantum anchor resolution now consumes `hierarchical_hit()`.

Important compatibility rule retained:

- `QuantumContext.anchor_id` still preserves the caller-provided trimmed anchor
  string for existing payload fixtures,
- canonical doc ids are used internally for lineage and document ownership.

### Fixture Coverage

Extended `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
and its fixture support so the crate now verifies:

- leaf-anchor hierarchical uplink,
- document-level stem fallback uplink,
- trace label rendering through the new model.

Added fixtures:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/hierarchical_hit_gamma.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/hierarchical_hit_doc.json`

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

- Structural metadata is not finished when the tree exists; retrieval still
  needs a stable contract that can consume that topology directly.
- Canonical lineage resolution should live in one place and feed both page-index
  helper APIs and hybrid retrieval.
- External retrieval payloads sometimes need compatibility-preserving raw anchor
  ids even when internal lineage resolution canonicalizes them.
- Fixture-backed tests are the right place to lock down traceability contracts
  such as `[Path: Root > Section > Leaf]` labels.

## Artifacts and Notes

Changed paths:

- `.agent/execplans/wendao-page-index-rust-core.md`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/hierarchical.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/hierarchical_hit.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/hierarchical_hit_gamma.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/hierarchical_hit_doc.json`
