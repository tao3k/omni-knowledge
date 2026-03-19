# 501. Xiuxian AST Generic tests/mod.rs Entrypoint Removal

Date: 2026-03-08

## Scope

This shard records the removal of the generic `packages/rust/crates/xiuxian-ast/tests/mod.rs`
entrypoint.

## Why This Change Was Needed

The file was a legacy integration-test entrypoint that re-exported other test
files as modules:

- `test_extract`
- `test_item`
- `test_lang`

That structure is both generic and redundant. Each of those files already
exists as its own integration-test binary, so the generic `mod.rs` added an
extra package-top entrypoint without contributing distinct coverage.

## What Changed

Removed `packages/rust/crates/xiuxian-ast/tests/mod.rs`.

The crate now relies on explicit integration-test binaries only, matching the
same standard used in the broader workspace:

- no generic `tests/mod.rs` launcher,
- one explicit test binary per entrypoint,
- directory modules used only behind those explicit test binaries.

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

- Generic `tests/mod.rs` entrypoints should be removed when they merely wrap
  explicit test binaries.
- Integration-test structure should be discoverable from file names alone;
  generic launchers obscure intent and can introduce redundant compilation or
  execution paths.
- Explicit entrypoints plus directory modules provide the cleanest long-term
  structure for growing Rust integration suites.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-ast/tests/mod.rs`
