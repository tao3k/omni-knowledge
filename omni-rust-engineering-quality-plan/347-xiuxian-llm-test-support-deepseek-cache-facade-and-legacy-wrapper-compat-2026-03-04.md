# 347. xiuxian-llm test-support deepseek cache facade and legacy-wrapper compatibility (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/test_support.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/native/test_cache.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
- Goal:
  - consolidate deepseek cache-related test helper implementations into a
    single facade inside `test_support` while preserving all existing legacy
    helper function signatures for compatibility.

## Implementation

1. Added test-support cache facade:
   - introduced `DeepseekCacheTestFacade` in `src/test_support.rs`.
   - facade now owns deepseek cache helper implementations for:
     - cache key build
     - valkey get/set + timeout normalization
     - local cache get/set/clear
     - cache store pipeline trigger
     - cache layer label mapping
     - cache text normalization (`view`/`owned`)
2. Converted legacy helper functions to compatibility wrappers:
   - existing free functions in `src/test_support.rs` now delegate to
     `DeepseekCacheTestFacade` methods.
   - external call sites and test signatures remain unchanged.
3. Continued native-side facade routing:
   - `native/mod.rs` remains lean and re-exports `DeepseekNativeCacheTestFacade`.
   - `test_api/native_cache.rs` remains facade-driven (no direct raw forwarding
     calls).
4. No behavior changes and no broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit --test llm_vision_deepseek_cache_text_normalization_unit --test llm_vision_deepseek_cache_write_contract_unit --test llm_vision_deepseek_cache_layer_unit`
  - result: `28 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek cache test helper logic is now centrally maintained in one façade.
- Legacy free-function entrypoints remain available, so no downstream tests or
  callers are broken.
- Strict clippy and targeted regression gates remain green.
