# 378. xiuxian-qianji layout unwrap removal and llm lane revalidation followup (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - updated:
    - `src/layout/engine.rs`
- Goal:
  - remove `clippy::unwrap_used` blockers in layout edge-routing path and
    revalidate previously blocked `--features llm` scheduler dispatch lane.

## Implementation

1. Replaced panic-prone edge endpoint lookups in layout engine:
   - removed:
     - `nodes.iter().find(...).unwrap()` for source and target nodes.
   - added:
     - explicit `let Some(...) = ... else { continue; };` fallback to skip
       malformed edge references without panicking.
2. Performed local style cleanup in the same block:
   - consolidated waypoint push sequence to `vec![...]`,
   - modernized a couple of `format!` call sites to inline argument style.
3. No lint-suppression attributes introduced.

## Verification

- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass (warnings remain in `layout/*`, no hard errors)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `xiuxian-qianji` clippy gate is unblocked from `unwrap_used` hard errors in
  layout engine.
- Prior `--features llm` dispatch revalidation lane now runs green in current
  workspace state.
