# 323. xiuxian-qianji const resolver pipelines for root and leaf dispatch (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs`
- Goal: replace inline resolver function arrays at call sites with fixed
  `const` resolver pipelines, keeping execution order explicit and reducing
  dispatch callsite noise.

## Implementation

1. Added root-level resolver pipeline constant:
   - in `mechanism_dispatch.rs`:
     - `const ROOT_RESOLVERS: [resolver_chain::ResolverFn; 3] = [...]`
   - `build(...)` now calls:
     - `resolver_chain::run(&ROOT_RESOLVERS, context)`
2. Added leaf-level resolver pipeline constant:
   - in `leaf_dispatch/mod.rs`:
     - `const LEAF_RESOLVERS: [resolver_chain::ResolverFn; 3] = [...]`
   - `build(...)` now calls:
     - `resolver_chain::run(&LEAF_RESOLVERS, context)`
3. Preserved behavior:
   - resolver ordering remains unchanged.
   - mismatch error semantics remain unchanged.
   - no API signature changes in public-facing paths.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/resolver_chain.rs`
  - result:
    - `mechanism_dispatch.rs`: `30`
    - `leaf_dispatch/mod.rs`: `24`
    - `resolver_chain.rs`: `22`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Resolver pipeline configuration is now centralized as constants per routing
  layer.
- Dispatch callsites are shorter and less error-prone when future resolvers are
  added or reordered.
