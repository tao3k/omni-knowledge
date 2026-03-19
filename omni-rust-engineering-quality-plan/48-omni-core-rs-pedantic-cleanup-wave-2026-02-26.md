# Omni-Core-RS Pedantic Cleanup Wave (2026-02-26)

## Scope

This wave continued the codex-aligned Rust quality track by reducing
`omni-core-rs` pedantic lint debt through root-cause code fixes in tests and
test-adjacent modules.

Target areas:

- `packages/rust/bindings/python/tests/test_skill_index.rs`
- `packages/rust/bindings/python/src/ast/mod.rs` (test module)
- `packages/rust/bindings/python/src/tags.rs` (test module)

## Changes Implemented

### 1) `test_skill_index` lint cleanup (no suppression)

File:

- `packages/rust/bindings/python/tests/test_skill_index.rs`

Actions:

- Removed all `unwrap()` usage and replaced with structured error propagation.
- Introduced file-level test result alias:
  - `type TestResult = std::result::Result<(), Box<dyn std::error::Error>>;`
- Converted all test functions to return `TestResult` and use `?`.
- Fixed documentation style (`doc_markdown`) by adding backticks to API names.
- Replaced manual empty string creation (`"".to_string()`) with `String::new()`.
- Replaced `Default::default()` on `annotations` with
  `ToolAnnotations::default()`.
- Normalized imports to remove `items_after_statements` warnings.

### 2) `ast/mod.rs` test unwrap cleanup

File:

- `packages/rust/bindings/python/src/ast/mod.rs`

Actions:

- Updated `test_extract_items_python` to return
  `Result<(), Box<dyn std::error::Error>>`.
- Replaced unwraps with fallible flow:
  - `py_extract_items(...).map_err(|e| std::io::Error::other(...))?`
  - `serde_json::from_str(&json)?`

### 3) `tags.rs` raw string pedantic cleanup

File:

- `packages/rust/bindings/python/src/tags.rs`

Actions:

- Replaced `r#"...\"#` with `r"..."` for the symbol-outline test fixture to
  satisfy `clippy::needless_raw_string_hashes`.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p omni-core-rs
cargo clippy -p omni-core-rs --test test_skill_index -- -W clippy::pedantic
cargo clippy -p omni-core-rs --all-targets -- -W clippy::pedantic
cargo test -p omni-core-rs --test test_skill_index
cargo check -p omni-core-rs
```

Observed environment-specific limitation:

- `cargo test -p omni-core-rs --tests` runs integration tests but still invokes
  crate unit-test binary (`src/lib.rs`) and aborts in this environment with:
  - `dyld: symbol not found in flat namespace '_PyBool_Type'`
- This is a known runtime-linking issue for Python symbol resolution and is
  outside this lint cleanup scope.

## Outcome

- `omni-core-rs` now passes strict pedantic clippy for all targets in this
  workspace environment.
- `test_skill_index` test suite is lint-clean and passes (`10/10`).
- This wave used root-cause fixes only; no lint suppression attributes were
  added.
