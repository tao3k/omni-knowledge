# 310. xiuxian-qianji compiler stateful lanes module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/stateful_mechanisms.rs` (new)
- Goal: extract stateful mechanism construction lanes (`annotation`,
  `formal_audit`, `llm`) out of compiler core while preserving feature-gated behavior.

## Implementation

1. Added stateful mechanisms module:
   - `src/engine/compiler/stateful_mechanisms.rs`
   - introduced:
     - `annotation(orchestrator, registry, node_def)`
     - `formal_audit_with_llm(orchestrator, registry, node_def, client)`
     - `formal_audit_native(node_def)`
     - `formal_audit_requires_llm_guard(node_def)`
     - `llm(node_def, client)` (`feature = "llm"`)
2. Updated compiler wiring:
   - `compiler.rs` now declares `mod stateful_mechanisms;`
   - `build_annotation_mechanism` now delegates to
     `stateful_mechanisms::annotation(...)`.
   - `dispatch_formal_audit` now delegates to stateful module paths:
     - with `llm` feature: chooses native/llm-augmented branch and reuses
       `resolve_llm_client_for_node`.
     - without `llm` feature: keeps strict guard behavior then falls back to
       native formal-audit mechanism.
   - `dispatch_llm` now delegates to `stateful_mechanisms::llm(...)` in
     `llm` feature builds and preserves previous error behavior otherwise.
3. Removed duplicated construction logic from compiler core:
   - deleted inlined `ContextAnnotator`, `LlmAnalyzer`,
     and `LlmAugmentedAuditMechanism` construction blocks.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/stateful_mechanisms.rs`
  - result:
    - `compiler.rs`: `206`
    - `compiler/stateful_mechanisms.rs`: `100`
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

- Compiler core is now focused on orchestration/dispatch, with stateful mechanism
  construction isolated in a dedicated domain module.
- Feature-gated behavior and runtime semantics remain unchanged.
