# 340. xiuxian-llm deepseek local-cache policy/storage split and top-level policy tests (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/cache/local.rs` (retired)
  - `src/llm/vision/deepseek/native/cache/local/mod.rs`
  - `src/llm/vision/deepseek/native/cache/local/policy.rs`
  - `src/llm/vision/deepseek/native/cache/local/storage.rs`
  - `src/llm/vision/deepseek/native/cache/mod.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_local_cache_policy_unit.rs`
- Goal:
  - complete the interrupted local-cache modularization by splitting policy and
    storage concerns, then expose deterministic test hooks and lock policy
    behavior with crate-top tests.

## Implementation

1. Completed local-cache directory modularization:
   - retired `native/cache/local.rs`.
   - kept `native/cache/local/mod.rs` as orchestration entry.
   - added `native/cache/local/policy.rs` for env/default/capacity policy.
   - added `native/cache/local/storage.rs` for `OnceLock<RwLock<HashMap<...>>>`
     access and mutation primitives.
2. Kept module boundaries explicit:
   - `cache/mod.rs` remains interface-focused and re-exports only needed
     operations.
   - test-only hooks are exported with scoped visibility (no public API bloat).
3. Wired local-cache test hooks through the existing test bridge:
   - `native/mod.rs` -> `deepseek/test_api/native_cache.rs` ->
     `deepseek/test_api/mod.rs` -> `deepseek/mod.rs` -> `src/test_support.rs`.
4. Added crate-level policy tests:
   - `tests/llm_vision_deepseek_local_cache_policy_unit.rs`.
   - validates `len >= max_entries` clear-before-insert behavior.
   - validates `max_entries = 0` normalization to minimum capacity `1`.
   - uses a static `Mutex` guard to serialize local-cache mutation in the test
     binary.
5. No broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit`
  - result: `22 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek local cache now follows the same high-cohesion module pattern used
  in other recent deepseek splits.
- Policy and storage responsibilities are physically separated, reducing change
  risk for future cache tuning.
- The local-cache behavior contract is now protected at crate-top test level
  with deterministic execution.
