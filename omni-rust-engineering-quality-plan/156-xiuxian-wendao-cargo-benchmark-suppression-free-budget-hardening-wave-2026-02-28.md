# 156. Xiuxian-Wendao Cargo Benchmark Suppression-Free Budget Hardening Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus: `tests/test_cargo_benchmark*`
- Goal: remove file-level lint suppressions and converge to root-cause fixes that remain stable under `nextest` parallel execution.

## Why This Wave

`test_cargo_benchmark` still depended on file-level `#![allow(...)]` blocks (including `missing_docs`, `doc_markdown`, `format_push_string`, and `too_many_lines`) and hard-coded wall-clock thresholds. This conflicted with the project rule of suppression-free quality convergence.

## Changes Implemented

1. Removed all file-level suppression blocks from:
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/cargo_toml_parsing_performance.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/complex_dependency_parsing_performance.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/parsing_vs_io_overhead.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/workspace_cargo_toml_parsing_performance.rs`

2. Added a shared benchmark budget policy in `test_cargo_benchmark/mod.rs`:
   - `OMNI_WENDAO_BENCH_SLACK_FACTOR` (default `2.0`)
   - automatic `nextest` contention multiplier via `NEXTEST_RUN_ID`
   - `benchmark_budget(local, ci)` helper for consistent thresholding

3. Reworked string construction helpers in `test_cargo_benchmark/mod.rs`:
   - replaced `push_str(&format!(...))` with `append_format(..., format_args!(...))`
   - reduced clippy pressure (`format_push_string`, format-style pedantic warnings) without adding any `allow`

4. Updated performance test assertions to use budget helper:
   - local/CI-aware timing budgets now replace fixed thresholds
   - assertion messages now print actual runtime and active budget

5. Removed one additional explicit `expect_err` usage in:
   - `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools.rs`
   - replaced with explicit `Result` matching and failure message carrying unexpected output

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Engineering Outcome

- `test_cargo_benchmark` moved to suppression-free enforcement with stable, environment-aware benchmark budgets.
- The cargo benchmark lane now aligns with the same quality posture already applied to `test_symbols_benchmark` and `test_pyproject_benchmark`.

## Next Slice

- Continue the same pattern for remaining `xiuxian-wendao/tests` files that still carry file-level `#![allow(...)]` markers (starting with non-benchmark high-frequency lanes to maximize warning-debt burn-down per edit).
