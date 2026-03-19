# 381. xiuxian-qianji context-uri layout convergence on latest branch (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/layout/engine.rs`
  - `src/layout/bpmn.rs`
  - `src/bin/qianji.rs`
- Goal:
  - converge the latest `context_uri` branch version of layout/BPMN code to
    strict clippy policy without using suppression attributes.

## Implementation

1. `layout/engine.rs` (latest `context_uri` model retained):
   - preserved protocol metadata field:
     - `NodePosition.context_uri: Option<String>`
   - removed panic paths:
     - replaced edge endpoint `expect(...)` lookup with safe `Option` guards
       (`continue` on malformed references),
   - restored branch semantics for tests:
     - edge labels and weights now propagate from runtime graph edges,
     - probability suffixing (for `< 0.99`) is handled in layout edge labels,
   - removed lossy casts:
     - replaced `as f32` conversions with bounded helper conversion,
   - improved node classification:
     - start/end/gateway/business-rule/service-task mapping from node id and
       degree profile,
   - converted `compute_obsidian_graph` to associated function to eliminate
     `unused_self`,
   - added full public docs and `#[must_use]` on public constructors/returns.

2. `layout/bpmn.rs`:
   - replaced repeated `push_str(&format!(...))` with `write_fmt` helper path,
   - removed unreachable wildcard in BPMN tag mapping (exhaustive enum match),
   - added `#[must_use]` and API docs to export function,
   - preserved `qianji:context_uri` metadata emission from node payloads,
   - reintroduced sequence-flow labels and DI label rendering:
     - keeps `80%`/`20%` branch evidence and `<bpmndi:BPMNLabel>` contract
       expected by `test_layout_bpmn`.

3. `src/bin/qianji.rs`:
   - aligned graph export call site to associated function:
     - `QianjiLayoutEngine::compute_obsidian_graph(&engine)`.

4. No `allow(...)` attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/layout/engine.rs packages/rust/crates/xiuxian-qianji/src/layout/bpmn.rs packages/rust/crates/xiuxian-qianji/src/bin/qianji.rs`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass (no warnings/errors for `xiuxian-qianji`)
- Regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_layout_bpmn --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `20 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- The latest `context_uri`-oriented layout implementation is now clippy-clean
  under touched-crate strict gate.
- The BPMN export keeps protocol metadata and waypoint routing while satisfying
  existing integration test contracts.
- Core and LLM dispatch/test lanes remain green after convergence.
