# 335. xiuxian-llm deepseek device policy/probe split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/env/device.rs` (retired)
  - `src/llm/vision/deepseek/native/env/device/mod.rs`
  - `src/llm/vision/deepseek/native/env/device/policy.rs`
  - `src/llm/vision/deepseek/native/env/device/probe.rs`
- Goal:
  - separate deepseek device selection policy from platform probing details,
    keeping public behavior and test hooks unchanged.

## Implementation

1. Converted `env/device.rs` to directory module:
   - `device/mod.rs` as interface and orchestration shell.
   - `device/policy.rs` for deterministic device-selection policy:
     - explicit/fallback mode mapping,
     - platform/feature-aware supported-device resolution,
     - auto/metal/cuda fallback behavior.
   - `device/probe.rs` for metal availability probing:
     - runtime probe path for `macos + vision-dots-metal`,
     - non-matching target fast path returning `false`.
2. Preserved existing API surface:
   - `parse_device_kind()` still exported for native runtime consumption.
   - `resolve_device_kind_label_for_tests()` remains available for test
     contracts.
3. Kept visibility tight and scoped:
   - no broad `pub` expansion outside intended deepseek native boundary.
4. No lint suppression introduced.

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

- Device policy logic and platform probe logic are now decoupled, reducing
  mixed concerns in the deepseek env layer.
- Deepseek runtime behavior and existing tests remain stable under strict
  clippy + nextest gates.
