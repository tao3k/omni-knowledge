# Omni Vector Skill Search Modularization (2026-02-26)

## Objective

Reduce structural complexity in `xiuxian-vector` tool-search production code by
decomposing `skill/ops_impl/search.rs` into focused modules, while preserving
runtime behavior and strict pedantic quality gates.

## Scope

### Before

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs` was a single
  implementation file (~660 lines).

### After

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs` is now a thin
  entry file (4 lines) composing focused files:
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search/api.rs`
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search/types.rs`
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search/vector_rows.rs`
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search/ranking.rs`

## Design Notes

1. No API behavior change: public `VectorStore` search methods remain intact.
2. Split by concern:
   - API/orchestration flow (`api.rs`),
   - search batch/data carrier types (`types.rs`),
   - row extraction and vector-result assembly (`vector_rows.rs`),
   - query parsing and rerank heuristics (`ranking.rs`).
3. Removed lint suppression from the old file boundary and kept pedantic checks
   fully clean without introducing new `allow` directives.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

The skill-search layer now has clearer module boundaries for future typed-error
isolation, ranking-heuristic evolution, and targeted testing with lower
maintenance cost.
