# 338. xiuxian-llm deepseek valkey init/ops split and failure-path tests (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/cache/valkey/*`
  - `src/llm/vision/deepseek/native/cache/mod.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/test_api.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_valkey_failures_unit.rs`
- Goal:
  - split deepseek valkey cache implementation into initialization and operation
    domains, then add top-level tests for invalid-url/connection-failure/timeout
    clamp paths.

## Implementation

1. Converted valkey cache from single file to directory module:
   - retired `native/cache/valkey.rs`.
   - added:
     - `native/cache/valkey/mod.rs` (entry + static cache + test hooks),
     - `native/cache/valkey/client_init.rs` (env/config parsing and client build),
     - `native/cache/valkey/ops.rs` (`GET`/`SETEX` behaviors and timeout application).
2. Separated concerns:
   - `client_init.rs`:
     - env/config read path,
     - timeout normalization (`>=1ms`),
     - client bootstrap failure handling.
   - `ops.rs`:
     - connection acquisition paths,
     - read/write command execution,
     - key composition and socket timeout application.
3. Added explicit failure-path test hooks (scoped visibility):
   - `valkey_get_with_for_tests`,
   - `valkey_set_with_for_tests`,
   - `normalize_valkey_timeout_ms_for_tests`.
   - surfaced through `native/mod.rs`, `deepseek/test_api.rs`, and
     `test_support.rs`.
4. Added crate-level regression tests:
   - `tests/llm_vision_deepseek_valkey_failures_unit.rs`:
     - invalid URL does not panic (`None`/`false`),
     - connection-refused endpoint does not panic (`None`/`false`),
     - timeout normalization clamps `0 -> 1`.
5. No lint suppression attributes introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit`
  - result: `20 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek valkey cache path now has clear init/ops boundaries.
- Failure-path behavior is explicitly asserted at top-level test boundary.
- Strict clippy + targeted nextest gates remain green.
