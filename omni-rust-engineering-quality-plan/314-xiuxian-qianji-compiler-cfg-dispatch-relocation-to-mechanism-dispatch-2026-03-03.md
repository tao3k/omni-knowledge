# 314. xiuxian-qianji compiler cfg dispatch relocation to mechanism dispatch (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/mechanism_dispatch.rs`
- Goal: relocate remaining cfg-heavy dispatch logic (`formal_audit`, `llm`) and
  mechanism builders out of compiler core, consolidating all task dispatch in
  `mechanism_dispatch` while preserving behavior.

## Implementation

1. Compiler shell contraction:
   - removed from `compiler.rs`:
     - `build_knowledge_mechanism(...)`
     - `build_annotation_mechanism(...)`
     - `dispatch_formal_audit(...)` (`llm` and non-`llm` variants)
     - `dispatch_llm(...)` (`llm` and non-`llm` variants)
   - `compiler.rs` now focuses on:
     - dependency injection (`new`)
     - compile pipeline orchestration (`compile`)
2. Dispatch domain consolidation:
   - expanded `compiler/mechanism_dispatch.rs`:
     - dispatch now builds `KnowledgeSeeker` directly.
     - annotation path now directly delegates to
       `stateful_mechanisms::annotation(...)`.
     - cfg-specific `formal_audit` and `llm` routing now lives in this module
       via local `dispatch_formal_audit(...)` and `dispatch_llm(...)` helpers.
3. Behavior compatibility:
   - preserved `formal_audit` LLM-controller branch behavior and error guard
     semantics for non-`llm` builds.
   - preserved node-level/global LLM client fallback behavior through
     `llm_client::resolve_for_node(...)`.
4. Compiler size reduction:
   - `compiler.rs` line count reduced from `158` to `95`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs`
  - result:
    - `compiler.rs`: `95`
    - `compiler/mechanism_dispatch.rs`: `82`
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

- Compiler core is now a thin, stable orchestration shell.
- cfg-specific mechanism dispatch complexity is isolated in a dedicated module,
  improving maintainability and reducing future expansion pressure on
  `compiler.rs`.
