# Omni Vector Core Constructor Collapsible-If Cleanup (2026-02-26)

## Objective

Further reduce suppression debt in `xiuxian-vector` core initialization by
removing `clippy::collapsible_if` from `VectorStore::new` without changing API
shape.

## Scope

### Changed file

- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`

### What changed

1. Replaced nested directory-initialization condition in `VectorStore::new`
   with a let-chain guard:
   - `path != ":memory:"`
   - `let Some(parent) = base_path.parent()`
   - `!parent.exists()`
2. Removed `clippy::collapsible_if` from the function-level suppression list.
3. Kept `clippy::unused_async` temporarily because constructor API is still
   async-compatible in current call chains.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`
- `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-vector/src | sort`

Result: all checks passed; `ops/core.rs` no longer carries
`collapsible_if` suppression.

## Outcome

Core constructor control flow is cleaner and suppression scope is narrower,
while retaining backward-compatible async constructor call patterns.
