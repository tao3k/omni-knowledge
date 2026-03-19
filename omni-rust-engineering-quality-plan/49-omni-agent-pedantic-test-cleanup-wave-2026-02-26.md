# 修仙道场 (Xiuxian Daochang) Pedantic Test Cleanup Wave (2026-02-26)

## Scope

This wave continued codex-aligned Rust quality convergence for `xiuxian-daochang`
with a focused test-lane cleanup under strict pedantic clippy.

Target areas:

- `packages/rust/crates/xiuxian-daochang/src/config/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/test_native_tools.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/native_tools_zhixing.rs`
- `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`

## Changes Implemented

### 1) Config unit tests: root-cause unwrap/expect removal

File:

- `packages/rust/crates/xiuxian-daochang/src/config/tests.rs`

Actions:

- Replaced nested `mod tests` layout (module inception pattern) with
  top-level test module file content.
- Removed all `unwrap()`/`expect()` usage.
- Added explicit test result type:
  - `type TestResult = std::result::Result<(), Box<dyn std::error::Error>>`
- Introduced `ConfigHomeGuard` with `Drop`-based restoration for
  `PRJ_CONFIG_HOME` to guarantee environment cleanup.
- Replaced optional-field unwrap assertions with `as_deref()` comparisons.

### 2) Native tool integration test cleanup

File:

- `packages/rust/crates/xiuxian-daochang/tests/test_native_tools.rs`

Actions:

- Updated trait methods returning literals to `&'static str`.
- Replaced `expect()` calls with fallible test flow.
- Converted async test to `anyhow::Result<()>` and `?` propagation.

### 3) Additional pedantic warning cleanup

Files:

- `packages/rust/crates/xiuxian-daochang/tests/agent/native_tools_zhixing.rs`
- `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`

Actions:

- Replaced `MockManifestation::default()` (unit struct) with
  `MockManifestation`.
- Rewrote `Option` chain from `map(...).unwrap_or_else(...)` to
  `map_or_else(...)` per pedantic guidance.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --test test_native_tools
cargo test -p xiuxian-daochang --lib test_unified_config_loading_priority
cargo test -p xiuxian-daochang --lib test_modular_wendao_fallback
```

## Outcome

- `xiuxian-daochang` now passes strict pedantic clippy for all targets in this
  workspace run.
- Config and native-tool test paths were migrated from panic-style testing to
  structured error propagation.
- No new lint suppression attributes were introduced.
