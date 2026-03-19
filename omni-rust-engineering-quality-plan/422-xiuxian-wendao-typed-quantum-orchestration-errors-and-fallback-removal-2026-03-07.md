# 422. Xiuxian Wendao Typed Quantum-Orchestration Errors and Fallback Removal

Date: 2026-03-07

## Scope

This shard records the fourteenth pure-Rust hybrid-retriever slice.

The slice removes the last scalar fallback from the `quantum_fusion`
orchestration path and replaces it with explicit typed errors that propagate
through `xiuxian-wendao`, `xiuxian-wendao-vector`, and `xiuxian-daochang`.

## Why This Change Was Needed

Shard `421` routed the real orchestration path through `BatchQuantumScorer`, but
it still preserved an infallible public API by logging internal failures and
falling back to scalar fusion.

That was still engineering debt:

- the production path was columnar, but failure semantics were still hidden,
- callers could not distinguish backend-search failures from orchestration
  failures,
- the fallback kept two scoring paths alive,
- `xiuxian-daochang` and `xiuxian-wendao-vector` were not yet forced to handle
  real orchestration errors explicitly.

For high-quality Rust, once a path is intended to be authoritative, its error
surface should be explicit and typed rather than hidden behind recovery logic.

## What Changed

### 1) `quantum_contexts_from_anchors(...)` is now explicitly fallible

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now exports `QuantumContextBuildError` and changes:

- `LinkGraphIndex::quantum_contexts_from_anchors(...)`

from:

- `Vec<QuantumContext>`

to:

- `Result<Vec<QuantumContext>, QuantumContextBuildError>`

Why this matters:

- the Arrow batch build and score extraction path is now the only scoring path,
- failure modes are explicit instead of being logged-and-hidden,
- callers now have a real contract for orchestration failure handling.

### 2) The scalar fallback was physically removed

`score_quantum_context_candidates(...)` no longer logs and falls back to
`fuse_saliency_score(...)`.

Instead it now:

- builds the Arrow scoring batch,
- runs `BatchQuantumScorer`,
- extracts the fused column,
- returns a typed error if any step fails.

Why this matters:

- there is one authoritative scoring path,
- orchestration no longer keeps a hidden legacy behavior branch,
- future batch-native optimizations do not need to preserve a second scalar lane.

### 3) Semantic ignition now distinguishes backend and orchestration failures

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
now exports:

- `QuantumSemanticIgnitionError<E>`

with separate variants for:

- `Backend { backend_name, source }`
- `Orchestration { backend_name, source }`

Why this matters:

- backend search failures and Wendao orchestration failures are no longer
  conflated,
- the backend identifier is preserved in the error surface,
- semantic-ignition callers can now match the exact failure layer.

### 4) `xiuxian-wendao-vector` now maps orchestration failures explicitly

`packages/rust/crates/xiuxian-wendao-vector/src/adapter/error.rs` now adds:

- `WendaoVectorSemanticIgnitionError::Orchestration`

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs`
now maps:

- `QuantumSemanticIgnitionError::Backend` -> underlying vector adapter error
- `QuantumSemanticIgnitionError::Orchestration` -> adapter `Orchestration`

Why this matters:

- the vector integration crate preserves its stable public error type,
- orchestration failures are not collapsed into fake adapter-search failures,
- the caller-facing semantic runtime can report the true fault domain.

### 5) `xiuxian-daochang` now handles orchestration failure at its boundary

`packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/runtime.rs`
now uses `anyhow::Context` when calling `quantum_contexts_from_anchors(...)`.

Why this matters:

- the top-level tool runtime now acknowledges that hybrid-context construction
  can fail,
- failure reporting at the application boundary keeps the Wendao source chain,
- the caller no longer depends on silent recovery inside Wendao.

### 6) Tests were updated to the new error surface

Updated tests:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`

Notable regression coverage:

- semantic-ignition backend failures now assert on
  `QuantumSemanticIgnitionError::Backend` instead of comparing raw backend
  errors,
- orchestration callers now compile against the real typed `Result` surface.

## Architectural Takeaways

### Do not keep hidden recovery paths once the new path is authoritative

A temporary fallback is useful during migration, but once the columnar scorer is
fully integrated, continuing to hide failures only makes the system harder to
reason about.

### Layered error enums clarify responsibility boundaries

`QuantumContextBuildError` and `QuantumSemanticIgnitionError<E>` separate:

- Arrow/batch orchestration failures,
- backend anchor-search failures,
- application-boundary handling.

That is a materially better design than returning plain backend errors from a
multi-stage orchestration function.

### Preserve stable caller-facing boundaries with explicit mapping

`xiuxian-wendao-vector` did not need a breaking public API rename. It only
needed an additional typed variant to represent the new upstream failure class.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/error.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/runtime.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao -p xiuxian-wendao-vector -p xiuxian-daochang
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result cargo check -p xiuxian-wendao -p xiuxian-wendao-vector -p xiuxian-daochang --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -p xiuxian-daochang -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector --test test_runtime
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector --test test_hybrid_search
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-result NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit
```

Observed outcomes:

- `cargo check -p xiuxian-wendao -p xiuxian-wendao-vector -p xiuxian-daochang --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -p xiuxian-daochang -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 74 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 73 skipped`).
- `cargo nextest run -p xiuxian-wendao-vector --test test_runtime` passed (`4 passed`).
- `cargo nextest run -p xiuxian-wendao-vector --test test_hybrid_search` passed (`2 passed`).
- `cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit` passed (`20 passed, 1 skipped`).

## Limits and Next Slice

This slice makes the orchestration path typed and explicit, but it still builds
an intermediate Arrow batch inside `orchestrate.rs`.

The next modernization slice should push one step further:

- introduce a batch-native orchestration entry that accepts prepared semantic
  rows directly,
- reduce intermediate row-to-batch conversion inside orchestration,
- keep the current typed error boundary while moving more of the runtime toward
  a first-class columnar pipeline.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/420-xiuxian-wendao-arrow-native-batch-quantum-scorer-2026-03-07.md`
  - `assets/knowledge/omni-rust-engineering-quality-plan/421-xiuxian-wendao-quantum-fusion-orchestration-batch-scorer-integration-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/422-xiuxian-wendao-typed-quantum-orchestration-errors-and-fallback-removal-2026-03-07.md`
