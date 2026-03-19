# 333. xiuxian-llm deepseek native env directory modularization (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/env.rs` (retired)
  - `src/llm/vision/deepseek/native/env/mod.rs`
  - `src/llm/vision/deepseek/native/env/device.rs`
  - `src/llm/vision/deepseek/native/env/parse.rs`
  - `src/llm/vision/deepseek/native/env/paths.rs`
- Goal:
  - decompose mixed-responsibility deepseek env module into focused domain
    modules while preserving external call sites and runtime behavior.

## Implementation

1. Converted `env.rs` to a directory module:
   - replaced single-file env implementation with `env/mod.rs` plus
     `device.rs`, `parse.rs`, and `paths.rs`.
2. Split concerns by domain:
   - `device.rs`:
     - deepseek runtime device resolution strategy,
     - explicit/fallback acceleration mapping,
     - platform-feature aware metal/cuda fallback behavior,
     - test-facing device-label resolver.
   - `parse.rs`:
     - typed environment parsing (`u32`, `usize`, `u64`, `bool`),
     - config fallback mapping for parse misses.
   - `paths.rs`:
     - weights path resolution and existence checks,
     - snapshot/prompt/cache env+config path/value resolution.
3. Preserved external API surface for sibling modules:
   - `native/cache.rs` and `native/engine/mod.rs` continue to import through
     `super::env::{...}`.
   - `native/mod.rs` continues to re-export
     `resolve_device_kind_label_for_tests`.
4. Applied visibility scoping for safe re-export:
   - used `pub(in crate::llm::vision::deepseek::native)` on child-module
     internals consumed across `native` siblings.
   - avoided broad `pub` exposure.
5. Fixed cfg-sensitive import hygiene:
   - `sanitize_error_string` is imported conditionally for the macOS + metal
     probe path to avoid unused import warnings on non-matching targets.

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

- Deepseek env logic now follows explicit domain boundaries (`device`, `parse`,
  `paths`) with lower maintenance coupling.
- No lint suppression was introduced.
- Touched crate quality gates and dependent dispatch regression lanes remain
  green.
