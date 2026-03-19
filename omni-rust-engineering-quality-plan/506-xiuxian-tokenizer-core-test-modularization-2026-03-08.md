# 506. Xiuxian Tokenizer Core Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_tokenizer.rs`
integration test in `xiuxian-tokenizer`.

## Why This Change Was Needed

The original file mixed several tokenizer surfaces in one entrypoint:

- token counting,
- model-specific counting,
- truncation behavior,
- text chunking behavior.

These are related APIs, but they should not remain bundled in one top-level
implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer.rs` so it
now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer/` with
focused modules:

- `mod.rs` for the module graph only,
- `counting.rs` for plain and model-specific token counting,
- `truncation.rs` for truncate behavior,
- `chunking.rs` for chunking behavior.

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

- Counting, truncation, and chunking behaviors should each have their own test
  module even in relatively small tokenizer crates.
- Thin entrypoints keep the tokenizer suite aligned with the same package-level
  structure used elsewhere in the workspace.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer/mod.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer/counting.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer/truncation.rs`
- `packages/rust/crates/xiuxian-tokenizer/tests/test_tokenizer/chunking.rs`
