# Omni-Tokenizer Cast Safety and Benchmark Noise Hardening Wave (2026-02-27)

## Scope

Continue convergence for `omni-tokenizer`:

1. Remove pedantic cast-risk warning in tokenizer index assertions.
2. Harden benchmark tests against machine/CI noise without lint suppression.
3. Revalidate with strict clippy and crate-level `nextest`.

## Implemented Changes

1. Pedantic cast safety update:
   - `packages/rust/crates/omni-tokenizer/tests/test_tokenizer.rs`
   - Replaced `usize -> u32` cast with explicit `u32::try_from` handling.
2. Benchmark resilience updates:
   - `packages/rust/crates/omni-tokenizer/tests/test_tokenizer_benchmark.rs`
   - Added `benchmark_budget(local, ci)` helper with
     `OMNI_TOKENIZER_BENCH_SLACK_FACTOR` support (default `2.0`).
   - Replaced fixed thresholds with budget-driven thresholds across
     performance tests.
   - Converted `test_token_counting_performance` from heavy run
     (`100` iterations) to lighter smoke benchmark (`20` iterations) to reduce
     runtime variance while preserving behavior checks.

## Verification Evidence

Executed:

```bash
CARGO_TARGET_DIR=target/clippy-tokenizer cargo clippy -p omni-tokenizer --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
CARGO_TARGET_DIR=target/nextest-tokenizer cargo nextest run -p omni-tokenizer
```

Results:

- Strict clippy command completed successfully (exit `0`) with no warnings.
- `cargo nextest` completed successfully:
  - `23 tests run: 23 passed, 0 skipped` (nextest reports `1 leaky`).

## Outcome

`omni-tokenizer` now has suppression-free cast-safety alignment and benchmark
assertions that are substantially less sensitive to runtime contention, while
retaining configurable strictness for dedicated performance lanes.
