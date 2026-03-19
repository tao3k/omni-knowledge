# 349. xiuxian-llm test-support directory moduleization and interface-only mod (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/test_support/mod.rs`
  - `src/test_support/acceleration.rs`
  - `src/test_support/deepseek_config.rs`
  - `src/test_support/deepseek_runtime.rs`
  - `src/test_support/deepseek_cache_api.rs`
  - `src/test_support/deepseek_cache/*`
  - removed: `src/test_support.rs`
- Goal:
  - convert `test_support` from a single file module into a directory module
    with interface-only `mod.rs`, while preserving all existing
    `xiuxian_llm::test_support::*` public helper signatures.

## Implementation

1. Promoted `test_support` to directory module:
   - replaced `src/test_support.rs` with `src/test_support/mod.rs`.
   - `mod.rs` now only declares/re-exports submodules (no implementation
     logic).
2. Split non-cache helper domains:
   - `acceleration.rs`:
     - `parse_acceleration_device_for_tests`
     - `resolve_acceleration_device_with_for_tests`
     - `load_acceleration_device_with_paths`
   - `deepseek_config.rs`:
     - `DeepseekCacheConfigSnapshot`
     - `DeepseekConfigSnapshot`
     - deepseek config load/model-root helper APIs
   - `deepseek_runtime.rs`:
     - deepseek device-kind and cpu-fallback helper APIs
3. Isolated cache wrapper compatibility layer:
   - added `deepseek_cache_api.rs` to host legacy free-function wrappers.
   - wrappers continue delegating to `DeepseekCacheTestFacade`.
4. Kept existing deepseek cache sub-facade structure:
   - `deepseek_cache/mod.rs` remains interface-only.
   - implementation stays split by concern (`key`, `valkey`, `local`, `text`,
     `write`, `facade`).
5. Public API compatibility:
   - no external helper name/signature changed.
   - existing crate-top deepseek tests and parity contracts run unchanged.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit --test llm_vision_deepseek_cache_text_normalization_unit --test llm_vision_deepseek_cache_write_contract_unit --test llm_vision_deepseek_cache_layer_unit --test llm_vision_deepseek_cache_facade_compat_unit`
  - result: `33 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `test_support` now matches the repository modularization standard:
  interface-only `mod.rs` + focused domain modules.
- API compatibility is preserved and covered by existing deepseek contract tests.
- Strict clippy and targeted regression gates remain green.
