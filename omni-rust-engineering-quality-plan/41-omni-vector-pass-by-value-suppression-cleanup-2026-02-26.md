# Omni Vector Pass-By-Value Suppression Cleanup (2026-02-26)

## Objective

Reduce lint-suppression debt in `xiuxian-vector` by removing
`clippy::needless_pass_by_value` from production code and aligning function
implementations with actual ownership usage.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/weighted_rrf.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/writer_impl/ingest_ops.rs`

### What changed

1. Removed `#[allow(clippy::needless_pass_by_value)]` from
   `apply_weighted_rrf`.
2. Refactored keyword-fusion internals so `keyword_results` is consumed in
   merge phase and converted to a context map for later boost alignment, rather
   than only borrowed.
3. Removed `#[allow(clippy::needless_pass_by_value)]` from
   `VectorStore::add`.
4. Refactored `add` to prepare tool rows through an ownership-oriented pipeline
   (`prepared_tools`) before keyword indexing and Lance write batching.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`
- `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-vector/src | sort`

Result: all checks passed, and both target suppressions were removed.

## Outcome

`xiuxian-vector` now has lower suppression debt in keyword fusion and writer ingest
paths, while preserving behavior under full test coverage.
