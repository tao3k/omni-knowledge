# 419. Xiuxian Vector Send-Safe Keyword Index and Hybrid Future Boundaries

Date: 2026-03-07

## Scope

This shard records the eleventh pure-Rust hybrid-retriever slice.

The slice does not yet swap `xiuxian-daochang` onto the shared
`xiuxian-wendao-vector` runtime, but it removes the foundational async/thread
safety blockers that previously made that convergence impossible.

## Why This Change Was Needed

Phase 10 proved one real caller-owned hybrid lane in `wendao.search`, but the
shared runtime still could not safely cross the `ZhenfaTool` boundary.

Three blockers were hiding underneath that gap:

- `VectorStore` cloned an `Rc<KeywordIndex>`.
- `KeywordIndex` kept its writer cache in `RefCell<Option<IndexWriter>>`.
- Both hybrid seam aliases, `QuantumSemanticIgnitionFuture` and
  `WendaoHybridQueryVectorizerFuture`, erased `Send` from their boxed futures.

That meant the stack was not merely missing a runtime bootstrap; it was also
encoding non-thread-safe assumptions in core retrieval types.

## What Changed

### 1) `xiuxian-vector` removed `Rc` from shared runtime state

`packages/rust/crates/xiuxian-vector/src/lib.rs` now stores:

- `Option<Arc<KeywordIndex>>`

instead of `Option<Rc<KeywordIndex>>`.

`packages/rust/crates/xiuxian-vector/src/ops/core.rs` now initializes the
keyword index with `Arc::new(...)`.

Why this matters:

- cloned `VectorStore` values can now participate in thread-safe async
  runtimes,
- the public runtime surface no longer advertises single-thread-only ownership
  for shared keyword state.

### 2) `KeywordIndex` replaced interior `RefCell` mutation with `Mutex`

`packages/rust/crates/xiuxian-vector/src/keyword/index/shared.rs` now stores:

- `Mutex<Option<IndexWriter>>`

and exposes small lock helpers used by
`packages/rust/crates/xiuxian-vector/src/keyword/index/write_ops.rs`.

Why this matters:

- the Tantivy writer cache is now synchronized explicitly,
- `KeywordIndex` can satisfy `Sync` instead of relying on `RefCell`,
- write-path semantics remain localized to the keyword index module.

### 3) Hybrid seam future aliases are now `Send`

`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
now defines `QuantumSemanticIgnitionFuture` as `+ Send` and requires
`QuantumSemanticIgnition: Send + Sync`.

`packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/vectorizer.rs`
now defines `WendaoHybridQueryVectorizerFuture` as `+ Send` and requires
`WendaoHybridQueryVectorizer: Send + Sync`.

Why this matters:

- the traited retrieval seams now preserve sendability instead of erasing it,
- caller runtimes can hold these futures across `.await` points inside
  multi-threaded executors.

### 4) Focused tests now guard thread-safety at compile time

New tests:

- `packages/rust/crates/xiuxian-vector/tests/test_send_sync.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_send_runtime.rs`

The tests verify:

- `VectorStore` is `Send + Sync`,
- `KeywordIndex` is `Send + Sync`,
- `WendaoVectorSemanticIgnition` and `WendaoVectorHybridSearcher` are
  `Send + Sync`,
- `WendaoVectorHybridSearcher::search_text(...)` produces a `Send` future.

Why this matters:

- the regression surface is now encoded directly in tests,
- future refactors cannot silently reintroduce `Rc`, `RefCell`, or non-`Send`
  boxed futures.

## Architectural Takeaways

### Async boundary traits must encode sendability explicitly

If a seam is intended to cross task or executor boundaries, `Send` belongs in
its future alias and trait contract, not as an accidental property of one
implementation.

### Avoid `Rc` and `RefCell` in long-lived shared runtime structs

These types are fine for single-threaded local state, but they are the wrong
primitives for reusable runtime components such as `VectorStore` and
`KeywordIndex`.

### Use compile-time tests for concurrency contracts

`assert_send_sync::<T>()` and `assert_send(future)` are low-cost, high-signal
regression guards for shared runtime infrastructure.

## Files Changed

- `packages/rust/crates/xiuxian-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/index_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/index/shared.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/index/init_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/index/write_ops.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_send_sync.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/hybrid_search/vectorizer.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_send_runtime.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-vector -p xiuxian-wendao -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence cargo check -p xiuxian-daochang --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-vector --test test_send_sync
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector --test test_send_runtime
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector --test test_text_hybrid_search
CARGO_TARGET_DIR=/tmp/xiuxian-send-convergence cargo clippy -p xiuxian-vector -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-daochang --message-format short` completed cleanly (`Finished dev profile ... in 28.11s`), proving the higher-level caller still compiles against the tightened seam bounds.
- `cargo nextest run -p xiuxian-vector --test test_send_sync` passed (`2 passed`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition` passed (`4 passed, 69 skipped`).
- `cargo nextest run -p xiuxian-wendao-vector --test test_send_runtime` passed (`2 passed`).
- `cargo nextest run -p xiuxian-wendao-vector --test test_text_hybrid_search` passed (`3 passed`).
- `cargo clippy -p xiuxian-vector -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed cleanly after fixing one `clippy::doc_markdown` warning in a comment.

## Next Step

The next clean slice is no longer about `Send`.

It is to provision or reuse a real Wendao semantic vector table/bootstrap path
so `xiuxian-daochang` can replace the caller-owned cosine scorer with the
shared `xiuxian-wendao-vector` runtime without rebuilding semantic state on
every request.
