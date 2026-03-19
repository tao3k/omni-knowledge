# 339. xiuxian-llm deepseek test-api domain split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/test_api.rs` (retired)
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/test_api/config_runtime.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/native_device.rs`
- Goal:
  - split the deepseek test bridge into domain-focused modules while preserving
    all existing test helper signatures and call paths.

## Implementation

1. Converted `deepseek/test_api.rs` to directory module:
   - `test_api/mod.rs` as interface-only export layer.
   - `test_api/config_runtime.rs` for config + model-root test hooks.
   - `test_api/native_cache.rs` for cache-key + valkey hooks.
   - `test_api/native_device.rs` for device label + fallback retry hooks.
2. Kept externally consumed helper names unchanged:
   - `load_config_with_paths_for_tests`
   - `resolve_model_root_with_for_tests`
   - `normalize_model_root_for_tests`
   - `build_cache_key_with_for_tests`
   - `resolve_device_kind_label_with_for_tests`
   - `should_retry_cpu_fallback_with_for_tests`
   - `valkey_get_with_for_tests`
   - `valkey_set_with_for_tests`
   - `normalize_valkey_timeout_ms_for_tests`
3. No behavior changes and no lint suppressions added.

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

- Deepseek test bridge now mirrors production-domain boundaries.
- Test helper surface is unchanged for callers, but maintenance locality is
  improved.
- Strict clippy + targeted nextest gates remain green.
