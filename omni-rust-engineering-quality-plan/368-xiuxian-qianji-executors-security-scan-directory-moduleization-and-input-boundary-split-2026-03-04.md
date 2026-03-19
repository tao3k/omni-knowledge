# 368. xiuxian-qianji executors security-scan directory moduleization and input-boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/security_scan.rs`
  - added:
    - `src/executors/security_scan/mod.rs`
    - `src/executors/security_scan/mechanism.rs`
    - `src/executors/security_scan/input.rs`
- Goal:
  - replace mixed `security_scan` executor implementation with concern-split
    modules while preserving public `SecurityScanMechanism` API and scan
    behavior.

## Implementation

1. Converted `executors::security_scan` to directory module:
   - `mod.rs` is interface-only and re-exports `SecurityScanMechanism`.
2. Split responsibilities by concern:
   - `mechanism.rs`:
     - scanner execution loop, violation aggregation, abort/continue flow.
   - `input.rs`:
     - context files-key parsing (array/string), base-dir resolution,
       and effective scan-path derivation.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::executors::security_scan::SecurityScanMechanism`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/security_scan/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/security_scan/mechanism.rs packages/rust/crates/xiuxian-qianji/src/executors/security_scan/input.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `security_scan` now follows interface-only module entry with explicit boundary
  between context input parsing and scanner execution flow.
- Compiler dispatch and runtime behavior remain stable under core lanes.
