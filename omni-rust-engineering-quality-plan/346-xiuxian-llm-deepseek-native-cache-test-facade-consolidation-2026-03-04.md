# 346. xiuxian-llm deepseek native cache test-facade consolidation (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/native/test_cache.rs` (new)
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/test_support.rs`
- Goal:
  - replace scattered native cache test forwarders with a single native-level
    cache test façade and keep higher-level test APIs stable.

## Implementation

1. Added native cache façade:
   - new `native/test_cache.rs` with `DeepseekNativeCacheTestFacade`.
   - façade aggregates cache-test capabilities:
     - cache key build
     - valkey get/set + timeout normalize
     - local cache get/set/clear
     - cache text normalize (view/owned)
     - cache store entrypoint
     - cache layer label mapping
2. Simplified `native/mod.rs` to interface-only behavior for cache tests:
   - removed 11 standalone cache-test forwarding functions.
   - re-exported `DeepseekNativeCacheTestFacade` to `deepseek` scope.
3. Updated test bridge consumer:
   - `test_api/native_cache.rs` now routes all calls through
     `DeepseekNativeCacheTestFacade` instead of direct per-function native
     forwarders.
4. Preserved external test API contracts:
   - signatures exposed by `deepseek/test_api`, `deepseek/mod.rs`, and
     `src/test_support.rs` remain unchanged for callers/tests.
5. No broad lint suppression was added.

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

- Native cache test plumbing is now centralized in a single façade.
- `native/mod.rs` returned to a lean interface role with less coupling risk.
- Existing top-level test contracts remain stable and fully green.
