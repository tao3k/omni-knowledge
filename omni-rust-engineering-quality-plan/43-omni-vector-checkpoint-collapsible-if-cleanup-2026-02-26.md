# Omni Vector Checkpoint Collapsible-If Cleanup (2026-02-26)

## Objective

Reduce lint-suppression debt in `xiuxian-vector` checkpoint store by removing
`clippy::collapsible_if` allowances and replacing nested conditionals with
explicit let-chain control flow.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/write_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/read_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/lifecycle.rs`

### What changed

1. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::save_checkpoint`.
2. Refactored user-metadata extraction in `save_checkpoint` to a single
   let-chain condition.
3. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::get_by_id`.
4. Refactored row decode path in `get_by_id` into a single let-chain check.
5. Removed `#[allow(clippy::collapsible_if)]` from
   `CheckpointStore::get_or_create_dataset`.
6. Refactored cache fast-path check to let-chain (`!force_create && let Some(...)`).

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`
- `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-vector/src | sort`

Result: all checks passed; checkpoint-store `collapsible_if` suppressions were removed.

## Outcome

Checkpoint read/write/lifecycle paths are now lint-clean for this class of
control-flow issue, with behavior preserved under full crate test coverage.
