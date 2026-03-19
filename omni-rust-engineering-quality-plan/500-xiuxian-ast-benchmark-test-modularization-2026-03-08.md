# 500. Xiuxian AST Benchmark Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_ast_benchmark.rs` integration test in `xiuxian-ast`.

## Why This Change Was Needed

The original benchmark file mixed several distinct concerns into one large test
module:

- synthetic Python source generation,
- benchmark budget calculation,
- ast-grep performance tests,
- tree-sitter performance tests,
- pattern-matching correctness checks.

This made the benchmark suite hard to maintain and obscured the boundary between
fixture generation, performance thresholds, and correctness assertions.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for synthetic source generation and benchmark budget helpers,
- `ast_grep.rs` for ast-grep performance coverage,
- `tree_sitter.rs` for tree-sitter performance coverage,
- `correctness.rs` for structural correctness assertions.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-ast --tests
cargo nextest run -p xiuxian-ast --no-fail-fast
cargo clippy -p xiuxian-ast -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-ast --tests` passed.
- `cargo nextest run -p xiuxian-ast --no-fail-fast` passed (`74 passed, 0 skipped`).
- `cargo clippy -p xiuxian-ast -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Benchmark suites should isolate fixture generation, timing budget logic,
  performance cases, and correctness checks into explicit modules.
- Local support modules are the right place for synthetic corpus generation and
  environment-sensitive budget calculations.
- Correctness assertions should not be buried inside performance-oriented test
  modules.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/mod.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/support.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/ast_grep.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/tree_sitter.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_ast_benchmark/correctness.rs`
