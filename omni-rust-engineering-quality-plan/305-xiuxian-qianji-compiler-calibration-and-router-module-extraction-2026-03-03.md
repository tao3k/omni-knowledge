# 305. xiuxian-qianji compiler calibration and router module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/calibration.rs` (new)
  - `src/engine/compiler/router.rs` (new)
- Goal: extract calibration target parsing and router branch parsing/weight
  validation from compiler core into dedicated domain modules.

## Implementation

1. Added calibration module:
   - `src/engine/compiler/calibration.rs`
   - introduced:
     - `target_node_id(node_def)`
2. Added router module:
   - `src/engine/compiler/router.rs`
   - introduced:
     - `branches(node_def)`
     - `branch_weight(weight)` finite-float validation helper
3. Updated compiler wiring:
   - `compiler.rs` now declares `mod calibration;` and `mod router;`
   - `build_calibration_mechanism` now calls `calibration::target_node_id(node_def)`.
   - `build_router_mechanism` now calls `router::branches(node_def)`.
   - removed compiler-local router parse helpers and weight validator.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/calibration.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/router.rs`
  - result:
    - `compiler.rs`: `416`
    - `compiler/calibration.rs`: `10`
    - `compiler/router.rs`: `35`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler orchestration core is further reduced while preserving parse/validation behavior.
- Calibration and router concerns are now explicitly isolated, improving reviewability
  and maintainability under strict no-suppression engineering policy.
