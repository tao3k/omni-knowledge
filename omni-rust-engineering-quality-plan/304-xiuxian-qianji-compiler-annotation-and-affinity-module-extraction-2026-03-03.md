# 304. xiuxian-qianji compiler annotation and affinity module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/annotation.rs` (new)
- Goal: extract annotation binding parsing and execution-affinity resolution out
  of compiler core to keep orchestration logic focused and easier to audit.

## Implementation

1. Added `annotation` domain module:
   - `src/engine/compiler/annotation.rs`
   - introduced:
     - `AnnotationMechanismConfig`
     - `mechanism_config(node_def)`
     - `node_execution_affinity(node_def)`
2. Moved parser responsibilities from `compiler.rs`:
   - `annotation_*` configuration parsing (persona/template/execution mode/input keys/history/output).
   - `node_execution_affinity` and role-class derivation from persona.
   - local helpers for semantic placeholder normalization and non-empty filtering.
3. Updated compiler wiring:
   - `compiler.rs` now declares `mod annotation;`
   - `build_annotation_mechanism` consumes `annotation::mechanism_config(node_def)`.
   - LLM formal-audit annotator path also consumes `annotation::mechanism_config(node_def)`.
   - `add_manifest_nodes` now uses `annotation::node_execution_affinity(&node_def)`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/annotation.rs`
  - result:
    - `compiler.rs`: `458`
    - `compiler/annotation.rs`: `149`
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

- Compiler orchestration density dropped further without behavior or default-contract drift.
- Annotation and affinity logic now has explicit domain boundaries, reducing
  future change risk and making review/debug easier under strict no-suppression policy.
