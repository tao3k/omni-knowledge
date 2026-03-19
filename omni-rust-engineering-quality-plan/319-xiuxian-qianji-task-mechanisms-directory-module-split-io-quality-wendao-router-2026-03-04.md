# 319. xiuxian-qianji `task_mechanisms` directory module split (`io`, `quality`, `wendao_router`) (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/task_mechanisms.rs` (removed)
  - `src/engine/compiler/task_mechanisms/mod.rs` (new)
  - `src/engine/compiler/task_mechanisms/io_control.rs` (new)
  - `src/engine/compiler/task_mechanisms/quality.rs` (new)
  - `src/engine/compiler/task_mechanisms/wendao_router.rs` (new)
- Goal: split leaf mechanism constructors by domain so IO-control logic, quality
  logic, and wendao/router logic are no longer mixed in one file.

## Implementation

1. Converted `task_mechanisms.rs` to directory module:
   - removed single-file implementation.
   - added `task_mechanisms/mod.rs` as stable interface layer.
2. Added focused domain modules:
   - `io_control.rs`:
     - `command`, `write_file`, `suspend`
   - `quality.rs`:
     - `calibration`, `mock`, `security_scan`
   - `wendao_router.rs`:
     - `wendao_ingester`, `wendao_refresh`, `router`
3. Preserved API contract at module boundary:
   - `task_mechanisms/mod.rs` keeps the same callable function names used by
     dispatch layers (`calibration`, `mock`, `command`, `write_file`,
     `suspend`, `security_scan`, `wendao_ingester`, `wendao_refresh`, `router`)
     via thin forwarding wrappers.
4. No behavior changes:
   - only internal module boundaries and routing ownership changed.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/io_control.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/quality.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/wendao_router.rs`
  - result:
    - `task_mechanisms/mod.rs`: `43`
    - `task_mechanisms/io_control.rs`: `33`
    - `task_mechanisms/quality.rs`: `29`
    - `task_mechanisms/wendao_router.rs`: `40`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Mechanism constructor ownership is now explicit by domain and easier to extend
  without growing one mixed file.
- `task_mechanisms/mod.rs` now acts as a stable interface boundary while
  implementation complexity stays in dedicated submodules.
