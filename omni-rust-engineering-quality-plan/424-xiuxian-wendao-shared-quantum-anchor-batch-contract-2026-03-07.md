# 424. Xiuxian Wendao Shared Quantum Anchor-Batch Contract

Date: 2026-03-07

## Scope

This shard records the sixteenth pure-Rust hybrid-retriever slice.

The slice introduces a shared validated Arrow batch contract for
`quantum_fusion`, so both orchestration and batch scoring consume the same
`QuantumAnchorBatchView` instead of re-validating the same input layout in two
separate modules.

## Why This Change Was Needed

Shard `423` added a first-class batch-native orchestration entry, but one piece
of engineering debt remained:

- `orchestrate.rs` validated the input batch schema and nullability,
- `scoring.rs` validated the same batch schema and nullability again,
- both modules separately looked up the same columns and downcast them to Arrow
  arrays,
- row iteration still mixed validation concerns with actual domain work.

That duplication was not catastrophic, but it was not high-quality Rust
engineering either. Once a batch contract becomes authoritative, it should be
validated once and then reused through a small typed view.

## What Changed

### 1) Added `anchor_batch.rs` as the shared batch-contract module

New file:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/anchor_batch.rs`

It defines:

- `QuantumAnchorBatchView<'a>`
- `QuantumAnchorBatchRow<'a>`
- `QuantumAnchorBatchError`

Why this matters:

- the Arrow input contract now has one domain-specific home,
- batch column lookup, downcasting, and null validation are no longer copied
  across modules,
- the rest of the retrieval pipeline can operate on a prevalidated view instead
  of re-checking raw Arrow structures repeatedly.

### 2) Validation now happens once before orchestration work begins

`QuantumAnchorBatchView::new(...)` now performs:

- required-column lookup,
- `Utf8`/`Float64` type validation,
- first-null detection for required columns.

After that, `rows()` yields typed `QuantumAnchorBatchRow` values without
repeating null checks inside every consumer loop.

Why this matters:

- orchestration code can focus on semantic-path and graph logic,
- scoring code can focus on saliency fusion,
- the batch contract is now explicit and front-loaded.

### 3) `BatchQuantumScorer` now consumes the shared validated view

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scoring.rs`
no longer performs its own duplicated column lookup and null scanning inside
`score_batch(...)`.

Instead it:

- constructs `QuantumAnchorBatchView`,
- delegates to `score_anchor_batch_view(...)`,
- iterates typed rows from the validated view.

Why this matters:

- the scorer keeps the same public API,
- internal duplication is removed,
- row-by-row saliency fusion no longer carries schema-validation noise.

### 4) `quantum_fusion` orchestration now consumes the same view

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now constructs `QuantumAnchorBatchView` once in
`quantum_contexts_from_anchor_batch(...)`.

Candidate preparation then becomes an infallible pass over validated rows.
Only the boundary that constructs the batch view remains fallible for input
contract reasons.

Why this matters:

- validation and business logic are cleanly separated,
- the candidate-building pass is simpler and more local,
- the orchestration path and scorer now share one notion of what a valid anchor
  batch means.

### 5) Public error surfaces keep their own typed boundaries

The shared internal error type is not exposed publicly.
Instead it is mapped into:

- `QuantumContextBuildError`
- `BatchQuantumScorerError`

Why this matters:

- internal deduplication does not flatten the external API,
- callers still match the error families they already understand,
- the internal contract can evolve without forcing unrelated caller churn.

## Regression Coverage Added

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

New coverage added for the batch-native orchestration contract:

- missing identifier column -> `MissingInputColumn`
- wrong identifier column type -> `InvalidInputUtf8Column`
- wrong score column type -> `InvalidInputFloat64Column`
- null identifier cell -> `NullInputValue`
- custom column names plus skipped blank/unresolved rows still succeed

Existing scorer regression coverage remained green:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`

## Architectural Takeaways

### Validate the execution format once, then hand out a typed view

A typed view is a better abstraction than scattered ad hoc Arrow checks. It is
small enough to stay cheap, but strong enough to make downstream code simpler.

### Internal deduplication should not leak into public error design

The shared contract lives internally, while `QuantumContextBuildError` and
`BatchQuantumScorerError` remain separate. That is the right balance between
reuse and boundary clarity.

### Batch pipelines improve when validation and domain work are separated

Once null and type checks are front-loaded, the loops that do actual graph work
or score fusion become easier to read, easier to test, and harder to regress.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scoring.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-anchor-contract NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 79 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch` passed (`5 passed, 77 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 79 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 78 skipped`).

## Limits and Next Slice

This slice centralizes the batch contract, but semantic-path recovery and graph
expansion are still driven row by row.

The next modernization slice should focus on the next real pressure point:

- isolate semantic-anchor resolution into a smaller, explicitly named stage,
- keep batch validation separate while reducing orchestration branching further,
- avoid inventing pseudo-columnar abstractions where the graph lookup is still
  fundamentally per-anchor.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/423-xiuxian-wendao-batch-native-quantum-anchor-batch-entry-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/424-xiuxian-wendao-shared-quantum-anchor-batch-contract-2026-03-07.md`
