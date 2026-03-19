# 321. xiuxian-qianji mechanism dispatch unified resolver-chain interface (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/mechanism_dispatch.rs`
  - `src/engine/compiler/mechanism_dispatch/resolver_chain.rs` (new)
  - `src/engine/compiler/mechanism_dispatch/stateless.rs`
  - `src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs`
  - `src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs`
- Goal: replace ad-hoc chained `if let Some(...)` routing with a unified
  resolver-chain interface and consistent resolver signatures.

## Implementation

1. Added generic resolver-chain helper:
   - `mechanism_dispatch/resolver_chain.rs`
   - introduced:
     - `type ResolveOutcome = Result<Arc<dyn QianjiMechanism>, QianjiError>`
     - `type ResolverFn = fn(TaskType, &QianjiCompiler, &NodeDefinition) -> Option<ResolveOutcome>`
     - `run(resolvers, task_type, compiler, node_def) -> Option<ResolveOutcome>`
2. Unified resolver signatures:
   - `stateless::build(...)` now returns `Option<ResolveOutcome>` (wrapping
     mechanism construction in `Ok(...)`).
   - `stateful_cfg::build(...)` now returns `Option<ResolveOutcome>`.
   - `leaf_dispatch::build(...)` now accepts compiler context and returns
     `Option<ResolveOutcome>`.
3. Converted top-level dispatch to chain execution:
   - `mechanism_dispatch.rs` now uses:
     - `resolver_chain::run(&[stateless::build, stateful_cfg::build, leaf_dispatch::build], ...)`
   - kept explicit fallback topology error for impossible chain exhaustion.
4. Converted leaf-level dispatch to chain execution:
   - `leaf_dispatch/mod.rs` now routes through the same helper over:
     - `io_control::build`
     - `quality_guard::build`
     - `wendao_router::build`
   - retains prior leaf mismatch error semantics.
5. Cleanup:
   - removed now-unused imports introduced by signature unification.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/resolver_chain.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateless.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/stateful_cfg/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/io_control.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/quality_guard.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch/leaf_dispatch/wendao_router.rs`
  - result:
    - `mechanism_dispatch.rs`: `28`
    - `resolver_chain.rs`: `20`
    - `stateless.rs`: `24`
    - `stateful_cfg/mod.rs`: `20`
    - `leaf_dispatch/mod.rs`: `31`
    - `leaf_dispatch/io_control.rs`: `17`
    - `leaf_dispatch/quality_guard.rs`: `17`
    - `leaf_dispatch/wendao_router.rs`: `17`
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

- Dispatch flow now uses one consistent chain-of-responsibility pattern across
  both top-level and leaf-level routing.
- Resolver contract is explicit and reusable, reducing duplicate branching
  boilerplate and future expansion cost.
