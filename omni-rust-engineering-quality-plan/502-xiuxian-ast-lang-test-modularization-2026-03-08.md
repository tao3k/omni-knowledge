# 502. Xiuxian AST Lang Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the compact but mixed `test_lang.rs`
integration test in `xiuxian-ast`.

## Why This Change Was Needed

Even though the file was small, it still mixed two distinct language-surface
contracts in one top-level file:

- extension-to-language conversion,
- reverse extension listing behavior.

The workspace rule is to split by concern, not line count, so this file still
qualified for normalization.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-ast/tests/test_lang.rs` so it now acts as
an explicit integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-ast/tests/test_lang/` with focused modules:

- `mod.rs` for the module graph only,
- `conversions.rs` for `from_extension` and `try_from` behavior,
- `extensions.rs` for extension-list coverage.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-ast --tests
cargo nextest run -p xiuxian-ast --test test_lang --no-fail-fast
cargo clippy -p xiuxian-ast -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-ast --tests` passed.
- `cargo nextest run -p xiuxian-ast --test test_lang --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-ast -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Small test files still need modularization when they cover multiple distinct
  contracts.
- Explicit launchers plus directory modules scale better than leaving small
  mixed files in place simply because they are short.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-ast/tests/test_lang.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_lang/mod.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_lang/conversions.rs`
- `packages/rust/crates/xiuxian-ast/tests/test_lang/extensions.rs`
