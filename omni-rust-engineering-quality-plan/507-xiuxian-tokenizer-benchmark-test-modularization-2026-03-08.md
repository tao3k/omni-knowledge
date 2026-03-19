# 507. Xiuxian Tokenizer Benchmark Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_tokenizer_benchmark.rs` integration test in `xiuxian-tokenizer`.

## Why This Change Was Needed

The original benchmark file mixed several distinct concerns into one large test
module:

- synthetic text/code/JSON generation,
- benchmark budget logic,
- throughput benchmarks,
- format-specific benchmarks,
- truncation performance,
- correctness assertions.

This obscured the boundary between fixture generation, budget policy, and the
actual benchmark surfaces.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for benchmark budget logic, warm-up, and synthetic fixture generation,
- `throughput.rs` for token counting, large text, batch, varying-size, and wrapper throughput,
- `formats.rs` for code and JSON tokenization benchmarks,
- `truncation.rs` for truncate performance,
- `correctness.rs` for functional correctness assertions.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-tokenizer --tests
cargo nextest run -p xiuxian-tokenizer --no-fail-fast
cargo clippy -p xiuxian-tokenizer -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-tokenizer --tests` passed.
- `cargo nextest run -p xiuxian-tokenizer --no-fail-fast` passed (`16 passed, 0 skipped`).
- `cargo clippy -p xiuxian-tokenizer -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Benchmark suites should isolate support generation, throughput, format
  coverage, truncation, and correctness into explicit modules.
- Correctness assertions should not be buried inside performance-oriented test
  files.
- Thin entrypoints make long-lived benchmark suites easier to evolve without
  reopening a single monolithic file.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/support.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/throughput.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/formats.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/truncation.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer_benchmark/correctness.rs`
