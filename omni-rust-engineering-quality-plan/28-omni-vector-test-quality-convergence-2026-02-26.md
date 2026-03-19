# Omni Vector Test Quality Convergence (2026-02-26)

## Objective

Converge `xiuxian-vector` test code to strict `clippy::pedantic` quality without
introducing any lint suppressions.

This slice follows the project hard rule:

- fix root causes,
- do not weaken lint gates,
- keep behavior and snapshot contracts stable.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_path_handling.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_partitioning.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_fusion_snapshots.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_vector_index.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_entity_aware_benchmark.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_keyword_backend_quality.rs`

### Key engineering improvements

1. Panic-style test flows converted to `Result<()>` + `?` propagation.
2. Removed remaining `unwrap`/`expect` style usage in targeted tests.
3. Replaced manual error `match` patterns with `let...else` where clearer.
4. Resolved pedantic doc/format issues (`doc_markdown`, `uninlined_format_args`).
5. Eliminated precision-loss cast warnings in quality metrics logic by using
   explicit bounded conversions (`usize` -> `u16` -> `f32`) and `From`.
6. Replaced `map(...).unwrap_or(...)` with `map_or(...)`.
7. Refactored oversized snapshot test function paths in
   `test_keyword_backend_quality.rs` into reusable helpers:
   - document payload builders,
   - scenario builders,
   - backend comparison runners,
   - scene summary assembler.

## Verification Evidence

### Lint gates

- `cargo clippy -p xiuxian-vector --test test_path_handling -- -W clippy::pedantic`
- `cargo clippy -p xiuxian-vector --test test_maintenance -- -W clippy::pedantic`
- `cargo clippy -p xiuxian-vector --test test_keyword_backend_quality -- -W clippy::pedantic`
- `cargo clippy -p xiuxian-vector --tests -- -W clippy::pedantic`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`

Result: clean (no warnings/errors in the executed lanes).

### Test gates

- `cargo test -p xiuxian-vector --test test_path_handling -- --nocapture`
- `cargo test -p xiuxian-vector --test test_maintenance -- --nocapture`
- `cargo test -p xiuxian-vector --test test_keyword_backend_quality -- --nocapture`
- `cargo test -p xiuxian-vector --tests`

Result: all executed tests passed.

## Outcome

`xiuxian-vector` now has a clean strict-pedantic baseline across all targets and
tests in this execution window, with no lint suppression added.
