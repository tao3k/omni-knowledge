# 313. xiuxian-qianji compiler mechanism dispatch module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/mechanism_dispatch.rs` (new)
- Goal: extract task-type mechanism dispatch match from compiler core into a
  dedicated dispatch module so compiler remains an orchestration shell.

## Implementation

1. Added dedicated dispatch module:
   - `src/engine/compiler/mechanism_dispatch.rs`
   - introduced:
     - `build(&QianjiCompiler, &NodeDefinition) -> Result<Arc<dyn QianjiMechanism>, QianjiError>`
   - responsibilities:
     - parse task kind via `TaskType::parse(...)`.
     - dispatch to knowledge/annotation/formal_audit/llm paths on compiler.
     - dispatch leaf tasks to `task_mechanisms::*`.
2. Updated compiler wiring:
   - `compiler.rs` now declares `mod mechanism_dispatch;`
   - removed inline `build_mechanism(...)` match block from compiler impl.
   - `compile(...)` now delegates node mechanism construction to
     `mechanism_dispatch::build(self, node_def)`.
3. Internal visibility alignment:
   - promoted selected builder/dispatch methods to `pub(super)` to support
     module-level dispatch without exposing crate public API.
4. Size outcome:
   - compiler core line count reduced from `175` to `158`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/mechanism_dispatch.rs`
  - result:
    - `compiler.rs`: `158`
    - `compiler/mechanism_dispatch.rs`: `26`
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

- Compiler core now focuses on dependency wiring plus compile pipeline orchestration.
- Task-type routing responsibility is isolated and easier to evolve/test as a
  standalone dispatch domain.
