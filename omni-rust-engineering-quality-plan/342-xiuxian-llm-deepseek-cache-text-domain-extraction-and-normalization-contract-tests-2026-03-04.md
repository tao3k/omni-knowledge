# 342. xiuxian-llm deepseek cache-text domain extraction and normalization contract tests (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/cache/mod.rs`
  - `src/llm/vision/deepseek/native/cache/text.rs` (new)
  - `src/llm/vision/deepseek/native/engine/cache_io.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_cache_text_normalization_unit.rs` (new)
- Goal:
  - extract cache-text normalization into a dedicated cache-domain module and
    lock its behavior via crate-top contract tests, while preserving existing
    runtime semantics.

## Implementation

1. Extracted cache-text normalization into a domain module:
   - added `native/cache/text.rs`.
   - moved and centralized:
     - `trim_non_empty(&str) -> Option<&str>`
     - `normalize_owned_non_empty(String) -> Option<String>`
2. Kept cache I/O orchestration focused:
   - `native/engine/cache_io.rs` now consumes normalization from
     `native/cache/text.rs` instead of defining inline helpers.
   - preserved valkey->local backfill behavior and cache-hit/empty telemetry.
3. Added explicit test hooks for cache-text normalization:
   - exported through:
     - `native/cache/mod.rs`
     - `native/mod.rs`
     - `deepseek/test_api/native_cache.rs`
     - `deepseek/test_api/mod.rs`
     - `deepseek/mod.rs`
     - `src/test_support.rs`
4. Added crate-top contract tests:
   - `tests/llm_vision_deepseek_cache_text_normalization_unit.rs`.
   - validates:
     - view (`&str`) normalization trim + empty filtering.
     - owned (`String`) normalization trim + empty filtering.
     - pointer preservation for already-trimmed owned input (proves
       no-reallocation fast path remains intact).
5. No broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit --test llm_vision_deepseek_cache_text_normalization_unit`
  - result: `25 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek cache-text normalization is now a first-class cache-domain concern,
  no longer hidden in engine orchestration code.
- Behavior is protected by crate-top tests, including the no-reallocation owned
  fast path.
- Runtime behavior and regression lanes remain green.
