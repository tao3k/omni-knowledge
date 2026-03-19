# 379. xiuxian-qianji layout pedantic root-cause cleanup and function decomposition (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/layout/engine.rs`
  - `src/layout/bpmn.rs`
- Goal:
  - remove remaining pedantic warnings in the layout path without any lint
    suppression, and keep dispatch/test lanes green.

## Implementation

1. `layout/bpmn.rs` warning cleanup (`format_push_string`, format-inline, pattern):
   - introduced a small `push_fmt` helper using `String::write_fmt` for
     allocation-free string assembly instead of `push_str(&format!(...))`,
   - inlined format arguments (`{label}`, `{wx}`, `{wy}`),
   - replaced redundant pattern matching with `edge.label.is_some()`,
   - added `#[must_use]` and API documentation to `generate_bpmn_xml`.

2. `layout/engine.rs` warning cleanup and structure hardening:
   - completed public API docs for `BpmnType`, `NodePosition`, `EdgeLayout`,
     `LayoutResult`, and `QianjiLayoutEngine::new`,
   - fixed `single_char_pattern` by using `replace('_', " ")`,
   - decomposed `compute_from_engine` into focused helpers:
     - `node_dimensions`
     - `parent_level`
     - `build_node`
     - `feedback_waypoints`
     - `forward_waypoints`
     - `edge_label`
     - `build_edge_layout`
   - this removed the `clippy::too_many_lines` warning by shrinking the
     orchestration function and isolating responsibilities.

3. No `allow(...)` attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/layout/engine.rs packages/rust/crates/xiuxian-qianji/src/layout/bpmn.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass (`0 warnings`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Layout-related clippy pedantic debt in `xiuxian-qianji` is now resolved
  through code improvements, not suppression.
- `compute_from_engine` is now split into single-purpose helpers with clearer
  maintainability and lower change risk.
- Core + LLM dispatch regression lanes remain green after the refactor.
