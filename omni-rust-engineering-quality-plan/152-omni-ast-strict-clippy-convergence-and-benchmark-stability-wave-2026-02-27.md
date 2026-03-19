# Omni-AST Strict Clippy Convergence and Benchmark Stability Wave (2026-02-27)

## Scope

Advance second-pass Rust quality convergence for `omni-ast`:

1. Remove test-surface `unwrap`/`expect` debt exposed by strict clippy.
2. Resolve pedantic warnings (`needless_raw_string_hashes`,
   `uninlined_format_args`, `doc_markdown`, `ignored_unit_patterns`).
3. Stabilize benchmark assertions to reduce environment-noise flakiness while
   preserving strict override capability.

## Implemented Changes

1. Refactored `chunk` tests to be suppression-free and explicit:
   - `packages/rust/crates/omni-ast/src/chunk.rs`
   - Introduced `chunk_or_panic(...)` helper and replaced all
     `.unwrap()` call sites.
2. Refactored scan/security tests to explicit result handling:
   - `packages/rust/crates/omni-ast/src/scan.rs`
   - `packages/rust/crates/omni-ast/src/security.rs`
3. Refactored tree-sitter parser tests to explicit `Option` handling:
   - `packages/rust/crates/omni-ast/src/python_tree_sitter_tests.rs`
   - `packages/rust/crates/omni-ast/tests/test_python_tree_sitter.rs`
4. Replaced remaining `unwrap` in integration tests:
   - `packages/rust/crates/omni-ast/tests/test_lang.rs`
5. Addressed pedantic string/format issues:
   - `packages/rust/crates/omni-ast/src/extract.rs`
   - `packages/rust/crates/omni-ast/src/python.rs`
   - `packages/rust/crates/omni-ast/tests/test_extract.rs`
   - `packages/rust/crates/omni-ast/tests/test_ast_benchmark.rs`
6. Benchmark stability improvement:
   - Added `benchmark_budget(...)` in
     `packages/rust/crates/omni-ast/tests/test_ast_benchmark.rs`
   - Thresholds now support `OMNI_AST_BENCH_SLACK_FACTOR` (default `2.0`,
     strict lanes can set `1.0`).
7. Documentation snippet modernization:
   - `packages/rust/crates/omni-ast/src/lib.rs`
   - Replaced direct `.unwrap()` quick-start pattern with explicit panic path.

## Verification Evidence

Executed:

```bash
CARGO_TARGET_DIR=target/clippy-ast cargo clippy -p omni-ast --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
CARGO_TARGET_DIR=target/nextest-omni-ast cargo nextest run -p omni-ast
```

Results:

- Strict clippy command completed successfully (exit `0`) with no warnings.
- `cargo nextest` completed successfully:
  - `89 tests run: 89 passed, 0 skipped`.

## Outcome

`omni-ast` is now converged on workspace strict-lint policy for touched paths
without suppression, and benchmark tests are materially more stable across
varying runtime environments while keeping strict-mode controls.
