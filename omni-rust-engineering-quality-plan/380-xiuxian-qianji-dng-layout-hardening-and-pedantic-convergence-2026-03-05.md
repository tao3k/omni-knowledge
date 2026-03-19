# 380. xiuxian-qianji DNG layout hardening and pedantic convergence (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/layout/engine.rs`
  - `src/layout/bpmn.rs`
  - `src/bin/qianji.rs`
  - `tests/test_layout_bpmn.rs`
- Goal:
  - keep the Sovereign DNG/zone visualization semantics while removing
    `expect` panics and converging pedantic warnings without any lint
    suppression.

## Implementation

1. `layout/engine.rs` hardening and modular decomposition:
   - preserved DNG features (`zones`, `compute_obsidian_graph`, deep graph
     entities),
   - replaced panic-prone edge endpoint lookup with safe `Option` branching
     (`continue` on malformed references),
   - added typed helpers for node classification, sizing, level resolution,
     waypoint routing, edge labeling, and zone construction,
   - replaced lossy `usize as f32` casts with bounded conversion helper,
   - added complete API docs for all public enums/structs/fields/functions,
   - moved `compute_obsidian_graph` to an associated function to remove
     `unused_self` warning.

2. `layout/bpmn.rs` XML pipeline cleanup:
   - replaced repeated `push_str(&format!(...))` with `write_fmt` helper to
     remove allocation-heavy `format_push_string` pattern,
   - split long serializer into focused helpers (`header`, process nodes/edges,
     zone artifacts, DI zone/node/edge layers, edge label rendering),
   - kept zone-first DI rendering order and edge label DI overlays,
   - added `#[must_use]` + API docs for export entrypoint.

3. Caller alignment:
   - updated graph export path in `src/bin/qianji.rs` to call
     `QianjiLayoutEngine::compute_obsidian_graph(&engine)` after the associated
     function change.

4. Test lane housekeeping:
   - `tests/test_layout_bpmn.rs` keeps crate-level docs and continues to verify
     gateway rendering, probability labels, waypoint output, and DI labels.

5. No `allow(...)` attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/layout/engine.rs packages/rust/crates/xiuxian-qianji/src/layout/bpmn.rs packages/rust/crates/xiuxian-qianji/src/bin/qianji.rs`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji --all-targets --features llm -- -W clippy::too_many_lines`
  - result: pass for `xiuxian-qianji` (no crate-local warnings/errors)
  - note: upstream dependency `xiuxian-llm` still emits existing `doc_markdown`
    warnings in `providers/openai.rs`, outside this task scope.
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_layout_bpmn --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `20 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- DNG layout and BPMN exporter remain feature-complete while removing panic
  paths and pedantic debt in the touched area.
- Layout/XML code now follows helper-oriented boundaries that are easier to
  evolve under strict lint policy.
- Core and LLM dispatch regression lanes remain green after the refactor.
