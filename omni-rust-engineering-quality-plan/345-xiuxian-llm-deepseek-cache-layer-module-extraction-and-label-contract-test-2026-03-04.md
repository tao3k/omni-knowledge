# 345. xiuxian-llm deepseek cache-layer module extraction and label contract test (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/engine/cache_io/mod.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io/layer.rs` (new)
  - `src/llm/vision/deepseek/native/engine/cache_io/read.rs`
  - `src/llm/vision/deepseek/native/engine/core.rs`
  - `src/llm/vision/deepseek/native/engine/mod.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_cache_layer_unit.rs` (new)
- Goal:
  - extract `CacheLayer` to its own module and add a crate-top contract test
    that locks enum->telemetry label mapping.

## Implementation

1. Extracted cache-layer domain type:
   - added `cache_io/layer.rs` for `CacheLayer` and `as_str()` mapping.
   - removed enum definition from `cache_io/read.rs`.
2. Kept cache read pipeline typed:
   - `cache_io/read.rs` now imports `CacheLayer` from `layer.rs`.
   - `core.rs` continues using variant-based dispatch
     (`CacheLayer::Local`/`CacheLayer::Valkey`).
3. Added label-mapping test hook path:
   - `cache_io/mod.rs`: `cache_layer_labels_for_tests()`.
   - exported through:
     - `native/engine/mod.rs`
     - `native/mod.rs`
     - `deepseek/test_api/native_cache.rs`
     - `deepseek/test_api/mod.rs`
     - `deepseek/mod.rs`
     - `src/test_support.rs`
4. Added crate-top contract test:
   - `tests/llm_vision_deepseek_cache_layer_unit.rs`.
   - asserts stable mapping:
     - `CacheLayer::Local -> "local"`
     - `CacheLayer::Valkey -> "valkey"`
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

- Cache-layer type and label mapping are now isolated in a dedicated module.
- Enum label contract is fixed by crate-top test coverage.
- Strong typing, modular boundaries, and validation gates remain intact.
