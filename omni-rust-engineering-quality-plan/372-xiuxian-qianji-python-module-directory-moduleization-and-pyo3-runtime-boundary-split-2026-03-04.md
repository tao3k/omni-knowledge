# 372. xiuxian-qianji python module directory moduleization and pyo3 runtime boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/python_module.rs`
  - added:
    - `src/python_module/mod.rs`
    - `src/python_module/engine.rs`
    - `src/python_module/scheduler.rs`
    - `src/python_module/runtime.rs`
    - `src/python_module/llm_bridge.rs`
- Goal:
  - replace mixed PyO3 module implementation with concern-split directory
    modules while preserving Python-facing API and feature-gated behavior.

## Implementation

1. Converted `python_module` to a directory module:
   - `mod.rs` is now interface-only and owns `_xiuxian_qianji` module
     registration (`add_class`/`add_function`).
2. Split Python binding concerns by domain:
   - `engine.rs`:
     - `PyQianjiEngine` wrapper and mock-node/link graph wiring.
   - `scheduler.rs`:
     - `PyQianjiScheduler` wrapper and `run` execution path.
   - `runtime.rs`:
     - shared JSON parse/serialize and Tokio runtime creation helpers.
   - `llm_bridge.rs` (`feature = "llm"`):
     - `run_master_research_array` bridge for LLM-enabled master workflow.
3. Preserved feature boundaries:
   - `llm_bridge` and exported pyfunction remain gated by `feature = "llm"`.
   - include path corrected for new module location:
     `../../resources/research_master.toml`.
4. Public API compatibility preserved:
   - Python module name `_xiuxian_qianji`, classes `QianjiEngine`,
     `QianjiScheduler`, and optional `run_master_research_array` unchanged.
5. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/python_module/mod.rs packages/rust/crates/xiuxian-qianji/src/python_module/engine.rs packages/rust/crates/xiuxian-qianji/src/python_module/runtime.rs packages/rust/crates/xiuxian-qianji/src/python_module/scheduler.rs packages/rust/crates/xiuxian-qianji/src/python_module/llm_bridge.rs`
  - result: pass
- Tier-2 compile checks:
  - `cargo check -p xiuxian-qianji`
  - result: pass
  - `cargo check -p xiuxian-qianji --features pyo3`
  - result: pass
  - `cargo check -p xiuxian-qianji --features "pyo3 llm"`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
  - `cargo clippy -p xiuxian-qianji --features "pyo3 llm" -- -W clippy::too_many_lines`
  - result: pass
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- PyO3 binding surface now follows the same interface-only `mod.rs` standard
  used across the crate, with clear runtime/bridge/wrapper boundaries.
- Python-facing contracts remain stable, while internal runtime and JSON error
  mapping logic is centralized for easier maintenance.
