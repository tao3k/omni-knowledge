# 306. xiuxian-qianji compiler graph assembly module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/graph_assembly.rs` (new)
- Goal: extract manifest graph node/edge assembly from compiler core into a
  dedicated module while preserving compile-time graph validation behavior.

## Implementation

1. Added graph assembly module:
   - `src/engine/compiler/graph_assembly.rs`
   - introduced:
     - `add_manifest_nodes(engine, node_defs, build_mechanism, resolve_affinity)`
     - `add_manifest_edges(engine, id_to_index, edge_defs)`
     - internal `node_index_by_id(...)` helper
2. Updated compiler wiring:
   - `compiler.rs` now declares `mod graph_assembly;`
   - removed compiler-local `add_manifest_nodes`, `add_manifest_edges`,
     and `node_index_by_id` methods.
   - `compile()` now calls:
     - `graph_assembly::add_manifest_nodes(...)`
     - `graph_assembly::add_manifest_edges(...)`
3. Preserved behavior:
   - same mechanism construction path (`self.build_mechanism`)
   - same execution-affinity resolution (`annotation::node_execution_affinity`)
   - same edge resolution error contract (`{role} node not found: {node_id}`)

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/graph_assembly.rs`
  - result:
    - `compiler.rs`: `373`
    - `compiler/graph_assembly.rs`: `54`
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

- Compiler orchestration layer is further reduced and easier to reason about.
- Graph assembly responsibilities are now isolated in a domain-named module,
  improving maintainability without lint suppression or behavioral drift.
