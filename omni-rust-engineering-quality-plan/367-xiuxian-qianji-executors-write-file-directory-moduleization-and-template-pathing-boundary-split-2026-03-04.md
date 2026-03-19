# 367. xiuxian-qianji executors write-file directory moduleization and template/pathing boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/write_file.rs`
  - added:
    - `src/executors/write_file/mod.rs`
    - `src/executors/write_file/mechanism.rs`
    - `src/executors/write_file/template.rs`
    - `src/executors/write_file/pathing.rs`
  - updated:
    - `src/executors/mod.rs` (`#[path = "write_file.rs"]` removed to activate directory module)
- Goal:
  - replace mixed `write_file` executor implementation with concern-split
    modules while preserving public `WriteFileMechanism` API and path-safety
    behavior.

## Implementation

1. Converted `executors::write_file` to directory module:
   - `mod.rs` is interface-only and re-exports `WriteFileMechanism`.
2. Split responsibilities by concern:
   - `mechanism.rs`:
     - mechanism execution flow (template resolution, destination validation,
       file write, telemetry output).
   - `template.rs`:
     - semantic template resolution and `{{key}}` interpolation behavior.
   - `pathing.rs`:
     - root-dir discovery and destination canonicalization/path-escape guard.
3. Module-path cleanup:
   - removed obsolete `#[path = "write_file.rs"]` in `executors/mod.rs` so
     Rust resolves `write_file/mod.rs`.
4. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::executors::write_file::WriteFileMechanism`.
5. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/write_file/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/write_file/mechanism.rs packages/rust/crates/xiuxian-qianji/src/executors/write_file/template.rs packages/rust/crates/xiuxian-qianji/src/executors/write_file/pathing.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Write-file focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_write_file_mechanism --test test_scheduler_preflight`
  - result: `9 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `write_file` now follows interface-only module entry and explicit boundaries
  for template rendering and pathing/security logic.
- Path creation and root-escape protection behavior remains stable under
  dedicated and core dispatch regression lanes.
