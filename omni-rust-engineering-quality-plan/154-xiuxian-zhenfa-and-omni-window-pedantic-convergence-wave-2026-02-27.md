# Xiuxian-Zhenfa and Omni-Window Pedantic Convergence Wave (2026-02-27)

## Scope

Continue strict Rust quality convergence for two crates:

1. `xiuxian-zhenfa`: finish test-lane pedantic cleanup without lint suppression.
2. `omni-window`: close remaining pedantic doc-markdown warning and revalidate.

## Implemented Changes

1. Replaced remaining `expect_err` panic paths with explicit `Result` pattern
   matching in `xiuxian-zhenfa` tests:
   - `packages/rust/crates/xiuxian-zhenfa/tests/test_gateway.rs`
   - `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry.rs`
2. Kept lifetime contracts explicit for mock identifiers in native registry
   tests:
   - `packages/rust/crates/xiuxian-zhenfa/tests/test_native_registry.rs`
   - Updated `EchoTool::id()` signature to return `&'static str`.
3. Fixed macro test warnings in
   `packages/rust/crates/xiuxian-zhenfa/tests/test_zhenfa_tool_macro.rs`:
   - `unnecessary_wraps`: cache-key function now returns `None` for empty
     values and `Some(...)` for valid keys.
   - `unused_async`: added explicit async yield points in macro-driven async
     handlers, preserving async contract semantics while satisfying pedantic
     checks.
4. Finalized `omni-window` test docs markdown formatting:
   - `packages/rust/crates/omni-window/tests/test_window.rs`
   - Wrapped `SessionWindow` in backticks in module docs.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-zhenfa
cargo fmt -p omni-window

CARGO_TARGET_DIR=target/clippy-zhenfa cargo clippy -p xiuxian-zhenfa --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
CARGO_TARGET_DIR=target/clippy-window cargo clippy -p omni-window --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines

CARGO_TARGET_DIR=target/nextest-zhenfa cargo nextest run -p xiuxian-zhenfa
CARGO_TARGET_DIR=target/nextest-window cargo nextest run -p omni-window
```

Results:

- Strict clippy passed for both crates with exit `0`.
- `xiuxian-zhenfa` nextest passed:
  - `28 tests run: 28 passed, 0 skipped`.
- `omni-window` nextest passed:
  - `3 tests run: 3 passed, 0 skipped`.

## Outcome

Both `xiuxian-zhenfa` and `omni-window` are now clean under strict pedantic
plus structural lint gates for this wave, with no broad lint suppression added.
The test suites are aligned with the workspace no-`expect`/no-suppression
engineering standard and remain fully green under `cargo nextest`.
