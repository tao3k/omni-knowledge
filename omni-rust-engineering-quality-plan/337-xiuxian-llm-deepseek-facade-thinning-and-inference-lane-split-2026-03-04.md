# 337. xiuxian-llm deepseek facade thinning and inference lane split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/mod.rs`
  - `src/llm/vision/deepseek/test_api.rs` (new)
  - `src/llm/vision/deepseek/inference.rs` (retired)
  - `src/llm/vision/deepseek/inference/mod.rs` (new)
  - `src/llm/vision/deepseek/inference/runtime_lane.rs` (new)
- Goal:
  - reduce deepseek entry/facade module density and isolate feature-gated
    runtime lanes from public inference API flow.

## Implementation

1. Extracted test-facing bridge API from facade:
   - moved deepseek test helpers out of `deepseek/mod.rs` into
     `deepseek/test_api.rs`.
   - `deepseek/mod.rs` now primarily declares submodules and re-exports.
2. Converted inference into directory module:
   - replaced `deepseek/inference.rs` with:
     - `deepseek/inference/mod.rs` for public inference/prewarm facade,
     - `deepseek/inference/runtime_lane.rs` for feature-gated enabled/disabled
       execution paths.
3. Preserved external behavior:
   - `infer_deepseek_ocr_truth` and `prewarm_deepseek_ocr` signatures unchanged.
   - feature-off lane still returns inert `Ok(None)` / `Ok(())` behavior.
4. Warning cleanup:
   - removed stale `DeepseekConfigSnapshot` re-export from `deepseek/mod.rs`
     after helper extraction, eliminating unused-import warning.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass (warning-free for touched deepseek facade paths)
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit`
  - result: `17 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek facade modules are thinner and more role-specific.
- Feature-gated runtime lane logic is isolated from API facade code.
- No suppression attributes were added; quality gates remain green.
