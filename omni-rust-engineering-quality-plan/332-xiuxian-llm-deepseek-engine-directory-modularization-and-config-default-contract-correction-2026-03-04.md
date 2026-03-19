# 332. xiuxian-llm deepseek engine directory modularization and config default contract correction (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/engine/mod.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io.rs`
  - `src/llm/vision/deepseek/native/engine/image_decode.rs`
  - `src/llm/vision/deepseek/native/engine/retry.rs`
  - `src/llm/vision/deepseek/native/engine/telemetry.rs`
  - `tests/llm_vision_deepseek_config_unit.rs`
  - `resources/config/vision_deepseek.toml`
- Goal:
  - complete directory-module split for deepseek native engine with
    interface-only `mod.rs`,
  - keep behavior stable under strict clippy without suppression,
  - align config assertions with current embedded defaults.

## Implementation

1. Converted deepseek native engine into a directory module:
   - moved monolithic `engine.rs` into `engine/mod.rs`.
   - introduced focused submodules:
     - `cache_io.rs` for local/valkey cache read/write logic,
     - `image_decode.rs` for image payload decoding path,
     - `retry.rs` for safe-vision and CPU fallback retry policy,
     - `telemetry.rs` for inference telemetry shape and completion log output.
2. Fixed module-path imports introduced by split:
   - corrected `PreparedVisionImage` imports in engine child modules to
     `super::super::super::super::preprocess::PreparedVisionImage`.
3. Kept inference orchestration in `engine/mod.rs`:
   - retained engine init/prewarm behavior and fallback semantics.
   - delegated cache/decode/retry/telemetry concerns to submodules.
4. Corrected deepseek config test contract against embedded defaults:
   - updated `tests/llm_vision_deepseek_config_unit.rs`:
     - `image_size: Some(768) -> Some(640)`,
     - `max_new_tokens: Some(512) -> Some(256)`,
   - matching `resources/config/vision_deepseek.toml`:
     - `base_size = 1024`
     - `image_size = 640`
     - `max_new_tokens = 256`.

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

- Deepseek native engine now follows domain-split directory-module structure
  with clearer boundaries and lower maintenance risk.
- No lint suppression was added; strict clippy gate remains clean for
  `xiuxian-llm`.
- Deepseek config tests now match the canonical embedded defaults and are
  stable under targeted nextest execution.
