# 322. xiuxian-qianji mechanism dispatch context-based resolver signature unification (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch/resolver_chain.rs`
  - `src/engine/compiler/mechanism_dispatch.rs`
  - `src/engine/compiler/mechanism_dispatch/stateless.rs`
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs`
- Goal: remove repeated resolver parameter triplets
  (`task_type`, `compiler`, `node_def`) by introducing a single shared dispatch
  context type.

## Implementation

1. Introduced shared dispatch context:
   - in `resolver_chain.rs`, added:
     - `DispatchContext<'a> { task_type, compiler, node_def }`
   - resolver function type updated to:
     - `for<'a> fn(DispatchContext<'a>) -> Option<ResolveOutcome>`
2. Updated resolver-chain runtime:
   - `resolver_chain::run(...)` now accepts one `DispatchContext` value.
   - the context is `Copy`, so chain iteration stays lightweight.
3. Unified resolver signatures:
   - top-level resolvers:
     - `stateless::build(context)`
     - `stateful_cfg::build(context)`
     - `leaf_dispatch::build(context)`
   - leaf-level resolvers:
     - `io_control::build(context)`
     - `quality_guard::build(context)`
     - `wendao_router::build(context)`
4. Updated callers:
   - `mechanism_dispatch.rs` now builds one context and dispatches via
     `resolver_chain::run(...)`.
   - `leaf_dispatch/mod.rs` also uses `resolver_chain::run(...)` and preserves
     mismatch error semantics.
5. Cleanup:
   - removed imports made redundant by context-based signatures.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/resolver_chain.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs`
  - result:
    - `mechanism_dispatch.rs`: `31`
    - `resolver_chain.rs`: `22`
    - `stateless.rs`: `26`
    - `stateful_cfg/mod.rs`: `15`
    - `leaf_dispatch/mod.rs`: `26`
    - `leaf_dispatch/io_control.rs`: `18`
    - `leaf_dispatch/quality_guard.rs`: `18`
    - `leaf_dispatch/wendao_router.rs`: `18`
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

- Resolver interfaces are now uniform and easier to extend.
- Dispatch callsites no longer pass repeated argument triplets, reducing
  signature noise and future maintenance cost.
