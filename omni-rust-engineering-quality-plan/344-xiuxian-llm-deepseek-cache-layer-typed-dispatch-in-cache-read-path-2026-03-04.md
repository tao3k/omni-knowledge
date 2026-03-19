# 344. xiuxian-llm deepseek cache-layer typed dispatch in cache read path (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/engine/cache_io/read.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io/mod.rs`
  - `src/llm/vision/deepseek/native/engine/core.rs`
- Goal:
  - replace string-based cache-layer dispatch (`"local"` / `"valkey"`) with a
    typed enum to reduce misuse surface and tighten compile-time contracts.

## Implementation

1. Introduced typed cache-layer enum in cache read domain:
   - added `CacheLayer` in `cache_io/read.rs` with variants:
     - `CacheLayer::Local`
     - `CacheLayer::Valkey`
   - added `as_str()` helper for telemetry field output.
2. Migrated cache read API to typed dispatch:
   - `read_cache_entry` now accepts `CacheLayer` instead of `&'static str`.
   - removed wildcard branch because enum variants are exhaustive.
3. Updated deepseek engine call sites:
   - `core.rs` now calls:
     - `read_cache_entry(CacheLayer::Local, ...)`
     - `read_cache_entry(CacheLayer::Valkey, ...)`
4. Preserved log semantics while improving type safety:
   - telemetry still emits `cache_layer = "local" | "valkey"` via enum mapping.
5. No broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit --test llm_vision_deepseek_cache_text_normalization_unit --test llm_vision_deepseek_cache_write_contract_unit`
  - result: `27 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Cache read path is now strongly typed and compile-time constrained.
- Runtime behavior is unchanged; existing telemetry and fallback behavior remain
  intact.
- Strict clippy and targeted regression gates remain green.
