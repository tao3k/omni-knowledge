# 418. Daochang Hybrid Wendao Search Runtime and Typed Semantic Export

Date: 2026-03-07

## Scope

This shard records the tenth pure-Rust hybrid-retriever slice.

The slice adds a typed semantic-document export surface in `xiuxian-wendao`
and wires one real `xiuxian-daochang` caller, `wendao.search`, through that
export plus `EmbeddingClient` so hybrid retrieval can run end-to-end without
test-only scaffolding.

## Why This Change Was Needed

Phase 9 added the first real query-vectorizer adapter, but the caller path was
still incomplete.

Two gaps remained:

- `xiuxian-daochang` still had no stable typed API for consuming
  `PageIndex`-derived semantic rows from `xiuxian-wendao`.
- `wendao.search` still did not execute one real hybrid lane end-to-end from
  planned query text to rendered semantic contexts.

Without this slice, the Rust contracts were still partially theoretical.

## What Changed

### 1) `xiuxian-wendao` now exports typed semantic documents

`packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/semantic_document.rs`
now defines:

- `LinkGraphSemanticDocument`
- `LinkGraphSemanticDocumentKind`

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/semantic_documents.rs`
now adds:

- `LinkGraphIndex::semantic_documents()`
- `LinkGraphIndex::semantic_documents_for(...)`

Why this matters:

- downstream runtimes no longer need private tree access or ad hoc traversal,
- the export keeps stable anchor ids, semantic paths, line ranges, and kind
  metadata,
- semantic content is stored as `Arc<str>`, which keeps the caller boundary
  allocation-aware instead of re-copying owned strings everywhere.

### 2) Semantic export preserves `PageIndex` traceability semantics

The export now emits:

- one summary row per document,
- one section row per `PageIndexNode`,
- the complete logical ancestry path for every row,
- line-range metadata when the source maps to one concrete section.

Why this matters:

- caller runtimes can keep `[Path: Root > Section > Leaf]` style provenance,
- semantic filters can distinguish summary vs section scope without guessing,
- Wendao remains the authoritative owner of anchor semantics.

### 3) `xiuxian-daochang` now hosts a dedicated `wendao_search/` runtime module

New files under
`packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/`:

- `mod.rs`
- `tool.rs`
- `runtime.rs`
- `render.rs`

Current runtime flow:

1. `WendaoSearchTool` builds the planned graph-search payload.
2. Graph-only plans render immediately without semantic work.
3. Hybrid/vector plans select semantic documents from the new Wendao export.
4. The caller-owned `EmbeddingClient` embeds both query text and semantic rows.
5. The runtime scores documents with pure-Rust cosine similarity.
6. Top rows become `QuantumAnchorHit` values and flow back through
   `LinkGraphIndex::quantum_contexts_from_anchors(...)`.
7. XML-Lite output appends semantic `<hit>` rows with trace labels.

Why this matters:

- `wendao.search` now has one real end-to-end hybrid lane,
- tool logic, runtime logic, and rendering are modularized instead of being
  collapsed into `bridge.rs`,
- the runtime stays pure Rust and caller-owned.

### 4) `EmbeddingClient` ownership now flows as `Arc`

The runtime bootstrap path now threads `Option<Arc<EmbeddingClient>>` through
agent construction so the zhenfa tool can borrow one shared embedding runtime.

Updated integration points include:

- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/memory.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhenfa.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/builder.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/embedding_runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/admission.rs`

Why this matters:

- the tool can safely share one embedding client with the wider runtime,
- ownership is explicit and future caller integrations can reuse the same path.

### 5) Focused tests now validate both the export and the real caller lane

New and updated tests:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/wendao_search_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_zhenfa_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/tests.rs`

They verify:

- semantic document export preserves summary/section structure and ancestry,
- the real `wendao.search` hybrid lane emits XML-Lite semantic hits with
  escaped path labels,
- the new caller runtime composes cleanly with the existing zhenfa test harness.

## Architectural Takeaways

### Export typed retrieval artifacts instead of leaking storage internals

`xiuxian-wendao` should expose semantic retrieval inputs as typed records,
not raw map access or caller-side tree walking.

### Keep caller-side orchestration in a capability-focused directory module

`tool.rs`, `runtime.rs`, and `render.rs` are different responsibilities.
Splitting them early keeps the zhenfa surface maintainable.

### Use caller-owned pure-Rust scoring when the lower runtime boundary is not yet compatible

The current `xiuxian-vector` stack is still not suitable for direct use under
`ZhenfaTool`'s `Send` future requirements.

The caller-owned embedding path plus pure-Rust cosine scoring keeps the hybrid
lane executable today without fake bounds or lint suppression.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/semantic_document.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/semantic_documents.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/bridge.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/tool.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/wendao_search/render.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/memory.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhenfa.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/builder.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/embedding_runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/admission.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/zhenfa.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/wendao_search_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_zhenfa_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/tests.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao -p xiuxian-daochang
CARGO_TARGET_DIR=/tmp/xiuxian-hybrid-check cargo check -p xiuxian-wendao -p xiuxian-daochang
CARGO_TARGET_DIR=/tmp/xiuxian-hybrid-check cargo clippy -p xiuxian-wendao -p xiuxian-daochang -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-hybrid-check NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph -E 'test(test_link_graph_page_index_exports_semantic_documents)'
CARGO_TARGET_DIR=/tmp/xiuxian-hybrid-check NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit
```

Observed outcomes:

- `cargo check -p xiuxian-wendao -p xiuxian-daochang` completed cleanly.
- `cargo clippy -p xiuxian-wendao -p xiuxian-daochang -- -W clippy::too_many_lines`
  completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph -E 'test(test_link_graph_page_index_exports_semantic_documents)'`
  passed (`1 passed, 72 skipped`).
- `cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit`
  passed (`20 passed, 1 skipped`).

## Next Step

The next clean slice is to converge this caller-owned semantic scorer with a
shared vector-backed runtime once the non-`Send` vector boundary is resolved,
so `wendao.search` can reuse one authoritative semantic backend instead of
maintaining a parallel cosine-scoring path.
