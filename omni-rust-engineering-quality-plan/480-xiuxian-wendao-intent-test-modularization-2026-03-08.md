# 480. Xiuxian Wendao Intent Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_intent.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file bundled several distinct intent-extraction concerns into
one top-level binary:

- action/target inference,
- keyword and context extraction,
- dotted-command tokenization.

That structure was still manageable in size, but it mixed separate behavior
clusters that should evolve independently.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_intent.rs` so it now
acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_intent/` with focused
modules:

- `mod.rs` for the module graph only,
- `action_target.rs` for action and target inference coverage,
- `keywords.rs` for keyword and context extraction behavior,
- `tokenization.rs` for dotted-command tokenization.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_intent --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_intent --no-fail-fast`
  passed (`14 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Intent extraction tests benefit from the same modularity rules as larger
  integration suites; mixed behavior clusters should still be separated.
- Tokenization edge cases deserve their own focused module so they do not get
  buried under generic action-target assertions.
- Keeping the launcher thin preserves a stable public test binary while the
  internal test structure remains easy to extend.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_intent.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_intent/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_intent/action_target.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_intent/keywords.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_intent/tokenization.rs`
