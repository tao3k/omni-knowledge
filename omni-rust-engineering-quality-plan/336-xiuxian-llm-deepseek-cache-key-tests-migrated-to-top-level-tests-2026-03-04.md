# 336. xiuxian-llm deepseek cache-key tests migrated to top-level tests (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/cache/tests.rs` (removed)
  - `tests/llm_vision_deepseek_cache_key_unit.rs` (new)
  - `src/test_support.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
- Goal:
  - align deepseek cache-key validation with top-level package test layout and
    keep equivalent behavior coverage without inline module test wiring.

## Implementation

1. Removed module-local deepseek cache tests:
   - deleted `src/llm/vision/deepseek/native/cache/tests.rs`.
   - removed `#[cfg(test)] mod tests;` from `native/cache/mod.rs`.
2. Added top-level cache-key integration tests:
   - `tests/llm_vision_deepseek_cache_key_unit.rs`
   - preserved original invariants:
     - cache key changes with decode budget (`max_new_tokens`),
     - cache key changes with prompt or vision dimensions.
3. Added explicit test-support bridge for cache-key construction:
   - `test_support::DeepseekCacheKeyInput`.
   - `test_support::build_deepseek_cache_key_for_tests(&DeepseekCacheKeyInput)`.
   - keeps integration tests outside internal module paths.
4. Added deepseek test-only forwarding hooks:
   - `deepseek::build_cache_key_with_for_tests(...)`.
   - `deepseek::native::build_cache_key_for_tests(...)`.
5. Warning-free API shaping:
   - replaced 9-argument helper signature with struct-based input to avoid
     `clippy::too_many_arguments` warnings without suppression.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass (no warning regressions from helper API)
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit`
  - result: `17 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek cache-key tests now follow package-top test placement.
- Internal module code no longer carries local test harness coupling for this
  case.
- Coverage and quality gates remain green.
