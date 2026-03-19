# 421. Xiuxian Wendao Quantum-Fusion Orchestration Batch-Scorer Integration

Date: 2026-03-07

## Scope

This shard records the thirteenth pure-Rust hybrid-retriever slice.

The slice moves `LinkGraphIndex::quantum_contexts_from_anchors(...)` off direct
scalar score fusion and onto the new Arrow-native `BatchQuantumScorer`, while
keeping the current public orchestration API stable.

## Why This Change Was Needed

Shard `420` introduced a reusable columnar scorer, but the real Wendao
orchestration path was still bypassing it.

That left the design only half-finished:

- the public hybrid-retrieval path still fused scores row by row,
- the batch scorer was available but not actually exercised by orchestration,
- future runtime work would still have to bridge two scoring implementations,
- there was no regression coverage for duplicate anchor rows through the real
  orchestration entry point.

A high-quality Rust retrieval system should not stop at building a better
primitive. It should also route the production orchestration path through that
primitive.

## What Changed

### 1) `quantum_contexts_from_anchors(...)` now scores in a columnar pass

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now splits the work into two distinct stages:

- `quantum_context_candidates(...)`
  - recovers semantic ancestry,
  - runs topology diffusion,
  - preserves related-cluster ids,
  - prepares normalized semantic and topology scores.
- `score_quantum_context_candidates(...)`
  - builds an Arrow `RecordBatch`,
  - runs `BatchQuantumScorer`,
  - extracts the fused `quantum_saliency` column,
  - feeds those scores back into the final `QuantumContext` values.

Why this matters:

- orchestration and scoring are now cleanly separated concerns,
- the production entry point now actually uses the columnar scorer,
- score assembly is no longer duplicated inline inside the orchestration loop.

### 2) Orchestration now has a narrow Arrow boundary

The orchestration module now defines a small internal batch schema:

- `anchor_id`
- `vector_score`

These columns are sufficient for `BatchQuantumScorer` because topology saliency
is provided through the side `HashMap<String, f64>` keyed by anchor id.

Why this matters:

- the batch contract remains minimal and explicit,
- orchestration does not materialize custom row DTOs purely for scoring,
- the scorer integration is easy to replace or extend later.

### 3) Candidate preparation was extracted into a focused internal model

`QuantumContextCandidate` is now the private in-memory representation of a
prepared orchestration row.

It holds:

- `anchor_id`
- `semantic_path`
- `related_clusters`
- `vector_score`
- `topology_score`

Why this matters:

- the orchestration loop no longer interleaves graph traversal with score
  fusion,
- each helper has one job,
- future changes to scoring or ranking can operate on candidates without
  rewriting the traversal logic.

### 4) Infallible public API was preserved with an explicit invariant guard

`LinkGraphIndex::quantum_contexts_from_anchors(...)` still returns
`Vec<QuantumContext>`.

Because the public API is currently infallible, the integration keeps a narrow
internal fallback path:

- batch build failure logs an error and falls back to scalar fusion,
- batch-score extraction failure logs an error and falls back to scalar fusion.

Why this matters:

- existing callers (`xiuxian-daochang`, semantic-ignition composition, tests)
  do not need an immediate API migration,
- the primary path is now columnar,
- the remaining debt is explicit: if we want fully typed orchestration errors,
  the next slice should make this API fallible instead of relying on an
  invariant guard.

### 5) Real orchestration coverage now guards duplicate anchor rows

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
now adds:

- `test_quantum_contexts_from_anchors_keep_duplicate_anchor_rows_distinct`

The test verifies that:

- duplicate anchor rows are both preserved,
- the contexts remain individually ranked by fused saliency,
- duplicate rows with different semantic scores do not collapse into one output,
- traceability labels remain intact.

## Architectural Takeaways

### Integrating a primitive matters as much as designing it

A reusable batch scorer has limited value if the main orchestration path keeps
using the old scalar fusion logic.

### Keep traversal and scoring separate

Graph traversal, context assembly, and score fusion are different concerns.
Introducing a private candidate model is a better long-term design than keeping
all three responsibilities inside one loop.

### If an API is infallible, isolate the recovery path

The right place for a temporary invariant guard is the orchestration seam
itself, not scattered across callers. That keeps the debt localized until the
public API can move to a typed `Result`.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-batch-orchestrate cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-batch-orchestrate cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-batch-orchestrate NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-batch-orchestrate NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 74 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 74 skipped`).

## Limits and Next Slice

This slice keeps the public orchestration API infallible.
That means the batch integration still carries a localized scalar fallback for
unexpected internal Arrow failures.

The next slice should choose one of two directions explicitly:

- make `quantum_contexts_from_anchors(...)` a typed `Result` and remove the
  invariant fallback,
- or push Arrow-native data deeper into orchestration so the batch scorer
  becomes the only realistic execution path.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/420-xiuxian-wendao-arrow-native-batch-quantum-scorer-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/421-xiuxian-wendao-quantum-fusion-orchestration-batch-scorer-integration-2026-03-07.md`
