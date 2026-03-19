# Omni Vector Admin Guards Lint-Debt Reduction (2026-02-26)

## Objective

Reduce suppression debt in `xiuxian-vector` admin guard paths by removing local
clippy suppressions through root-cause refactoring.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/guards.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/table_ops.rs`

### What changed

1. Removed file-level `#[allow(clippy::doc_markdown)]` in admin guards.
2. Removed method-level `#[allow(clippy::unused_self)]` by converting
   `ensure_non_reserved_column` into an associated guard function.
3. Updated call sites in table alteration operations to use
   `Self::ensure_non_reserved_column(...)`.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

Admin schema-guard logic now passes strict pedantic checks without those local
suppressions while preserving behavior and public API contracts.
