# 327. xiuxian-qianji quality-guard dispatch coverage expansion (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes.rs`
- Goal: complete coverage for `quality_guard` leaf dispatch lanes to harden
  compiler dispatch behavior under ongoing modular refactors.

## Implementation

1. Added `quality_guard` lane success tests:
   - `compiler_dispatches_calibration_task_via_leaf_lane`
   - `compiler_dispatches_mock_task_via_leaf_lane`
   - `compiler_dispatches_security_scan_task_via_leaf_lane`
2. Kept the route test suite centralized:
   - all additions stay in top-level integration test file
     `tests/test_compiler_dispatch_routes.rs`.
3. Preserved no-suppression policy:
   - no `allow(...)` attributes introduced; behavior verified by explicit tests
     and touched-crate clippy gate.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `15 passed`, `0 skipped`, `0 failed`

## Outcome

- Leaf dispatch coverage now includes the full `quality_guard` route group
  (`calibration`, `mock`, `security_scan`) in compiler integration tests.
- Route-level regressions in this dispatch branch are now caught early by
  targeted nextest runs.
