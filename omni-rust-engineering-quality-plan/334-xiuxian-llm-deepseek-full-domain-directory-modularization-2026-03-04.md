# 334. xiuxian-llm deepseek full-domain directory modularization (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/config/*`
  - `src/llm/vision/deepseek/runtime/*`
  - `src/llm/vision/deepseek/native/cache/*`
  - `src/llm/vision/deepseek/native/engine/*`
- Goal:
  - complete deepseek domain modularization by converting remaining
    single-file mixed-concern modules into directory modules with clear
    boundaries, while preserving runtime behavior and existing test contracts.

## Implementation

1. Converted `deepseek/config.rs` to directory module:
   - `config/mod.rs` as interface/re-export layer.
   - `config/raw.rs` for macro-backed TOML schema types.
   - `config/loader.rs` for default load/fallback policy.
   - `config/access.rs` for runtime accessor surface.
   - `config/snapshot.rs` for test snapshot mapping and explicit-path loader.
2. Converted `deepseek/runtime.rs` to directory module:
   - `runtime/mod.rs` for runtime enum + process-wide cache.
   - `runtime/model_root.rs` for root resolution, normalization, and defaults.
3. Converted `native/cache.rs` to directory module:
   - `native/cache/mod.rs` as interface/re-export layer.
   - `native/cache/key.rs` for cache-key hashing.
   - `native/cache/local.rs` for in-process cache storage.
   - `native/cache/valkey.rs` for distributed cache IO.
   - kept unit tests in `native/cache/tests.rs`.
4. Further split `native/engine/mod.rs` into focused modules:
   - `native/engine/mod.rs` as interface-only module.
   - `native/engine/lifecycle.rs` for runtime entry/fallback orchestration.
   - `native/engine/loader.rs` for model bootstrap/device selection wiring.
   - `native/engine/core.rs` for inference execution/caching pipeline.
   - existing focused modules retained:
     - `cache_io.rs`, `image_decode.rs`, `retry.rs`, `telemetry.rs`.
5. Visibility and import hardening:
   - applied scoped visibility (`pub(in crate::llm::vision::deepseek...)`)
     for cross-sibling use without broad `pub`.
   - corrected nested-module relative import paths after directory conversion.
   - no lint suppression attributes introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit`
  - result: `15 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek now follows end-to-end directory-module layout across config,
  runtime, native cache, and native engine boundaries.
- The largest deepseek single files were decomposed into domain-scoped modules,
  reducing mixed concerns and improving maintenance locality.
- Quality gates remain green with no suppression-first shortcuts.
