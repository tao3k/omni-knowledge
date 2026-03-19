# 365. xiuxian-qianji executors wendao-refresh directory moduleization and input/refresh boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/wendao_refresh.rs`
  - added:
    - `src/executors/wendao_refresh/mod.rs`
    - `src/executors/wendao_refresh/mechanism.rs`
    - `src/executors/wendao_refresh/input.rs`
    - `src/executors/wendao_refresh/refresh.rs`
- Goal:
  - replace mixed `wendao_refresh` executor with concern-split modules while
    preserving public mechanism API and incremental/full fallback behavior.

## Implementation

1. Converted `executors::wendao_refresh` to directory module:
   - `mod.rs` is interface-only and re-exports `WendaoRefreshMechanism`.
2. Split responsibilities by concern:
   - `mechanism.rs`:
     - mechanism data model and execution flow orchestration.
   - `input.rs`:
     - changed-path collection from context and root-dir resolution policy.
   - `refresh.rs`:
     - LinkGraph index bootstrap and incremental/full refresh execution
       decisions, including fallback handling and mode labeling.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::executors::wendao_refresh::WendaoRefreshMechanism`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh/mechanism.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh/input.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh/refresh.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Wendao-refresh focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_wendao_refresh_mechanism --test test_scheduler_preflight`
  - result: `7 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `wendao_refresh` now follows interface-only module entry and explicit
  boundaries between context input parsing and refresh execution strategy.
- Incremental/full fallback semantics remain stable across targeted and core
  execution lanes.
