# 360. xiuxian-qianji runtime-config directory moduleization and deterministic env-override semantics (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/runtime_config.rs`
  - added:
    - `src/runtime_config/mod.rs`
    - `src/runtime_config/constants.rs`
    - `src/runtime_config/model.rs`
    - `src/runtime_config/toml_config.rs`
    - `src/runtime_config/env_vars.rs`
    - `src/runtime_config/pathing.rs`
    - `src/runtime_config/loader.rs`
    - `src/runtime_config/resolve.rs`
- Goal:
  - replace monolithic runtime-config module with concern-split structure,
    preserve public runtime-config API, and harden test determinism in
    environment override handling.

## Implementation

1. Converted `runtime_config` to directory module:
   - `mod.rs` is interface-only, re-exporting:
     - `QianjiRuntimeEnv`
     - `QianjiRuntimeLlmConfig`
     - `QianjiRuntimeWendaoIngesterConfig`
     - `resolve_qianji_runtime_llm_config(_with_env)`
     - `resolve_qianji_runtime_wendao_ingester_config(_with_env)`
2. Split responsibilities by concern:
   - `constants.rs`: default constants.
   - `model.rs`: public runtime models.
   - `toml_config.rs`: TOML schema and overlay merge logic.
   - `pathing.rs`: project/config path resolution.
   - `loader.rs`: config candidate discovery and file parse flow.
   - `env_vars.rs`: environment override parsing and normalization.
   - `resolve.rs`: public resolver entrypoints.
3. Determinism fix for override semantics:
   - introduced explicit override-state handling (`missing`/`empty`/`value`)
     for `runtime_env.extra_env`.
   - ensured empty explicit overrides can block ambient process env leakage
     where intended (especially runtime-config tests under polluted shell env),
     while preserving precedence where named API-key overrides remain valid.
4. No public API signature changes.
5. No broad lint suppressions added.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/runtime_config/mod.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/constants.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/model.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/toml_config.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/env_vars.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/pathing.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/loader.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/resolve.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Runtime-config regression lane:
  - `cargo nextest run -p xiuxian-qianji --test runtime_config`
  - result: `9 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `runtime_config` now follows repository modularization rules with explicit
  domain boundaries and interface-only module entrypoint.
- Environment override behavior for test-injected runtime env is deterministic
  under shell environments that already export API-key variables.
