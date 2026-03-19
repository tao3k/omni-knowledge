# 499. Xiuxian AST Python Tree-Sitter Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_python_tree_sitter.rs` integration test in `xiuxian-ast`.

## Why This Change Was Needed

The original file mixed several decorator-parsing surfaces into one entrypoint:

- large multi-line decorator argument parsing,
- triple-quoted description parsing with commas,
- multi-function decorator parsing,
- negative behavior for undecorated functions.

These behaviors are related, but they are separate enough that they should not
remain in one top-level test file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter.rs` so
it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for a local assertion helper around optional references,
- `decorator_arguments.rs` for complex decorator and multi-function parsing,
- `negative.rs` for undecorated-function behavior.

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

- Complex decorator parsing and negative-path validation should not share one
  mixed implementation file.
- Small local helpers are acceptable in a dedicated support module when they
  clarify intent and avoid repeated unwrap-style assertions.
- Thin entrypoints keep parser integration suites aligned with the same test
  structure used across the rest of the workspace.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter/mod.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter/support.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter/decorator_arguments.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_python_tree_sitter/negative.rs`
