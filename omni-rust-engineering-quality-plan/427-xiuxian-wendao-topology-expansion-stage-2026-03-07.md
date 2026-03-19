# 427. Xiuxian Wendao Topology-Expansion Stage

Date: 2026-03-07

## Scope

This shard records the nineteenth pure-Rust hybrid-retriever slice.

The slice extracts graph-expansion candidate construction into a dedicated
`topology_expansion.rs` stage so `orchestrate.rs` moves even closer to pure
pipeline coordination.

## Why This Change Was Needed

After shards `425` and `426`, `orchestrate.rs` was already slimmer, but it still
owned one materially separate stage:

- starting from resolved semantic anchors,
- performing related-doc graph expansion,
- aggregating topology saliency,
- constructing post-expansion candidates.

That logic is not orchestration glue. It is the topology-expansion stage of the
hybrid retriever and should be named and isolated as such.

## What Changed

### 1) Added `topology_expansion.rs` as the graph-expansion stage

New file:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/topology_expansion.rs`

It now owns:

- `LinkGraphIndex::expand_quantum_context_candidates(...)`
- `LinkGraphIndex::quantum_related_ranked_doc_ids(...)`

Why this matters:

- graph-expansion responsibilities now have a clear module boundary,
- topology traversal and saliency aggregation no longer live inline inside the
  orchestration entry,
- future tuning of PPR expansion stays local to the stage that owns it.

### 2) `orchestrate.rs` now delegates resolved anchors into topology expansion

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now composes:

- batch validation,
- semantic-anchor resolution,
- topology expansion,
- scoring,
- scored-context reconstruction.

Why this matters:

- the file now reads like an explicit pipeline instead of a mixed-control file,
- each stage has its own physical home,
- continuing refactors can stay incremental without re-growing the orchestrator.

### 3) Candidate construction is now explicitly a topology concern

`expand_quantum_context_candidates(...)` is responsible for:

- expanding each resolved anchor through related-doc ranking,
- slicing `related_clusters` according to `related_limit`,
- computing `topology_score`,
- producing `QuantumContextCandidate` values for the scored-context stage.

Why this matters:

- the boundary between semantic resolution and topology expansion is now
  visible in code,
- the output of graph expansion is explicit and testable,
- stage ownership is easier to maintain across future retrieval changes.

## Regression Coverage Strengthened

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

Coverage strengthened in two places:

- batch-native custom-column path now asserts that `related_clusters` contains a
  real neighboring document (`docs/beta` or `docs/gamma`),
- batch-native duplicate-row coverage remains in place to protect row identity
  after topology expansion and scoring.

Why this matters:

- the extracted topology stage is now guarded by a concrete related-cluster
  assertion,
- the batch-native path has stronger evidence that graph expansion still feeds
  correct downstream context construction.

## Architectural Takeaways

### Graph expansion is a real stage, not leftover orchestration code

When a pipeline has explicit semantic resolution and explicit scoring, the
intermediate graph-expansion step should be named and isolated too.

### Stage extraction should follow responsibility boundaries, not line counts

This refactor was worthwhile because topology expansion answers a distinct
question from orchestration or scored-output reconstruction, not because the
file happened to be long.

### Preserve concrete loops when they are the real domain work

`topology_expansion.rs` still uses a direct loop over resolved anchors. That is
correct. The goal is not fake vectorization, but clear ownership of the graph
traversal stage.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/topology_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-topology-expansion-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 81 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch` passed (`7 passed, 77 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 81 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 80 skipped`).

## Limits and Next Slice

This slice leaves `orchestrate.rs` with only a small amount of real logic, but
it still owns both the public entry adaptation and the scoring invocation.

The next modernization slice should decide whether the scoring invocation path
itself deserves one last extracted stage, or whether the remaining orchestrator
is already at the right size and clarity threshold.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/426-xiuxian-wendao-scored-context-reconstruction-stage-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/427-xiuxian-wendao-topology-expansion-stage-2026-03-07.md`
