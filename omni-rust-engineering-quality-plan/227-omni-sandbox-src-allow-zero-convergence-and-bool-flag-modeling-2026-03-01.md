# 227. `omni-sandbox` `src` Allow-Zero Convergence and Bool-Flag Modeling (2026-03-01)

## Scope

- Eliminate remaining source-level lint suppressions in `omni-sandbox`.
- Replace suppression-based patterns with structural code changes.
- Revalidate strict clippy and targeted integration tests.

## Changes

1. Removed unsafe derive suppressions on Python-exposed models
- File: `packages/rust/crates/omni-sandbox/src/executor/mod.rs`
- Removed:
  - `#[allow(clippy::unsafe_derive_deserialize)]` on `SandboxConfig`
  - `#[allow(clippy::unsafe_derive_deserialize)]` on `MountConfig`
- Structural fix:
  - Dropped `Deserialize` derive from both `#[pyclass]` structs.
  - Kept Python constructor-based parsing paths unchanged.
  - Removed now-unused `serde::Deserialize` import from this module.

2. Removed `struct_excessive_bools` suppression in nsjail JSON model
- File: `packages/rust/crates/omni-sandbox/src/executor/nsjail.rs`
- Removed:
  - `#[allow(clippy::struct_excessive_bools)]` on `NsJailJsonConfig`
- Structural fix:
  - Introduced `BoolFlag` newtype:
    - `#[derive(Debug, Clone, Copy, Deserialize, Default)]`
    - `#[serde(transparent)]`
  - Replaced four `bool` namespace fields with `BoolFlag` fields:
    - `clone_newnet`, `clone_newuser`, `clone_newpid`, `clone_newns`
  - Updated command assembly checks to `is_enabled()` accessors.

3. Source suppression status
- Verified `omni-sandbox/src` has no remaining `#[allow(...)]` attributes.

## Validation Evidence

1. Strict clippy (all targets)

```bash
cargo clippy -p omni-sandbox --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Targeted integration tests

```bash
cargo nextest run -p omni-sandbox --test test_sandbox
```

- Exit code: `0`
- Result: `17 passed`, `0 failed`

## Outcome

- `omni-sandbox/src` reached allow-zero convergence for the touched suppression
  categories.
- `NsJailJsonConfig` now uses a typed flag model instead of raw boolean-field
  accumulation.
- Quality gates remain green after refactor.
