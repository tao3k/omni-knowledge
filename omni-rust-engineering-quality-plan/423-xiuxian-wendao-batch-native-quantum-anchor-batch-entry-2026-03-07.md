# 423. Xiuxian Wendao Batch-Native Quantum Anchor-Batch Entry

Date: 2026-03-07

## Scope

This shard records the fifteenth pure-Rust hybrid-retriever slice.

The slice adds a first-class batch-native orchestration entry in
`xiuxian-wendao` so callers can pass a prepared Arrow `RecordBatch` directly
into `quantum_fusion` without rebuilding a second canonical batch just for the
public API boundary.

## Why This Change Was Needed

Shard `422` made the orchestration path typed and removed the scalar fallback,
but the public entry point still centered the slice-based API:

- `LinkGraphIndex::quantum_contexts_from_anchors(&[QuantumAnchorHit], ...)`

That left one remaining modernization gap:

- prepared semantic rows still had to be converted into Rust structs before
  they could be reassembled into an Arrow batch,
- the orchestration code still treated batch input as an internal detail rather
  than a first-class contract,
- row identity across blank, unresolved, and duplicated anchors was easy to get
  wrong once saliency scores were extracted back out of the scored batch.

For a modern Rust retrieval pipeline, once Arrow is already the execution
format, the orchestration layer should accept Arrow natively and preserve row
identity explicitly.

## What Changed

### 1) Added a batch-native public entry

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now exposes:

- `LinkGraphIndex::quantum_contexts_from_anchor_batch(...)`

This method accepts:

- `&RecordBatch`
- `id_col: &str`
- `score_col: &str`
- `&QuantumFusionOptions`

Why this matters:

- callers with an existing Arrow batch no longer need to materialize an
  intermediate `Vec<QuantumAnchorHit>`,
- the public API now reflects the real columnar execution model,
- the orchestration boundary is closer to a true Arrow-native pipeline.

### 2) The legacy slice-based entry now delegates to the batch-native path

`LinkGraphIndex::quantum_contexts_from_anchors(...)` now builds one canonical
batch from `QuantumAnchorHit` values and delegates directly into
`quantum_contexts_from_anchor_batch(...)`.

Why this matters:

- there is now one authoritative orchestration implementation,
- the older slice-based API stays available as a compatibility wrapper,
- future optimizations only need to improve the batch-native core.

### 3) Candidate preparation now preserves input-batch row identity

`QuantumContextCandidate` now stores:

- `batch_row`
- `batch_anchor_id`
- `anchor_id`

This distinction is intentional:

- `batch_anchor_id` preserves the raw input string for scorer map lookup,
- `anchor_id` stores the trimmed canonical identifier used in the returned
  `QuantumContext`,
- `batch_row` lets the scorer output be read back without rebuilding or
  filtering the scored batch.

Why this matters:

- whitespace-padded anchor ids still match the original batch row during score
  lookup,
- unresolved and blank rows can be skipped without corrupting score alignment,
- duplicate anchors remain distinct because row identity is explicit.

### 4) Input validation errors now cover batch-column contracts

`QuantumContextBuildError` now includes:

- `MissingInputColumn`
- `InvalidInputUtf8Column`
- `InvalidInputFloat64Column`
- `NullInputValue`

Why this matters:

- the batch-native entry has a precise typed contract,
- Arrow layout problems are reported before graph traversal starts,
- callers can distinguish invalid input shape from later scoring failures.

### 5) Scoring now operates on the original input batch

The orchestration layer no longer builds a second candidate-only scoring batch.
Instead it:

- computes candidate metadata from the input rows,
- builds the topology score map from `batch_anchor_id`,
- invokes `BatchQuantumScorer` on the original batch,
- reads saliency scores back by `batch_row`.

Why this matters:

- the hot scoring path remains aligned with the batch the caller actually
  provided,
- intermediate batch reconstruction is avoided,
- the design is closer to a fully columnar orchestrator where row position is
  part of the contract.

## Regression Coverage Added

New test file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

New assertions cover:

- custom input column names (`anchor_ref`, `semantic_score`),
- whitespace-padded anchor ids that must still recover the canonical semantic
  path,
- unresolved and blank rows that must be skipped without panicking or shifting
  saliency alignment,
- wrong score-column types that must fail with
  `QuantumContextBuildError::InvalidInputFloat64Column`.

## Architectural Takeaways

### Batch-native APIs should become public once the runtime is already columnar

Keeping Arrow hidden behind slice-only APIs delays the real architecture.
The public boundary should expose the dominant execution format instead of
forcing callers through extra materialization steps.

### Preserve raw row identity separately from canonical domain identity

The orchestrator needs both:

- raw batch identity for score lookup and row alignment,
- canonical domain identity for semantic-path resolution and output.

Collapsing those two concepts would reintroduce subtle bugs around whitespace,
duplicate anchors, and skipped rows.

### Skip unresolved rows without rewriting the batch

A high-quality batch pipeline does not need to filter and rebuild Arrow data
just to ignore a few invalid semantic anchors. Preserving row indices lets the
orchestrator stay simple and keeps the scorer aligned with the caller's batch.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-batch-native cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-batch-native cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-batch-native NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-batch-native NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-quantum-batch-native NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch` passed (`2 passed, 77 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 76 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 75 skipped`).

## Limits and Next Slice

This slice makes batch input a first-class public orchestration contract, but
it still performs candidate preparation row by row before fusion.

The next modernization slice should push further into pure columnar orchestration:

- reduce repeated string extraction and per-row branching in candidate
  preparation,
- investigate whether semantic-path recovery can be split into a batch-friendly
  prepass and a graph-lookup pass,
- keep expanding typed error coverage only where the batch contract becomes more
  explicit, not more implicit.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/421-xiuxian-wendao-quantum-fusion-orchestration-batch-scorer-integration-2026-03-07.md`
  - `assets/knowledge/omni-rust-engineering-quality-plan/422-xiuxian-wendao-typed-quantum-orchestration-errors-and-fallback-removal-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/423-xiuxian-wendao-batch-native-quantum-anchor-batch-entry-2026-03-07.md`
