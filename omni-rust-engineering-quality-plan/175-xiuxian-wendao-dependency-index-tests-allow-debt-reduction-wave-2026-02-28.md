# 175. Xiuxian-Wendao Dependency Index Tests Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_dependency_indexer.rs`
  - `tests/test_dependency_integration.rs`

## Why This Wave

After TOML-only migration, these two dependency-indexer test files still had
file-level `#![allow(...)]` suppression blocks. This wave removes those
suppression blocks and fixes root causes directly.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_dependency_indexer.rs`
   - `tests/test_dependency_integration.rs`

2. Root-cause cleanup in `tests/test_dependency_integration.rs`:
   - `doc_markdown` fix in module docs:
     - updated `indexer.rs build()` to `` `indexer.rs` `build()` ``.
   - `uninlined_format_args` cleanup:
     - `format!("{}/Cargo.toml", temp_root)` -> `format!("{temp_root}/Cargo.toml")`
     - same style cleanup for `src/` and `src/lib.rs` paths.
   - `needless_raw_string_hashes` cleanup:
     - converted `lib_content` from `r#"... "#` to `r"... "`.

No new suppression attributes were introduced.

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

- Result: pass
- Note: one intermediate warning (`needless_raw_string_hashes`) was fixed and
  rerun to clean pass.

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `28`
  - After this wave: `26`
  - Net reduction: `2` files

## Engineering Outcome

- Dependency-indexer test lane is now suppression-free in these two core files.
- Remaining suppression debt is now concentrated in larger LinkGraph and CLI
  scenario test files.
