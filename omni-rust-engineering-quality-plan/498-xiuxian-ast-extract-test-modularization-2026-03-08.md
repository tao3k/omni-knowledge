# 498. Xiuxian AST Extract Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_extract.rs`
integration test in `xiuxian-ast`.

## Why This Change Was Needed

The original file bundled several distinct extraction surfaces into one test
entrypoint:

- basic Python function extraction,
- line-number and capture behavior,
- language-specific extraction for Rust, Python classes, and JavaScript,
- value extraction and single-capture lookup behavior.

These concerns belong to the same extraction API surface, but they should not
remain mixed in one top-level file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-ast/tests/test_extract.rs` so it now acts
as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-ast/tests/test_extract/` with focused
modules:

- `mod.rs` for the module graph only,
- `support.rs` for reusable fixture sources and extraction helpers,
- `basic.rs` for Python extraction, capture, empty-result, and line-number behavior,
- `languages.rs` for Rust, class, and JavaScript extraction,
- `values.rs` for variable extraction and `extract()` behavior.

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

- Extraction tests should separate basic capture semantics, language-specific
  extraction, and value-oriented queries into focused modules.
- Shared source snippets belong in a local support module rather than being
  duplicated across test functions.
- Thin entrypoints keep extraction suites aligned with the package-wide test
  structure used elsewhere in the workspace.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-ast/tests/test_extract.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_extract/mod.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_extract/support.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_extract/basic.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_extract/languages.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_extract/values.rs`
