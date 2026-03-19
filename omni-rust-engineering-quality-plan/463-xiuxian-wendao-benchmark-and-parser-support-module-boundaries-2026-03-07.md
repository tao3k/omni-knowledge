# 463. Xiuxian Wendao Benchmark And Parser Support Module Boundaries

Date: 2026-03-07

## Scope

This shard records the modular cleanup of the benchmark and parser test families
so their `mod.rs` files no longer host helper implementations.

## Why This Change Was Needed

Several test families still used `mod.rs` as a mixed interface/helper bucket:

- `test_cargo_benchmark`
- `test_pyproject_benchmark`
- `test_symbols_benchmark`
- `parser_contracts`

Those module roots carried shared fixture generators, benchmark budgets, string
builders, and path helpers. That structure violated the repository rule that
`mod.rs` should remain interface-only.

## What Changed

### 1) Restored benchmark module roots to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/mod.rs`

These module roots now only declare child modules.

### 2) Extracted benchmark helpers into dedicated support modules

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/support.rs`

These support modules now own benchmark budgets, fixture generators, string
formatting helpers, and shared benchmark constants.

### 3) Localized imports inside benchmark child files

Updated all child benchmark files to import only the helpers and domain APIs
they actually use instead of depending on `use super::*;`.

### 4) Restored `parser_contracts/mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/mod.rs`

Added:

- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/support.rs`

The fixture-path helper now lives in `support.rs`, and `org.rs` imports it
explicitly.

## Architectural Takeaways

- Benchmark suites benefit from the same modular discipline as normal tests.
  Shared generators and performance-budget helpers belong in `support.rs`, not
  in `mod.rs`.
- `mod.rs` should describe module boundaries, not provide ambient helpers.
- Explicit child-file imports make benchmark dependencies visible and prevent
  hidden coupling to module-root implementation details.
- Even tiny helper functions such as fixture-path builders should move out of
  `mod.rs` once a parser or benchmark family grows beyond a single file.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/cargo_toml_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/complex_dependency_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/parsing_vs_io_overhead.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/workspace_cargo_toml_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/minimal_pyproject_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/mixed_pyproject_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/pyproject_extras_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/pyproject_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/regex_fallback_parsing_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/mixed_symbol_extraction_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/python_symbol_extraction_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/rust_symbol_extraction_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/symbol_index_search_performance.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/org.rs`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_cargo_benchmark --test test_pyproject_benchmark --test test_symbols_benchmark --test test_parser_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The targeted `cargo nextest run ...` passed (`19 passed, 1 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New support modules:
  - `packages/rust/crates/xiuxian-wendao/tests/test_cargo_benchmark/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_pyproject_benchmark/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_symbols_benchmark/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/parser_contracts/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/463-xiuxian-wendao-benchmark-and-parser-support-module-boundaries-2026-03-07.md`
