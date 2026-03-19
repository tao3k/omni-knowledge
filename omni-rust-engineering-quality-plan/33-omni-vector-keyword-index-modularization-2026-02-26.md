# Omni Vector Keyword Index Modularization (2026-02-26)

## Objective

Reduce production-code complexity in `xiuxian-vector` keyword retrieval by
decomposing `keyword/index.rs` into focused modules while preserving behavior
and keeping strict pedantic quality gates clean.

## Scope

### Before

- `packages/rust/crates/xiuxian-vector/src/keyword/index.rs` was a single
  implementation file (~510 lines) mixing schema/init, write paths, search
  paths, and point-lookup logic.

### After

- `packages/rust/crates/xiuxian-vector/src/keyword/index.rs` is now a thin entry
  file (5 lines) that composes focused files:
  - `packages/rust/crates/xiuxian-vector/src/keyword/index/shared.rs`
  - `packages/rust/crates/xiuxian-vector/src/keyword/index/init_ops.rs`
  - `packages/rust/crates/xiuxian-vector/src/keyword/index/write_ops.rs`
  - `packages/rust/crates/xiuxian-vector/src/keyword/index/search_ops.rs`
  - `packages/rust/crates/xiuxian-vector/src/keyword/index/read_ops.rs`

## Design Notes

1. No API behavior change: public `KeywordIndex` constructor and operation
   methods are preserved.
2. Split by responsibility:
   - schema/index initialization + migration,
   - write and batch-upsert operations,
   - ranked search operations,
   - exact lookup by tool name.
3. Removed file-local `doc_markdown` suppression from the previous monolithic
   file and retained pedantic compliance through documentation cleanup.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

Keyword retrieval code now has clearer physical boundaries, making future
typed-error evolution, targeted test additions, and maintenance safer and
faster.
