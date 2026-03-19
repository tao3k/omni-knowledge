# 426. Xiuxian Wendao Scored-Context Reconstruction Stage

Date: 2026-03-07

## Scope

This shard records the eighteenth pure-Rust hybrid-retriever slice.

The slice extracts scored-batch decoding and final `QuantumContext`
reconstruction into a dedicated `scored_context.rs` stage so `orchestrate.rs`
continues shrinking toward pure stage composition.

## Why This Change Was Needed

Shard `425` extracted semantic-anchor resolution, but the output side of
`orchestrate.rs` still mixed three responsibilities:

- decoding the fused saliency column from the scored Arrow batch,
- reconstructing domain `QuantumContext` values from staged candidates,
- applying the final deterministic ordering.

Those concerns all belong to the post-scoring output stage, not to the main
orchestration coordinator. A modern Rust pipeline should make its output stage
just as explicit as its input and resolution stages.

## What Changed

### 1) Added `scored_context.rs` as the output-stage module

New file:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scored_context.rs`

It defines:

- `QuantumContextCandidate`
- `quantum_contexts_from_scored_batch(...)`

Why this matters:

- the post-scoring boundary now has a concrete module and name,
- saliency-column decoding and final context assembly are grouped together,
- `orchestrate.rs` no longer carries output reconstruction details inline.

### 2) Candidate state moved to the scored-context boundary

`QuantumContextCandidate` now lives alongside the code that consumes it after
batch scoring.

Why this matters:

- the type lives closest to the stage that finishes the pipeline,
- row-aligned reconstruction details stay local to the scored-context module,
- the orchestration coordinator no longer owns a state type whose final purpose
  is output reassembly.

### 3) Saliency decoding is now isolated from orchestration

`scored_context.rs` now owns:

- fused-column lookup,
- `Float64` downcast validation,
- null saliency handling,
- final stable ordering of `QuantumContext` values.

Why this matters:

- orchestration no longer needs to know how the scored batch is decoded,
- the output-stage invariants are all in one place,
- future output-shape changes can be contained within one module.

### 4) `orchestrate.rs` now reads more clearly as a pipeline coordinator

After the refactor, `orchestrate.rs` is effectively responsible for:

- entry-point adaptation,
- batch validation,
- semantic-anchor resolution,
- topology expansion,
- scoring invocation,
- delegation to scored-context reconstruction.

Why this matters:

- the file now better matches its name,
- stage boundaries are increasingly visible in the code,
- future refactors can target one stage at a time instead of reopening a
  monolithic orchestration file.

## Regression Coverage Added

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

New regression added:

- duplicate anchor rows in the batch-native path remain distinct and correctly
  ordered after scored-batch reconstruction.

Why this matters:

- the new output stage is directly covered by row-index-sensitive behavior,
- saliency extraction and final ordering remain protected for duplicate rows,
- the batch-native path now mirrors the duplicate-row guarantees already tested
  for the slice-based API.

## Architectural Takeaways

### Output reconstruction is a real stage, not a helper footnote

If a pipeline has explicit input validation and semantic-resolution stages, it
should also have an explicit output reconstruction stage. The code should make
that symmetry obvious.

### Keep staged state near the stage that consumes it

`QuantumContextCandidate` is not a general orchestration concept. It is the
bridge between graph expansion and scored-output reconstruction, so it belongs
near that boundary.

### Separate score production from score interpretation

The scorer produces a batch. Another stage interprets that batch and rebuilds
domain objects. Treating those as separate layers reduces coupling and keeps
future evolution cleaner.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scored_context.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage cargo check -p xiuxian-wendao --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
CARGO_TARGET_DIR=/tmp/xiuxian-scored-context-stage NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 81 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_anchor_batch` passed (`7 passed, 77 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion` passed (`3 passed, 81 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 80 skipped`).

## Limits and Next Slice

This slice gives the output stage a real home, but `orchestrate.rs` still owns
both entry-point adaptation and topology-expansion candidate construction.

The next modernization slice should continue tightening the core:

- decide whether graph-expansion candidate construction deserves its own stage,
- keep the public API unchanged while shrinking orchestration further,
- avoid over-abstracting the graph-expansion loop if it is still the most
  concrete and readable place for that logic.

## Artifacts and Notes

- Blueprint: `.data/blueprints/wendao_hybrid_retriever.md`
- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/425-xiuxian-wendao-semantic-anchor-resolution-stage-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/426-xiuxian-wendao-scored-context-reconstruction-stage-2026-03-07.md`
