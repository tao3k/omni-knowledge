# 420. Xiuxian Wendao Arrow-Native Batch Quantum Scorer

Date: 2026-03-07

## Scope

This shard records the twelfth pure-Rust hybrid-retriever slice.

The slice lands an Arrow-native batch scorer inside `xiuxian-wendao` so the
hybrid retrieval stack can fuse semantic and topology saliency in a columnar
pass instead of row-by-row Rust object materialization.

## Why This Change Was Needed

The existing `quantum_fusion` slice already knew how to combine semantic and
topology scores, but only through scalar helpers embedded in orchestration
logic.

That left a clear engineering gap:

- no reusable Arrow-native scorer existed for batch fusion,
- downstream runtime work would otherwise keep rebuilding ad hoc row loops,
- the blueprint's columnar scoring direction was not yet represented in the
  Wendao API surface,
- there was no focused test lane guarding schema handling, null safety, or
  typed batch errors.

For a modern Rust retrieval stack, the scoring boundary should accept typed
columnar inputs, preserve existing Arrow buffers, and return a typed failure
mode instead of leaking stringly-typed errors.

## What Changed

### 1) `xiuxian-wendao` now depends on workspace `arrow`

`packages/rust/crates/xiuxian-wendao/Cargo.toml` now uses the workspace-managed
`arrow` dependency directly.

Why this matters:

- the crate can operate on native `RecordBatch` values without pulling in an
  ad hoc wrapper layer,
- Arrow version alignment stays centralized at the workspace boundary,
- the scoring API remains explicit about its columnar contract.

### 2) `BatchQuantumScorer` now provides a typed columnar fusion API

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scoring.rs`
now exposes:

- `BatchQuantumScorer`
- `BatchQuantumScorerError`
- `QUANTUM_SALIENCY_COLUMN`

`BatchQuantumScorer::score_batch(...)`:

- reads `Utf8` ids and `Float64` semantic scores directly from the input
  `RecordBatch`,
- looks up topology saliency from a caller-provided `HashMap<String, f64>`,
- reuses the existing scalar fusion logic via `fuse_saliency_score(...)`,
- appends a new Arrow `Float64Array` named `quantum_saliency`,
- preserves input schema metadata when constructing the output batch.

Why this matters:

- the scorer operates on Arrow columns instead of row structs,
- existing batch columns are reused through Arrow's shared array ownership,
- callers get a narrow, testable API for hybrid score fusion.

### 3) Batch failures are now strongly typed

The scorer does not return `Result<RecordBatch, String>`.
It returns `Result<RecordBatch, BatchQuantumScorerError>` with dedicated
variants for:

- missing columns,
- wrong Arrow column types,
- null values in required cells,
- Arrow batch construction failures.

Why this matters:

- public Rust APIs should not erase failure semantics into strings,
- integration code can match exact failure modes,
- error reporting stays aligned with the repository's typed-domain direction.

### 4) Public exports and focused tests were added

Exports were threaded through:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`

Focused integration coverage was added in:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

The tests verify:

- fused saliency is appended as a new column,
- schema metadata survives the batch rebuild,
- wrong similarity-column types fail with `InvalidFloat64Column`,
- null identifier rows fail with `NullValue`.

## Architectural Takeaways

### Columnar boundaries should stay columnar

The right place to materialize the new fusion result is the new output score
column itself. The rest of the input batch should remain Arrow-native instead of
being copied into `Vec<Struct>` or custom row DTOs.

### Typed error enums are part of high-quality Rust engineering

Batch processing code becomes easier to evolve and test when failure modes are
encoded as a domain enum instead of formatted strings.

### Preserve metadata when rebuilding schemas

`Schema::new_with_metadata(...)` keeps upstream batch metadata intact. That is a
small implementation detail, but it prevents silent loss of context once more
retrieval pipeline stages begin annotating Arrow batches.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/Cargo.toml`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scoring.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-batch-scorer cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-batch-scorer cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-batch-scorer NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-batch-scorer NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
```

Observed outcomes:

- `cargo fmt -p xiuxian-wendao` completed cleanly.
- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 73 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`2 passed, 74 skipped`).

## Limits and Next Slice

This slice intentionally stops at the reusable scorer boundary.
It does not yet wire `BatchQuantumScorer` into `orchestrate.rs`.

The next hybrid-retriever slice should:

- feed pre-normalized topology saliency into the scorer,
- perform vector-score thresholding before batch scoring,
- let orchestration pass `RecordBatch` values through the scorer directly
  instead of reconstructing row-wise scoring inputs.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/420-xiuxian-wendao-arrow-native-batch-quantum-scorer-2026-03-07.md`
