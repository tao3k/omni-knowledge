# 383. xiuxian-qianji runtime-config resolve directory moduleization convergence (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed `src/runtime_config/resolve.rs`
  - finalized `src/runtime_config/resolve/mod.rs`
  - finalized `src/runtime_config/resolve/llm.rs`
  - finalized `src/runtime_config/resolve/wendao.rs`
- Goal:
  - complete the half-migrated `runtime_config::resolve` split into a
    directory module, eliminate duplicate module source ambiguity, and keep
    strict lint/test gates green without suppression attributes.

## Implementation

1. Completed module-boundary migration:
   - removed legacy monolithic file `src/runtime_config/resolve.rs`,
   - kept only directory-module entry `src/runtime_config/resolve/mod.rs`,
   - preserved existing public API surface exported by
     `runtime_config::resolve::*`.

2. Concern split is now explicit and stable:
   - `resolve/llm.rs`:
     - LLM profile resolution (`llm` feature and non-`llm` fallback branches),
   - `resolve/wendao.rs`:
     - memory-promotion/Wendao ingest defaults resolution,
   - `resolve/mod.rs`:
     - runtime file-loading orchestration and public wrappers.

3. Root-cause pedantic cleanup (no `allow` usage):
   - resolved `clippy::unnecessary_wraps` by making internal helper
     `resolve_qianji_runtime_wendao_ingester(...)` return
     `QianjiRuntimeWendaoIngesterConfig` directly,
   - preserved public `io::Result<...>` API contract in wrapper layer by
     returning `Ok(...)` from `resolve/mod.rs`.

4. No lint-suppression attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/runtime_config/resolve/mod.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/resolve/llm.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/resolve/wendao.rs packages/rust/crates/xiuxian-qianji/src/runtime_config/mod.rs`
  - result: pass
- Type/syntax gate:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Extended lint surface check:
  - `cargo clippy -p xiuxian-qianji --all-targets --features llm -- -W clippy::too_many_lines`
  - result: pass for `xiuxian-qianji` (workspace still reports existing
    `xiuxian-llm` dead-code warnings; not modified in this slice)
- Regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test runtime_config --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `28 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `runtime_config::resolve` is fully migrated to directory modules with no
  duplicate old/new source coexistence.
- Internal responsibilities are cleaner (`llm` vs `wendao`), while external
  runtime-config APIs remain stable.
- Strict lint and targeted regression gates stay green for this convergence
  wave.
