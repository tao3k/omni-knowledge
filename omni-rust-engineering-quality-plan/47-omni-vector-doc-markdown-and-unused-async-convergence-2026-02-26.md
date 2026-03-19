# Omni-Vector Doc-Markdown and Unused-Async Convergence (2026-02-26)

## Scope

This wave continued the codex-aligned Rust quality track for `xiuxian-vector` with
two explicit goals:

1. Remove remaining file-level `doc_markdown` suppressions and convert docs to
   lint-clean form.
2. Remove `unused_async` suppressions via root-cause API/runtime fixes, not
   lint silencing.

## Changes Implemented

### 1) Removed file-level `doc_markdown` suppressions

Removed suppression attributes from:

- `packages/rust/crates/xiuxian-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/maintenance.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/migration.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/scalar.rs`

Then ran:

```bash
cargo clippy -p xiuxian-vector --all-targets --fix --allow-dirty --allow-staged -- -W clippy::doc_markdown
```

This applied targeted doc fixes (backticks and symbol formatting) across
`xiuxian-vector` modules without adding new `allow(...)` attributes.

### 2) Removed `unused_async` suppressions via runtime/API cleanup

#### Constructor path uses real async filesystem I/O

- `packages/rust/crates/xiuxian-vector/src/checkpoint/store.rs`
  - `CheckpointStore::new(...)` now uses `tokio::fs::create_dir_all(...).await`.
- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`
  - `VectorStore::new(...)` now uses `tokio::fs::create_dir_all(...).await`.

This preserves async constructor signatures while removing fake-async bodies.

#### Query metrics API cleanup

- `packages/rust/crates/xiuxian-vector/src/ops/observability.rs`
  - `get_query_metrics(...)` changed from:
    - `async fn -> Result<QueryMetrics, VectorStoreError>`
  - to:
    - `fn -> QueryMetrics`
  - Added `#[must_use]` per pedantic guidance.

Updated call sites:

- `packages/rust/crates/xiuxian-vector/tests/test_observability.rs`
- `packages/rust/bindings/python/src/vector/store.rs`

### 3) Removed final `arc_with_non_send_sync` suppression

- `packages/rust/crates/xiuxian-vector/src/lib.rs`
  - `keyword_index` storage changed from `Option<Arc<KeywordIndex>>` to
    `Option<Rc<KeywordIndex>>`.
- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`
  - Removed `#[allow(clippy::arc_with_non_send_sync)]` from
    `enable_keyword_index`.
  - Index initialization now uses `Rc::new(...)`.

Rationale: keyword index state is single-owner runtime-local state in the
current architecture; using `Rc` matches the actual non-`Send`/non-`Sync`
semantics and removes misleading shared-thread intent.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-vector -p omni-core-rs
cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic
cargo check -p omni-core-rs
cargo test -p xiuxian-vector --tests
```

Notes:

- `cargo clippy -p omni-core-rs --all-targets -- -W clippy::pedantic` still
  fails due pre-existing historical lint debt (`unwrap_used`, doc style) outside
  this change scope.
- The binding-side compile contract for this change is validated via
  `cargo check -p omni-core-rs`.

## Outcome

- `xiuxian-vector/src` suppression count is now `0`.
- All changes in this wave are root-cause fixes (docs/runtime/API), not
  warning suppression.
