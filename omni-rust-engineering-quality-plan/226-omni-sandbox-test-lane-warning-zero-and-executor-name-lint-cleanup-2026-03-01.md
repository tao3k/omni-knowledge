# 226. `omni-sandbox` Test-Lane Warning Zero and Executor-Name Lint Cleanup (2026-03-01)

## Scope

- Remove suppression-based handling in touched `omni-sandbox` executor code.
- Converge `omni-sandbox` integration tests to warning-zero under strict clippy
  without broad lint suppression.
- Preserve existing runtime behavior while normalizing test layout/style.

## Changes

1. Executor name API cleanup (`unused_self` suppression removal)
- Files:
  - `packages/rust/crates/omni-sandbox/src/executor/nsjail.rs`
  - `packages/rust/crates/omni-sandbox/src/executor/seatbelt.rs`
- Removed `#[allow(clippy::unused_self)]` on `name(&self)` methods.
- Updated both methods to delegate to the trait implementation:
  - `<Self as SandboxExecutor>::name(self)`
- Result: suppression removed with behavior preserved.

2. Integration test warning-zero convergence
- File: `packages/rust/crates/omni-sandbox/tests/test_sandbox.rs`
- Removed redundant inline `#[cfg(test)] mod tests` wrapper and kept all tests
  as top-level integration tests under `tests/`.
- Replaced warning-prone patterns:
  - `expect()` in tempdir test -> `Result`-returning test with `?`
  - `"".to_string()` -> `String::new()`
  - format string with positional argument -> inline format args (`{platform}`)
  - underscore binding usage (`_name`) -> direct assertions
- Added crate-level integration-test doc header to satisfy `missing-docs` in
  strict clippy test targets.

## Validation Evidence

1. Strict clippy (all targets)

```bash
cargo clippy -p omni-sandbox --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Targeted integration test lane

```bash
cargo nextest run -p omni-sandbox --test test_sandbox
```

- Exit code: `0`
- Result: `17 passed`, `0 failed`

## Outcome

- `omni-sandbox` touched slice is now suppression-free for the addressed
  executor name methods.
- `test_sandbox` lane is strict-clippy clean (warning-zero) and fully passing.
- Test structure now conforms to top-level `tests/` integration style without
  inline module wrappers.
