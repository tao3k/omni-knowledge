# 320. xiuxian-qianji `task_mechanisms` interface-only re-export via compiler-scope visibility (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler/task_mechanisms/mod.rs`
  - `src/engine/compiler/task_mechanisms/io_control.rs`
  - `src/engine/compiler/task_mechanisms/quality.rs`
  - `src/engine/compiler/task_mechanisms/wendao_router.rs`
- Goal: remove forwarding wrapper boilerplate from `task_mechanisms/mod.rs` and
  keep it as a pure interface re-export module.

## Implementation

1. Switched child function visibility to compiler-scope:
   - in submodules, changed function visibility from `pub(super)` to
     `pub(in crate::engine::compiler)`.
   - keeps exposure bounded to compiler namespace, without widening to public API.
2. Simplified interface module:
   - `task_mechanisms/mod.rs` now only contains:
     - module declarations
     - `pub(super) use ...` re-exports
   - removed all forwarding wrapper functions.
3. Preserved external call surface:
   - callers still use `task_mechanisms::{...}` with unchanged function names.
   - behavior and return types remain unchanged.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/mod.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/io_control.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/quality.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/task_mechanisms/wendao_router.rs`
  - result:
    - `task_mechanisms/mod.rs`: `7`
    - `task_mechanisms/io_control.rs`: `35`
    - `task_mechanisms/quality.rs`: `33`
    - `task_mechanisms/wendao_router.rs`: `46`
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

- `task_mechanisms/mod.rs` now fully follows the “interface-only `mod.rs`”
  standard.
- Boilerplate forwarding logic removed while keeping strict module-level
  visibility boundaries and stable behavior.
