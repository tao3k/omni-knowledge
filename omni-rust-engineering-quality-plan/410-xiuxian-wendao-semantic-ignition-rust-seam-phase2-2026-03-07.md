---
type: knowledge
metadata:
  title: "Xiuxian-Wendao Semantic Ignition Rust Seam Phase 2"
  date: "2026-03-07"
  status: "completed"
---

# Xiuxian-Wendao Semantic Ignition Rust Seam Phase 2 (2026-03-07)

## Scope

This shard records the second pure-Rust implementation slice of the Wendao hybrid retriever.

Phase 1 established `QuantumAnchorHit` and `QuantumContext` as the retrieval orchestration contract. Phase 2 adds the missing backend seam so upstream semantic search can feed Wendao without coupling `xiuxian-wendao` to `xiuxian-vector`, a provider-specific engine, or any Python wrapper.

## Files Added Or Updated

- `.agent/execplans/wendao-hybrid-retriever.md`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`

## Public API Added

- `QuantumSemanticSearchRequest<'a>`
- `QuantumSemanticIgnition`
- `QuantumSemanticIgnitionFuture<'a, E>`
- `LinkGraphIndex::quantum_contexts_from_semantic_ignition`

## Engineering Decisions Captured

### 1) Keep Wendao in charge of anchor semantics

The backend seam returns `Vec<QuantumAnchorHit>`, not `VectorSearchResult` or any other upstream row type.

Why this matters:

- `xiuxian-wendao` owns the meaning of an anchor id.
- The retrieval surface does not leak `xiuxian-vector` data contracts.
- Future backends can adapt their own result rows into the same Wendao-native anchor contract.

### 2) Use a request type instead of ad-hoc parameters

`QuantumSemanticSearchRequest<'a>` carries:

- optional raw query text,
- precomputed semantic vector,
- bounded limit.

Why this matters:

- the seam stays extensible,
- callers pass one cohesive unit instead of three loosely-related arguments,
- normalization rules live in one place.

### 3) Normalize at the boundary

`QuantumSemanticSearchRequest::normalized()` trims empty text and clamps `limit` to at least `1`.
`QuantumSemanticSearchRequest::is_empty()` lets Wendao short-circuit useless calls before touching the backend.

Why this matters:

- invalid or degenerate requests do not leak into backend implementations,
- empty semantic requests avoid unnecessary I/O,
- request hygiene becomes deterministic and testable.

### 4) Avoid public `async fn` trait warnings with an explicit future alias

The seam uses:

- `QuantumSemanticIgnition`
- `QuantumSemanticIgnitionFuture<'a, E>`

instead of a public trait with `async fn` methods.

Why this matters:

- the API does not introduce `async_fn_in_trait` warnings,
- the boundary remains pure Rust,
- backends can still implement the trait with `Box::pin(async move { ... })`.

### 5) Keep orchestration and backend I/O separate

`LinkGraphIndex::quantum_contexts_from_semantic_ignition` only does four things:

- normalize the request,
- short-circuit empty signals,
- delegate semantic anchor search,
- reuse `quantum_contexts_from_anchors` for the fusion phase.

Why this matters:

- Wendao does not duplicate fusion logic,
- backend implementations stay narrow,
- phase-1 and phase-2 contracts compose cleanly.

### 6) Test the seam at the package top level

Phase-2 tests live in `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`.

Coverage added:

- backend delegation and request normalization,
- empty-request short-circuiting,
- backend-error propagation.

Why this matters:

- the repository's top-level test placement rule remains intact,
- the seam is validated as externally-consumable API behavior,
- no inline `#[cfg(test)]` debt was introduced.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-hybrid-seam cargo check -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-hybrid-seam cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-hybrid-seam NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-hybrid-seam NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
```

Observed outcomes:

- `cargo check -p xiuxian-wendao` finished successfully.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` finished successfully.
- `semantic_ignition` targeted lane passed (`3 passed`).
- `quantum_fusion` targeted regression lane remained green (`2 passed`).

Note:

- A dedicated `CARGO_TARGET_DIR` was used because unrelated workspace validation held the shared `target/` build lock. This kept the validation evidence crate-scoped and reproducible.

## High-Quality Rust Takeaways

- Put the semantic backend seam on the orchestration crate boundary, not inside the backend crate.
- Return domain-native types at the boundary and keep backend-native rows behind adapters.
- Normalize public request structs once and reuse them across callers and tests.
- Prefer explicit future aliases over warning-prone public `async fn` traits when you need a stable public seam without adding extra dependencies.
- Keep the orchestration layer thin and compositional instead of duplicating phase-1 logic in phase 2.

## Next Step

The next Rust-only slice should add a concrete adapter in an upper integration crate that converts vector backend rows into `QuantumAnchorHit` values, with a clear anchor-id mapping policy such as:

- prefer `metadata["anchor_id"]`,
- fallback to `id` only when the backend row id is already canonical for Wendao.
