---
type: knowledge
metadata:
  title: "Xiuxian-Wendao Vector Adapter And Seam Boundary Convergence"
  date: "2026-03-07"
  status: "completed"
---

# Xiuxian-Wendao Vector Adapter And Seam Boundary Convergence (2026-03-07)

## Scope

This shard records the third pure-Rust implementation slice of the Wendao hybrid retriever.

Phase 2 defined a backend-agnostic semantic ignition seam. Phase 3 adds the concrete `xiuxian-vector` adapter and tightens the seam boundary after discovering that the original trait contract was over-constrained for the real backend.

## Files Added Or Updated

- `.agent/execplans/wendao-hybrid-retriever.md`
- `Cargo.toml`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao-vector/Cargo.toml`
- `packages/rust/crates/xiuxian-wendao-vector/README.md`
- `packages/rust/crates/xiuxian-wendao-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/mod.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/config.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/error.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/mapping.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/runtime.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_mapping.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_runtime.rs`

## Public API Added

- `xiuxian_wendao_vector::WendaoVectorSemanticIgnition`
- `xiuxian_wendao_vector::WendaoVectorSemanticIgnitionConfig`
- `xiuxian_wendao_vector::AnchorIdFallbackMode`
- `xiuxian_wendao_vector::map_search_result_to_anchor`
- `xiuxian_wendao_vector::map_search_results_to_anchors`

## Engineering Decisions Captured

### 1) Put the concrete adapter in a dedicated integration crate

The `xiuxian-vector` implementation does not live in `xiuxian-wendao` and does not live in `xiuxian-daochang`.

Why this matters:

- `xiuxian-wendao` remains the orchestration crate.
- `xiuxian-daochang` is not polluted with retrieval-adapter concerns.
- Dependency flow stays one-way: `xiuxian-wendao-vector -> xiuxian-vector + xiuxian-wendao`.

### 2) Do not over-constrain public traits beyond real backend guarantees

The original phase-2 seam used `Send + Sync` bounds on the trait and `Future + Send` on the boxed future. That was too strong for the real backend because `VectorStore` carries `Rc<KeywordIndex>` and is not thread-safe.

Why this matters:

- a public seam must model reality, not an aspirational contract that current backends cannot satisfy,
- removing unnecessary bounds is a quality improvement, not a relaxation mistake,
- the adapter can now compile without lying about thread safety.

### 3) Preserve custom metadata by choosing the right vector search surface

The adapter uses `VectorStore::search_optimized` with projected columns:

- `ID_COLUMN`
- `CONTENT_COLUMN`
- `METADATA_COLUMN`

Why this matters:

- the default `search()` path reconstructs `VectorSearchResult.metadata` from Arrow-native columns,
- that reconstruction hides custom keys such as `anchor_id`,
- the projection-aware search path preserves raw metadata so the adapter can recover canonical Wendao anchor ids.

### 4) Make anchor-id fallback explicit and auditable

`WendaoVectorSemanticIgnitionConfig` includes `AnchorIdFallbackMode`.

Current policy options:

- `Disabled`
- `ResultId`

Why this matters:

- fallback behavior is no longer implicit,
- callers can require metadata-only canonical anchors,
- using `result.id` becomes an explicit choice instead of accidental behavior.

### 5) Keep mapping logic pure and testable

The adapter splits into:

- `config.rs`
- `error.rs`
- `mapping.rs`
- `runtime.rs`

Why this matters:

- mapping can be tested without a live vector store,
- runtime integration stays thin,
- the crate follows the repository modularization rule instead of collapsing into one file.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter cargo fmt -p xiuxian-wendao -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter cargo check -p xiuxian-wendao -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-adapter NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
```

Observed outcomes:

- `xiuxian-wendao-vector` nextest lane passed (`5 passed`).
- `xiuxian-wendao` semantic ignition regression lane passed (`3 passed`).
- `xiuxian-wendao` quantum fusion regression lane passed (`2 passed`).
- `cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed cleanly.

## High-Quality Rust Takeaways

- Do not put concrete integration logic into the domain crate when a tiny upper integration crate gives a cleaner dependency graph.
- Public trait bounds should match actual backend capabilities; stronger is not better when it is false.
- Pick search APIs by data-contract behavior, not by name convenience. In this case, `search_optimized` with explicit projection was the correct surface, while `search()` was not.
- Separate pure mapping from runtime I/O so tests can validate policy without rebuilding the world.
- Turn fallback behavior into explicit config instead of hidden heuristics.

## Next Step

The next Rust-only slice should add optional adapter-level filters and policy knobs, for example:

- summary-only retrieval (`doc_type == 'summary'`),
- configurable `where_filter`,
- optional minimum semantic score threshold before Wendao runs topology diffusion.
