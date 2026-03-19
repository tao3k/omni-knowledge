# Omni Vector Ops Collapsible-If Cleanup (2026-02-26)

## Objective

Continue suppression-debt reduction in `xiuxian-vector` by removing additional
`clippy::collapsible_if` allowances in ops and checkpoint-adjacent paths.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl/delete_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/column_read.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/maintenance.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/write_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/read_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/lifecycle.rs`

### What changed

1. Removed `#[allow(clippy::collapsible_if)]` from `delete_by_file_path`.
2. Removed `#[allow(clippy::collapsible_if)]` from `get_utf8_at`.
3. Removed `#[allow(clippy::collapsible_if)]` from
   `auto_index_if_needed_with_thresholds`.
4. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::save_checkpoint`.
5. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::get_by_id`.
6. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::get_or_create_dataset`.
7. Refactored all above call paths to let-chain based control flow where
   applicable, preserving existing behavior.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`
- `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-vector/src | sort`

Result: all checks passed and the targeted `collapsible_if` suppressions were
removed.

## Outcome

`xiuxian-vector` now has a smaller suppression surface and cleaner control-flow
style in core ops/checkpoint implementations while retaining full test pass.
