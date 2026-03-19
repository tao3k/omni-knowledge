# Omni Vector Skill Indexing Lint-Debt Reduction (2026-02-26)

## Objective

Reduce suppression debt in `xiuxian-vector` skill indexing operations by removing
local clippy suppressions through root-cause code updates.

## Scope

### Changed file

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/indexing.rs`

### What changed

1. Removed file-level `#[allow(clippy::doc_markdown)]`.
2. Removed method-level `#[allow(clippy::unused_self)]` from:
   - `scan_unique_skill_tools`,
   - `scan_skill_tools_raw`.
3. Added stable diagnostic logs referencing `self.base_path`, so method receiver
   usage is meaningful without changing public signatures or behavior.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

Skill indexing operations now pass strict pedantic checks without those local
suppressions while retaining existing API contracts and runtime behavior.
