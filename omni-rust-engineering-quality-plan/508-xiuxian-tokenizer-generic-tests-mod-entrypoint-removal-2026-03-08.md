# 508. Xiuxian Tokenizer Generic tests/mod.rs Entrypoint Removal

Date: 2026-03-08

## Scope

This shard records the removal of the generic `packages/rust/crates/xiuxian-tokenizer/tests/mod.rs`
entrypoint.

## Why This Change Was Needed

The file was a legacy generic launcher that re-exported `test_tokenizer` as a
module. That is redundant once explicit integration-test binaries exist, and it
obscures the test surface behind a generic `mod.rs` name.

## What Changed

Removed `packages/rust/crates/xiuxian-tokenizer/tests/mod.rs`.

The crate now relies on explicit integration-test binaries and directory modules
behind those binaries only.

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

- Generic `tests/mod.rs` launchers should be removed when they only duplicate
  explicit test binaries.
- Test discovery should be driven by explicit filenames, not generic wrappers.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-tokenizer/tests/mod.rs`
