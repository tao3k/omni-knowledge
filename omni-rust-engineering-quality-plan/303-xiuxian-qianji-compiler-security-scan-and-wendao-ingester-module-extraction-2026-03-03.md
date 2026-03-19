# 303. xiuxian-qianji compiler security scan and wendao ingester module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/security_scan.rs` (new)
  - `src/engine/compiler/wendao_ingester.rs` (new)
- Goal: move `security_scan` and `wendao_ingester` parameter decoding out of
  compiler core into domain-focused modules, keeping compiler logic orchestration-only.

## Implementation

1. Added `security_scan` config module:
   - `src/engine/compiler/security_scan.rs`
   - introduced:
     - `SecurityScanMechanismConfig`
     - `mechanism_config(node_def)`
   - preserved defaults:
     - `files_key`: `staged_files`
     - `output_key`: `security_issues`
     - `abort_on_violation`: `true`
2. Added `wendao_ingester` config module:
   - `src/engine/compiler/wendao_ingester.rs`
   - introduced:
     - `WendaoIngesterMechanismConfig`
     - `mechanism_config(node_def)`
   - preserved runtime-default resolution via
     `resolve_qianji_runtime_wendao_ingester_config()` with hard-default fallback.
3. Updated compiler wiring:
   - `compiler.rs` now declares `mod security_scan;` and `mod wendao_ingester;`
   - `build_security_scan_mechanism` and `build_wendao_ingester_mechanism`
     now consume extracted configs and only construct mechanism instances.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/security_scan.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/wendao_ingester.rs`
  - result:
    - `compiler.rs`: `584`
    - `compiler/security_scan.rs`: `35`
    - `compiler/wendao_ingester.rs`: `69`
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

- `compiler.rs` density is reduced again while behavior/default contracts remain stable.
- Mechanism config boundaries now align with task domains (`security_scan`,
  `wendao_ingester`), improving maintainability and review clarity without lint suppression.
