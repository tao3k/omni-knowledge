# 416. Wendao Vector Deferred Query Vectorization Seam

Date: 2026-03-07

## Scope

This shard records the eighth pure-Rust hybrid-retriever slice.

The slice adds a text-first hybrid-search seam to `xiuxian-wendao-vector` so callers no longer need to precompute query vectors before Wendao planning runs.

## Why This Change Was Needed

Phase 7 added a real hybrid-search runtime, but it still required callers to provide a precomputed semantic query vector.

That left two engineering problems:

- graph-only requests still had to pay vectorization cost before planning could reject semantic ignition,
- higher-level callers still had to own the timing and normalization of query-vector generation.

That is the wrong cost model and the wrong ownership model.

## What Changed

### 1) `xiuxian-wendao-vector` now defines a pure-Rust query-vectorizer seam

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/vectorizer.rs` now defines:

- `WendaoHybridQueryVectorizer`
- `WendaoHybridQueryVectorizerFuture`

Why this matters:

- the integration crate now owns the contract for text-to-vector conversion,
- higher-level runtimes can supply their own embedding implementation without changing Wendao planning or vector adapter code,
- query vectorization is now a typed Rust seam, not call-site glue.

### 2) There is now a text-first hybrid-search request model

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/models.rs` now includes:

- `WendaoVectorHybridTextSearchRequest`

This request carries:

- raw text query,
- graph-search limit,
- `LinkGraphSearchOptions`,
- provisional-search overrides,
- `QuantumFusionOptions`.

Why this matters:

- callers can execute hybrid search from text without manually precomputing vectors,
- planning inputs stay explicit and typed,
- semantic vectorization is delayed until planning proves it is necessary.

### 3) `WendaoVectorHybridSearcher::search_text` now defers vectorization

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs` now includes:

- `search_text(...)`
- `search_text_planned(...)`
- `WendaoVectorHybridTextSearchError`

Current runtime flow:

1. run Wendao planning,
2. inspect `retrieval_plan`,
3. if planning selected `GraphOnly`, return immediately without vectorization,
4. otherwise invoke the query vectorizer,
5. launch semantic ignition through the existing execution bridge.

Why this matters:

- graph-sufficient requests avoid embedding work entirely,
- the semantic path still reuses the same validated execution bridge from previous slices,
- text-first callers now have one canonical runtime path.

### 4) Query vectorization now uses the normalized planned query

The text-first runtime now vectorizes:

- `planned.query`

instead of the raw user input.

Why this matters:

- semantic embeddings stay aligned with Wendao's normalized query text,
- directive syntax does not leak into vectorization,
- planning and semantic execution now consume the same logical query.

### 5) Integration tests now prove deferred vectorization behavior

`packages/rust/crates/xiuxian-wendao-vector/tests/test_text_hybrid_search.rs` now verifies:

- graph-only plans do not call the vectorizer,
- semantic plans do call the vectorizer exactly once,
- vectorizer failures are wrapped in `WendaoVectorHybridTextSearchError::Vectorize`,
- semantic execution still produces the expected `QuantumContext` output.

Why this matters:

- the optimization is enforced by tests,
- the new text-first seam is covered at the runtime boundary,
- callers can rely on both the cost model and the error model.

## Architectural Takeaways

### Plan first, vectorize second

If planning can determine that semantic ignition is unnecessary, embedding work should not happen.

This is both a performance improvement and a boundary improvement.

### Keep embedding implementations outside the integration contract

The integration crate should define the vectorization seam, not hardcode one embedding runtime.

That allows higher-level crates to plug in:

- in-process embedding,
- HTTP embedding,
- cached embedding,
- gateway-owned embedding runtimes.

### Normalize once and reuse the normalized query

When planning already resolves a canonical query string, semantic vectorization should use that same text.

This avoids subtle drift between graph search and semantic search.

## Files Changed

- `packages/rust/crates/xiuxian-wendao-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/mod.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/error.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/models.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/vectorizer.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_text_hybrid_search.rs`
- `packages/rust/crates/xiuxian-wendao-vector/README.md`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-text-hybrid cargo check -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-text-hybrid cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-text-hybrid NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector
```

Observed outcomes:

- `cargo check -p xiuxian-wendao-vector` completed cleanly.
- `cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao-vector` passed all lanes (`18 passed`).

## Next Step

The next clean Rust-only step is to add one higher-level adapter that implements `WendaoHybridQueryVectorizer` over an existing embedding runtime so text-first hybrid search can be invoked from a real caller without test-only stub vectorizers.
