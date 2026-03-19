# 309. xiuxian-qianji compiler leaf task mechanism builders extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/task_mechanisms.rs` (new)
- Goal: move non-`self` dependent task-mechanism constructors out of compiler
  core so the compiler primarily orchestrates dispatch and stateful lanes.

## Implementation

1. Added leaf task mechanism builders module:
   - `src/engine/compiler/task_mechanisms.rs`
   - introduced builders for:
     - `calibration`
     - `mock`
     - `command`
     - `write_file`
     - `suspend`
     - `security_scan`
     - `wendao_ingester`
     - `wendao_refresh`
     - `router`
2. Updated compiler wiring:
   - `compiler.rs` now declares `mod task_mechanisms;`
   - `build_mechanism` routes leaf tasks to `task_mechanisms::*`.
3. Removed duplicated constructor blocks from `compiler.rs`:
   - deleted local `build_*` methods for those leaf task lanes.
4. Preserved behavior:
   - all config parsers (`io_mechanisms`, `security_scan`, `wendao_ingester`,
     `wendao_refresh`, `router`, `calibration`) remain unchanged and are reused.
   - typed task dispatch and error contracts remain unchanged.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms.rs`
  - result:
    - `compiler.rs`: `292`
    - `compiler/task_mechanisms.rs`: `94`
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

- Compiler core is now materially thinner and focused on orchestration.
- Leaf task construction is centralized in a domain module, improving extension
  and review ergonomics without lint suppression or behavior drift.
