# 234. `xiuxian-daochang` Runtime Agent Factory Tests Top-Level Harness Migration (2026-03-01)

## Scope

- Remove `runtime_agent_factory` test mounting from `src` (`#[cfg(test)]` + `#[path]`).
- Keep tests under package-top `tests/` as the single execution surface.
- Preserve behavior and strict-clippy cleanliness without broad lint suppressions.

## Changes

1. Removed `src`-side test mounting in runtime agent factory module

- File: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mod.rs`
- Removed:
  - test-only imports (`LITELLM_DEFAULT_URL`, `McpServerEntry`, test helper imports)
  - `#[cfg(test)] #[path = "../../tests/runtime_agent_factory/inference.rs"] mod tests;`

2. Added top-level integration test harness

- File: `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory.rs`
- Added a dedicated harness that mounts only required source modules and exposes
  test-facing symbols for `tests/runtime_agent_factory/inference.rs`.
- Implemented minimal local `resolve` and `types` modules to satisfy the source
  module contracts without importing unrelated crate surfaces.

3. Added memory test adapter module for stable path resolution

- File: `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/memory.rs`
- Added explicit `#[path = "..."]` links to:
  - `src/runtime_agent_factory/memory/embedding.rs`
  - `src/runtime_agent_factory/memory/env_overrides.rs`
  - `src/runtime_agent_factory/memory/runtime.rs`
- Kept runtime-memory option assembly logic equivalent to source behavior.

4. Minor test cleanup to remain clippy-clean

- File: `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/inference.rs`
- Replaced underscore-bind no-op reads with `let _ = ...;` pattern to avoid
  `clippy::no_effect_underscore_binding`.

## Validation Evidence

1. Mandatory touched-crate strict clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors reported.

2. Migrated target strict clippy gate

```bash
cargo clippy -p xiuxian-daochang --test runtime_agent_factory -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: no warnings/errors reported.

3. Migrated target nextest gate

```bash
cargo nextest run -p xiuxian-daochang --test runtime_agent_factory
```

- Exit code: `0`
- Result: `18 passed`, `0 failed`.

## Outcome

- `runtime_agent_factory` test execution now follows top-level integration-test
  structure instead of `src`-side `cfg(test)` mounting.
- Migration is validated by strict clippy and nextest with zero suppressions and
  no remaining warnings in the touched lane.
