# 374. xiuxian-qianji contracts directory module boundary split for execution-bindings-manifest-and-mechanism (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - updated:
    - `src/contracts/mod.rs`
  - added:
    - `src/contracts/execution.rs`
    - `src/contracts/bindings.rs`
    - `src/contracts/manifest.rs`
    - `src/contracts/mechanism.rs`
- Goal:
  - convert mixed contract declarations into domain-focused modules while
    preserving all external type paths and serde compatibility.

## Implementation

1. Converted `contracts/mod.rs` into interface-only module entry:
   - now declares and re-exports:
     - `execution`
     - `bindings`
     - `manifest`
     - `mechanism`
2. Split contract concerns by domain:
   - `execution.rs`:
     - `NodeStatus`, `FlowInstruction`, `QianjiOutput`.
   - `bindings.rs`:
     - `NodeQianhuanExecutionMode`, `NodeQianhuanBinding`, `NodeLlmBinding`.
   - `manifest.rs`:
     - `NodeDefinition`, `EdgeDefinition`, `QianjiManifest`.
   - `mechanism.rs`:
     - `QianjiMechanism` async trait.
3. Preserved compatibility details:
   - `NodeDefinition.llm` keeps `#[serde(alias = "llm_config")]`.
   - All public symbols still reachable via `xiuxian_qianji::contracts::*`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/contracts/mod.rs packages/rust/crates/xiuxian-qianji/src/contracts/execution.rs packages/rust/crates/xiuxian-qianji/src/contracts/bindings.rs packages/rust/crates/xiuxian-qianji/src/contracts/manifest.rs packages/rust/crates/xiuxian-qianji/src/contracts/mechanism.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Contracts-focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_schema_contracts --test manifest_requires_llm`
  - result: `9 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Contracts layer now follows strict interface-only `mod.rs` policy and has
  explicit domain boundaries for execution model, node bindings, manifest
  schema, and mechanism trait definition.
- External API and parser behavior remain stable under focused schema/manifest
  tests and core dispatch regression lanes.
