# 417. Daochang Zhenfa Hybrid Query Vectorizer Adapter

Date: 2026-03-07

## Scope

This shard records the ninth pure-Rust hybrid-retriever slice.

The slice adds the first higher-level runtime adapter that implements
`xiuxian_wendao_vector::WendaoHybridQueryVectorizer` over the existing
`xiuxian-daochang` embedding client.

## Why This Change Was Needed

Phase 8 added a text-first hybrid-search seam in `xiuxian-wendao-vector`, but
it still relied on test-only stub vectorizers.

That left one real integration gap:

- Wendao planning could now defer vectorization,
- but `xiuxian-daochang` still had no clean, modular way to provide query
  embeddings to that seam.

Without this adapter, the new Rust contract would remain theoretical.

## What Changed

### 1) `xiuxian-daochang` now depends on `xiuxian-wendao-vector`

`packages/rust/crates/xiuxian-daochang/Cargo.toml` now includes:

- `xiuxian-wendao-vector`

Why this matters:

- the caller crate now explicitly owns the bridge into the hybrid-search seam,
- dependency direction remains clean: `xiuxian-daochang -> xiuxian-wendao-vector`
  rather than leaking caller concerns back down into Wendao.

### 2) Zhenfa gained a dedicated `query_vectorizer` module

`packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/query_vectorizer.rs`
now defines:

- `ZhenfaEmbeddingQueryVectorizer`
- `ZhenfaEmbeddingQueryVectorizeError`

The adapter wraps `Arc<EmbeddingClient>` and implements
`WendaoHybridQueryVectorizer` by calling:

- `EmbeddingClient::embed_with_model`

Why this matters:

- the adapter is isolated in its own domain-specific module instead of inflating
  `bridge.rs`,
- query vectorization stays aligned with existing daochang embedding ownership,
- the integration stays pure Rust and backend-agnostic from Wendao's point of
  view.

### 3) The adapter enforces a usable vector contract

The new adapter rejects three invalid states:

- blank query text,
- missing embedding response,
- empty embedding vector.

Why this matters:

- hybrid-search callers now get a typed runtime contract instead of silently
  forwarding bad vectors,
- the boundary is defensive before semantic ignition begins,
- error semantics are explicit and testable.

### 4) Integration tests now validate the adapter as a black box

New tests live under:

- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/query_vectorizer_tests.rs`

They verify:

- query text is trimmed before dispatch,
- model override is forwarded to `/embed/batch`,
- blank queries fail before any network call,
- missing vectors fail clearly,
- empty vectors fail clearly.

Why this matters:

- the adapter is validated through the same HTTP-facing embedding client it will
  use in production,
- the boundary is tested where ownership, normalization, and transport meet,
- the tests live in package-top `tests/`, matching repository standards.

### 5) Validation surfaced and fixed an existing bootstrap borrow conflict

During crate validation, `cargo check -p xiuxian-daochang` surfaced an existing
borrow checker failure in:

- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/native_tools.rs`

The failure came from two closures trying to mutably capture `native_tools`
inside `map_or_else(...)`.

The convergence fix replaced the dual-closure pattern with a direct `match`.

Why this matters:

- the crate now remains buildable while the hybrid-search adapter work lands,
- the fix improves ownership clarity instead of hiding the problem.

## Architectural Takeaways

### Put caller-specific runtime adapters in the caller crate

`xiuxian-wendao-vector` should define the integration contract.
`xiuxian-daochang` should own the concrete adapter because it already owns the
embedding client.

### Modularize by capability, not by convenience

The adapter lives in `agent/zhenfa/query_vectorizer.rs`, not in `bridge.rs`.
That keeps tool bridging and embedding adaptation as separate concerns.

### Validate boundary quality with black-box tests

The adapter tests do not reach into private internals.
They assert observable transport and error behavior through package-level
integration tests.

## Files Changed

- `packages/rust/crates/xiuxian-daochang/Cargo.toml`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/query_vectorizer.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/native_tools.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/zhenfa_query_vectorizer.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_zhenfa_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/query_vectorizer_tests.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-daochang
CARGO_TARGET_DIR=/tmp/xiuxian-daochang-zhenfa-vectorizer cargo check -p xiuxian-daochang
CARGO_TARGET_DIR=/tmp/xiuxian-daochang-zhenfa-vectorizer cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-daochang-zhenfa-vectorizer NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit
```

Observed outcomes:

- `cargo check -p xiuxian-daochang` completed cleanly after converging the
  existing `native_tools.rs` borrow conflict.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` completed
  cleanly.
- `cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit` passed
  (`19 passed, 1 skipped`).

## Next Step

The next clean slice is to wire `ZhenfaEmbeddingQueryVectorizer` into one real
hybrid-search caller path so text-first hybrid retrieval can run end-to-end from
`xiuxian-daochang` without test-only adapter construction.
