# 325. xiuxian-qianji compiler directory module entry migration (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs` (moved)
  - `src/engine/compiler/mod.rs` (new path)
- Goal: align compiler module layout with directory-module conventions by using
  a canonical `mod.rs` entrypoint for an already split module tree.

## Implementation

1. Migrated compiler module entry file:
   - moved `src/engine/compiler.rs` to `src/engine/compiler/mod.rs`.
2. Preserved module structure and behavior:
   - retained all existing submodule declarations and compiler orchestration
     logic without semantic changes.
3. Kept caller API stable:
   - `engine/mod.rs` continues to expose `pub mod compiler;`
   - external paths like `xiuxian_qianji::engine::compiler::QianjiCompiler`
     remain unchanged.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `6 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler internals now follow a clearer directory-first modular boundary.
- Entrypoint placement is consistent with ongoing split-module refactoring,
  reducing future structural drift in the `compiler/` tree.
