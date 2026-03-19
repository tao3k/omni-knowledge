# 415. Wendao Vector Hybrid Search Runtime Entry Point

Date: 2026-03-07

## Scope

This shard records the seventh pure-Rust hybrid-retriever slice.

The slice adds a real runtime/search entry point in `xiuxian-wendao-vector` that executes Wendao graph planning first and then launches semantic ignition only when the retrieval plan selects it.

## Why This Change Was Needed

Phase 6 closed the model gap between retrieval plans and vector execution plans, but there was still no actual runtime that used that bridge.

That left one remaining layer of manual glue:

- callers still had to invoke graph planning themselves,
- callers still had to read `retrieval_plan` and decide whether semantic ignition should run,
- callers still had to size semantic search requests manually.

Modern Rust engineering should not leave that orchestration scattered across call sites.

## What Changed

### 1) `xiuxian-wendao-vector` now owns a true hybrid-search runtime

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs` now defines:

- `WendaoVectorHybridSearcher`

This runtime holds:

- one `VectorStore`
- one `WendaoVectorSemanticIgnitionConfig`

Why this matters:

- the integration crate now has a concrete runtime surface instead of only helper layers,
- higher-level callers can reuse one configured hybrid-search object,
- backend-specific orchestration stays out of `xiuxian-wendao`.

### 2) The runtime accepts a typed hybrid-search request model

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/models.rs` now defines:

- `WendaoVectorHybridSearchRequest`
- `WendaoVectorHybridSearchOutput`

The request carries:

- raw graph query text,
- semantic query vector,
- graph-search limit,
- `LinkGraphSearchOptions`,
- optional provisional-search overrides,
- `QuantumFusionOptions`.

Why this matters:

- the runtime input is explicit and typed,
- planning inputs and semantic-fusion inputs stay separated but travel together,
- callers do not need to manually stitch together graph-search and semantic-stage arguments.

### 3) Retrieval plans now directly govern runtime execution

`WendaoVectorHybridSearcher::search(...)` now:

1. runs `LinkGraphIndex::search_planned_payload_with_agentic(...)`,
2. reads the embedded `retrieval_plan`,
3. uses `WendaoVectorSemanticExecutionPlan::from_retrieval_plan(...)`,
4. short-circuits if the plan selected `GraphOnly`,
5. launches semantic ignition only when the plan selected a semantic path.

Why this matters:

- the planned payload is now the authoritative runtime contract,
- the bridge introduced in phase 6 is no longer just a helper API,
- graph-only requests do not accidentally open the vector path.

### 4) Semantic request sizing now honors the planning budget

The runtime uses:

- `retrieval_plan.budget.candidate_limit`

as the semantic request limit passed into `QuantumSemanticSearchRequest`.

Why this matters:

- planning and execution now share one budget source,
- callers do not invent a second semantic limit policy,
- the runtime follows the retrieval-plan contract rather than bypassing it.

### 5) Integration tests now verify both runtime branches

`packages/rust/crates/xiuxian-wendao-vector/tests/test_hybrid_search.rs` now proves:

- graph-sufficient queries remain graph-only and skip semantic ignition,
- graph-insufficient queries run semantic ignition through the planned policy,
- summary-only semantic policy survives planning and affects semantic execution,
- hybrid output carries both the planned payload and semantic contexts.

Why this matters:

- the runtime boundary is covered by focused integration tests,
- the new entry point is validated against real `LinkGraphIndex` plus `VectorStore` fixtures,
- the crate now has a genuine Rust-only path from planning to semantic execution.

## Architectural Takeaways

### Put backend-specific orchestration in the integration crate

`xiuxian-wendao` should emit planning contracts, not execute backend-specific semantic stages.

The integration crate is the right place to combine:

- graph planning,
- retrieval-plan gating,
- vector adapter execution.

### Let the plan own both policy and budget

The retrieval plan already knows:

- which path was selected,
- what semantic policy applies,
- what execution budget should be honored.

The runtime should execute that contract, not reinterpret it.

### Prefer a stateful runtime object over loose helper functions

`WendaoVectorHybridSearcher` makes the ownership boundary obvious:

- configured store + config live together,
- search execution is one method,
- call sites stay smaller and more consistent.

## Files Changed

- `packages/rust/crates/xiuxian-wendao-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/mod.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/models.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_hybrid_search.rs`
- `packages/rust/crates/xiuxian-wendao-vector/README.md`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-hybrid cargo check -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-hybrid cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-hybrid NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector
```

Observed outcomes:

- `cargo check -p xiuxian-wendao-vector` completed cleanly.
- `cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed successfully for the target crate.
- A pre-existing transitive pedantic warning surfaced from `packages/rust/crates/xiuxian-wendao/src/skill_vfs/resolver/runtime.rs:41` during the clippy build.
- `cargo nextest run -p xiuxian-wendao-vector` passed all lanes (`15 passed`).

## Next Step

The next clean Rust-only step is to expose this runtime through a higher-level Rust caller that already owns query-vector generation, so text queries no longer need test-only precomputed vectors to reach the semantic stage.
