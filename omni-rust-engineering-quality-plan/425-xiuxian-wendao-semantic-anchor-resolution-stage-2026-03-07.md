# 425. Xiuxian Wendao Semantic-Anchor Resolution Stage

Date: 2026-03-07

## Scope

This shard records the seventeenth pure-Rust hybrid-retriever slice.

The slice extracts semantic-anchor resolution into a dedicated
`semantic_anchor.rs` stage inside `quantum_fusion`, so orchestration no longer
mixes batch entry handling with canonical-anchor normalization and page-index
recovery logic.

## Why This Change Was Needed

Shard `424` centralized Arrow batch validation, but `orchestrate.rs` still
carried two distinct responsibilities in one flow:

- orchestrating score fusion and context assembly,
- resolving each input row into a canonical semantic anchor with a seed doc id
  and semantic path.

Those are not the same concern. For high-quality Rust, the stage that answers
"is this row a usable semantic anchor?" should have a clear home and a clear
name instead of living inline inside orchestration loops.

## What Changed

### 1) Added `semantic_anchor.rs` as an explicit stage

New file:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`

It defines:

- `ResolvedQuantumAnchor`
- `LinkGraphIndex::resolve_quantum_anchors(...)`
- `LinkGraphIndex::resolve_quantum_anchor_row(...)`

Why this matters:

- the semantic-anchor stage now has a domain-specific namespace,
- anchor normalization, seed-doc lookup, and semantic-path recovery are grouped
  together instead of being spread through orchestration,
- future work on anchor resolution can evolve locally without bloating the
  public orchestration entry.

### 2) `orchestrate.rs` now composes stages instead of performing them inline

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
now follows this shape:

- validate Arrow batch contract,
- resolve semantic anchors,
- expand graph neighborhoods and compute topology scores,
- score the original batch,
- reassemble sorted `QuantumContext` values.

Why this matters:

- orchestration reads as a pipeline again,
- the file is closer to coordination logic rather than mixed coordination and
  semantic normalization,
- the remaining responsibilities inside `orchestrate.rs` are easier to reason
  about.

### 3) Canonical anchor identity is now produced by the resolution stage

`ResolvedQuantumAnchor` now holds:

- `batch_row`
- `batch_anchor_id`
- `anchor_id`
- `seed_doc_id`
- `semantic_path`
- `vector_score`

Why this matters:

- the resolution stage fully captures the semantic identity needed by later
  graph-expansion work,
- downstream candidate building can operate on resolved anchors instead of raw
  batch rows,
- the boundary between "resolution" and "expansion" is now explicit.

### 4) Candidate building now starts from resolved anchors only

`quantum_context_candidates_from_resolved_anchors(...)` no longer needs to:

- trim anchor ids,
- recover doc fallbacks,
- query the page index for semantic paths.

It now focuses on:

- related-doc expansion,
- topology-score aggregation,
- final candidate assembly.

Why this matters:

- the graph-expansion stage no longer hides semantic-resolution work inside it,
- later optimizations can target the real hot path directly,
- the code better matches the retrieval architecture described in the plan.

## Regression Coverage Added

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

New regression added:

- whitespace-padded doc fallback (`" plain "`) still resolves through the
  batch-native path and recovers the document semantic path.

Why this matters:

- the newly extracted resolution stage is now directly protected by a batch
  regression,
- canonical trimming and doc fallback remain covered after the refactor.

## Architectural Takeaways

### Name the semantic-resolution stage explicitly

If a retrieval pipeline has a step that transforms raw rows into canonical
semantic identities, that step deserves its own module. Hiding it in a loop
makes the architecture harder to see and harder to maintain.

### Keep orchestration focused on composition, not normalization details

`orchestrate.rs` should primarily compose validated input, resolved anchors,
graph expansion, and score extraction. The more normalization logic it carries,
the less useful it becomes as an architectural entry point.

### Distinguish semantic resolution from graph expansion

These stages both happen per anchor, but they answer different questions:

- semantic resolution: what anchor is this row really referring to?
- graph expansion: what surrounding topology should influence its saliency?

Treating them as one blob makes later optimization and debugging harder.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-semantic-anchor-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 80 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch` passed (`6 passed, 77 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 80 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 79 skipped`).

## Limits and Next Slice

This slice gives semantic-anchor resolution a real home, but the output side of
`orchestrate.rs` still combines saliency extraction, context assembly, and final
sorting in one module.

The next modernization slice should continue from there:

- extract scored-context reconstruction into its own stage or module,
- keep the boundary explicit between batch scoring output and final
  `QuantumContext` assembly,
- preserve the current typed error surface while shrinking orchestration toward
  pure stage composition.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/424-xiuxian-wendao-shared-quantum-anchor-batch-contract-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/425-xiuxian-wendao-semantic-anchor-resolution-stage-2026-03-07.md`
