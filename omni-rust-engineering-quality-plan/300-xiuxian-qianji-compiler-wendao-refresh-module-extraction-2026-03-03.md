# 300. xiuxian-qianji compiler wendao-refresh module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area: `src/engine/compiler.rs`
- Goal: move `wendao_refresh` parameter decoding out of the compiler core and
  keep the main builder focused on orchestration only.

## Implementation

1. Added dedicated module:
   - `src/engine/compiler/wendao_refresh.rs`
   - introduced:
     - `WendaoRefreshConfig`
     - `mechanism_config(node_def)`
     - focused helpers for bool/string/list parameter extraction
2. Updated compiler wiring:
   - `compiler.rs` declares `mod wendao_refresh;`
   - `build_wendao_refresh_mechanism` now consumes
     `wendao_refresh::mechanism_config(node_def)` and only constructs
     `WendaoRefreshMechanism`.
3. Preserved behavior defaults:
   - `output_key` default: `wendao_refresh`
   - `changed_paths_key` default: `changed_paths`
   - same defaults and trimming behavior for all optional params.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/formal_audit.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/llm_node.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/wendao_refresh.rs`
  - result:
    - `compiler.rs`: `707`
    - `compiler/formal_audit.rs`: `94`
    - `compiler/llm_node.rs`: `145`
    - `compiler/wendao_refresh.rs`: `78`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler remains behavior-compatible while `wendao_refresh` parsing logic is
  isolated into a stable, domain-named submodule.
