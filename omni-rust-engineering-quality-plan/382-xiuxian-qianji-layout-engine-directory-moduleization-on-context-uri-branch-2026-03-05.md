# 382. xiuxian-qianji layout engine directory moduleization on context-uri branch (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/layout/engine.rs` (replaced)
  - `src/layout/engine/mod.rs`
  - `src/layout/engine/types.rs`
  - `src/layout/engine/layout_core.rs`
  - `src/layout/engine/deep_graph.rs`
- Goal:
  - apply complexity-driven modularization on the latest `context_uri` layout
    implementation while preserving behavior and strict lint quality.

## Implementation

1. Replaced single-file layout engine with directory module:
   - removed monolithic `src/layout/engine.rs`,
   - introduced interface-only `src/layout/engine/mod.rs` for declarations and
     re-exports.

2. Concern split:
   - `types.rs`:
     - BPMN/deep-graph public data contracts
       (`BpmnType`, `NodePosition`, `EdgeLayout`, `LayoutResult`,
       `ZoneLayout`, `EntityKind`, `DeepNode`, `DeepEdge`,
       `DeepKnowledgeGraph`),
   - `layout_core.rs`:
     - `QianjiLayoutEngine` model + `compute_from_engine` + route/label/context
       helpers (`context_uri` retained),
   - `deep_graph.rs`:
     - `compute_obsidian_graph` associated-function implementation and deep
       graph construction path.

3. API stability:
   - preserved caller-visible import path:
     - `crate::layout::engine::{...}` remains valid via module re-exports,
   - preserved graph-export call site in `qianji.rs`:
     - `QianjiLayoutEngine::compute_obsidian_graph(&engine)`.

4. No lint-suppression attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/layout/engine/mod.rs packages/rust/crates/xiuxian-qianji/src/layout/engine/types.rs packages/rust/crates/xiuxian-qianji/src/layout/engine/layout_core.rs packages/rust/crates/xiuxian-qianji/src/layout/engine/deep_graph.rs packages/rust/crates/xiuxian-qianji/src/layout/bpmn.rs packages/rust/crates/xiuxian-qianji/src/bin/qianji.rs`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Extended lint surface check:
  - `cargo clippy -p xiuxian-qianji --all-targets --features llm -- -W clippy::too_many_lines`
  - result: pass
- Regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_layout_bpmn --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `20 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Layout engine now follows focused module boundaries instead of a monolith,
  aligned with repository modularization rules.
- `context_uri` behavior, BPMN output contracts, and deep-graph export remain
  intact after refactor.
- Strict lint/test gates remain green for `xiuxian-qianji`.
