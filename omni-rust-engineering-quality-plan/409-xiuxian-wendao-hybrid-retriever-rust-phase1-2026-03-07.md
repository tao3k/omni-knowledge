---
type: knowledge
metadata:
  title: "Xiuxian-Wendao Hybrid Retriever Rust Phase 1"
  date: "2026-03-07"
  status: "completed"
---

# Xiuxian-Wendao Hybrid Retriever Rust Phase 1 (2026-03-07)

## Scope

This shard records the first pure-Rust implementation slice of Wendao hybrid retrieval inside `xiuxian-wendao`.

The goal of this wave was not to wire a concrete embedding backend. The goal was to land a high-quality retrieval orchestration layer that:

- accepts typed semantic-anchor inputs,
- reconstructs full `PageIndex` ancestry for traceability,
- diffuses through `LinkGraph` topology with bounded PPR,
- fuses semantic and topology scores deterministically,
- stays completely inside Rust.

## Files Added Or Updated

- `.data/blueprints/wendao_hybrid_retriever.md`
- `.agent/execplans/wendao-hybrid-retriever.md`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scoring.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/page_indices.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Engineering Decisions Captured

### 1) Keep the retriever boundary in Rust and provider-agnostic

The retriever API accepts `QuantumAnchorHit` rather than reaching directly into any vector engine.

Why this matters:

- `xiuxian-wendao` stays focused on graph/tree orchestration.
- No Python binding or wrapper layer is introduced.
- Future vector integration can remain a Rust trait or adapter seam instead of becoming a cross-language dependency.

### 2) Promote semantic ancestry to a reusable API instead of burying it in call-site logic

`LinkGraphIndex::page_index_semantic_path` and `LinkGraphIndex::page_index_trace_label` were added as dedicated helpers.

Why this matters:

- callers do not need to understand the internal `PageIndex` storage layout,
- traceability becomes a stable contract,
- test coverage can target ancestry recovery directly.

### 3) Split orchestration from scoring

The new `quantum_fusion/` module is broken into focused files:

- `orchestrate.rs` handles anchor resolution, path recovery, topology expansion, and final ordering,
- `scoring.rs` handles score fusion math.

Why this matters:

- responsibilities stay clear,
- clippy pressure stays localized,
- future scoring experiments do not force changes into graph traversal code.

### 4) Treat traceability as a product contract, not a debugging afterthought

`QuantumContext` carries both raw component scores and a canonical trace label.

Why this matters:

- downstream LLM prompting can attach a stable `[Path: Root > Section > Leaf]` label,
- score explanations remain inspectable,
- retrieval output stays debuggable without extra reconstruction passes.

### 5) Prefer focused top-level tests over inline `#[cfg(test)]` blocks

The retrieval behavior is validated in `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`.

Why this matters:

- source modules remain implementation-focused,
- tests stay aligned with the repository rule that complex module tests should live under package-level `tests/`,
- the targeted nextest lane is easy to rerun.

## Public API Landed

- `LinkGraphIndex::page_index_semantic_path(&self, anchor_id: &str) -> Option<Vec<String>>`
- `LinkGraphIndex::page_index_trace_label(&self, anchor_id: &str) -> Option<String>`
- `LinkGraphIndex::quantum_contexts_from_anchors(&self, anchors: &[QuantumAnchorHit], options: &QuantumFusionOptions) -> Vec<QuantumContext>`
- `QuantumContext::trace_label() -> String`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_fusion
```

Observed outcomes:

- `cargo check -p xiuxian-wendao` finished successfully.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` finished successfully.
- `cargo nextest` ran the targeted `test_link_graph::quantum_fusion` lane and both tests passed.

Passing tests:

- `test_quantum_contexts_from_anchors_recover_semantic_path_and_fuse_scores`
- `test_page_index_semantic_path_supports_anchor_and_doc_fallbacks`

## High-Quality Rust Takeaways

- Start with a narrow typed contract before wiring external engines.
- Reuse existing domain primitives (`PageIndex`, PPR traversal) instead of adding a second retrieval abstraction too early.
- Keep traceability in the return type, not as side-channel logging.
- Use module boundaries to separate orchestration from math.
- Use targeted integration tests to lock behavior before backend coupling.

## Next Step

The next Rust-only slice should add a semantic ignition seam that remains fully native to Rust, for example a trait-based or adapter-based backend that produces `QuantumAnchorHit` values for `xiuxian-wendao` without introducing Python dependencies.
