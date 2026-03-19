# 348. xiuxian-llm test-support deepseek cache sub-facades and wrapper parity contracts (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/test_support.rs`
  - `src/test_support/deepseek_cache/mod.rs`
  - `src/test_support/deepseek_cache/facade.rs`
  - `src/test_support/deepseek_cache/key.rs`
  - `src/test_support/deepseek_cache/valkey.rs`
  - `src/test_support/deepseek_cache/local.rs`
  - `src/test_support/deepseek_cache/text.rs`
  - `src/test_support/deepseek_cache/write.rs`
  - `tests/llm_vision_deepseek_cache_facade_compat_unit.rs`
- Goal:
  - reduce cache-helper method crowding in `DeepseekCacheTestFacade` by moving
    implementation into focused domain modules while preserving legacy wrapper
    APIs and locking compatibility with explicit parity tests.

## Implementation

1. Split deepseek cache helper internals into domain modules:
   - added `src/test_support/deepseek_cache/` with focused modules:
     - `key.rs` (cache-key input + image-prep key construction)
     - `valkey.rs` (valkey get/set + timeout normalization)
     - `local.rs` (local cache get/set/clear)
     - `text.rs` (view/owned normalization)
     - `write.rs` (cache write path + cache-layer labels)
     - `facade.rs` (single delegating facade entrypoint)
   - `mod.rs` is interface-only and re-exports:
     - `DeepseekCacheKeyInput`
     - `DeepseekCacheTestFacade`
2. Slimmed `src/test_support.rs`:
   - removed dense inline facade implementation block.
   - retained all existing public compatibility wrappers and routed them through
     the split `deepseek_cache` module.
   - kept external function signatures unchanged.
3. Added explicit wrapper/facade parity tests:
   - new crate-top test file
     `tests/llm_vision_deepseek_cache_facade_compat_unit.rs`.
   - verifies wrapper/facade equivalence for:
     - cache key building
     - text normalization (`&str` and `String`)
     - timeout normalization and cache-layer labels
     - local cache get/set/clear behavior
     - cache write path and valkey failure-path forwarding.
4. No broad lint suppression was introduced.

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

- Deepseek cache test-support logic is now organized by domain concern instead
  of a single crowded implementation block.
- Legacy wrapper APIs remain stable and are now contract-locked against the
  facade implementation.
- Strict clippy and targeted regression gates remain green.
