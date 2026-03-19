# Omni Vector Admin Impl Doc-Markdown Cleanup (2026-02-26)

## Objective

Reduce lint-suppression debt in `xiuxian-vector` admin implementation paths by
removing unnecessary `doc_markdown` suppressions and validating strict pedantic
compliance.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/table_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/delete_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/index_ops.rs`

### What changed

1. Removed file-level `#[allow(clippy::doc_markdown)]` in all three admin impl
   modules listed above.
2. Kept code behavior unchanged; this was suppression cleanup plus regression
   verification.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

`admin_impl` now has lower suppression debt and remains fully green under
strict pedantic lint and full test gates.
