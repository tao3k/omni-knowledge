# 364. xiuxian-qianji executors wendao-ingester directory moduleization and entity/persistence boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/wendao_ingester.rs`
  - added:
    - `src/executors/wendao_ingester/mod.rs`
    - `src/executors/wendao_ingester/mechanism.rs`
    - `src/executors/wendao_ingester/entity.rs`
    - `src/executors/wendao_ingester/scope.rs`
    - `src/executors/wendao_ingester/persistence.rs`
- Goal:
  - replace mixed `wendao_ingester` implementation with concern-split modules
    while preserving public mechanism API and memory-promotion behavior.

## Implementation

1. Converted `executors::wendao_ingester` to directory module:
   - `mod.rs` is interface-only and re-exports `WendaoIngesterMechanism`.
2. Split responsibilities by concern:
   - `mechanism.rs`:
     - execution pipeline (`decision` resolution, entity/relation assembly,
       persistence orchestration, output shaping).
   - `entity.rs`:
     - promotion entity/topic/relation builders and key normalization/fallback id.
   - `scope.rs`:
     - graph scope resolution (`dynamic scope key` > `static scope` > default).
   - `persistence.rs`:
     - Valkey load/add/save graph persistence path with typed error mapping.
3. API compatibility preserved:
   - external import path remains:
     `xiuxian_qianji::executors::wendao_ingester::WendaoIngesterMechanism`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/wendao_ingester/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_ingester/mechanism.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_ingester/entity.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_ingester/scope.rs packages/rust/crates/xiuxian-qianji/src/executors/wendao_ingester/persistence.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Wendao-ingester focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_wendao_ingester_mechanism --test test_memory_promotion_pipeline`
  - result: `4 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `wendao_ingester` now follows interface-only module entry and explicit
  boundaries between execution orchestration, entity modeling, graph scope
  resolution, and persistence side effects.
- Memory-promotion behavior remains stable across targeted and core lanes.
