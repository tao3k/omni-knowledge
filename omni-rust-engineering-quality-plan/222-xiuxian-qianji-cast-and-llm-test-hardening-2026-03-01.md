# 222. `xiuxian-qianji` Cast and LLM-Test Hardening (2026-03-01)

## Scope

- Remove `allow`-based cast suppression in `xiuxian-qianji` production code.
- Eliminate `unwrap`/`expect` hard failures in `--all-features` clippy lanes.
- Preserve behavior with targeted regression checks.

## Changes

1. Removed cast-suppression debt in production paths
- `packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs`
  - Reworked `formal_audit_threshold_score` to deserialize JSON into `f32`
    directly (`serde_json::from_value`) and keep finite/range validation.
  - Reworked router branch parsing to use array-safe extraction and JSON->`f32`
    conversion without `as` narrowing casts.
  - Updated `to_branch_weight` to accept JSON value input and validate finite
    `f32` values.
- `packages/rust/crates/xiuxian-qianji/src/executors/router.rs`
  - Removed cast helper + `#[allow(clippy::cast_possible_truncation)]`.
  - Added JSON-driven `f32` extraction helper (`context_f32`).
- `packages/rust/crates/xiuxian-qianji/src/executors/calibration.rs`
  - Removed cast helper + `#[allow(clippy::cast_possible_truncation)]`.
  - Added JSON-driven `f32` extraction helper (`context_f32`).

2. Removed `unused_self` suppression in safety guard
- `packages/rust/crates/xiuxian-qianji/src/safety/mod.rs`
  - Removed `#[allow(clippy::unused_self)]` from `audit_topology`.
  - Included `max_loop_iterations` in cycle error message so `self` is
    semantically used.

3. Fixed all-features hard clippy errors (`unwrap_used` / `expect_used`)
- `packages/rust/crates/xiuxian-qianji/src/python_module.rs`
  - Replaced two `tokio::runtime::Runtime::new().unwrap()` sites with explicit
    `PyRuntimeError` propagation.
  - Migrated deprecated `py.allow_threads` usage to `py.detach`.
  - Renamed unused module argument `py` to `_py`.
- `packages/rust/crates/xiuxian-qianji/tests/llm_analyzer.rs`
  - Replaced `expect(...)` usage with explicit helper-based handling
    (`must_ok`, `must_array`).
- `packages/rust/crates/xiuxian-qianji/tests/test_bootcamp_api.rs`
  - Replaced JSON parse `expect(...)` with explicit panic path carrying error
    details.
- `packages/rust/crates/xiuxian-qianji/src/lib.rs`
  - Fixed `doc_markdown` wording by backticking `PyO3`.

## Validation Evidence

1. Strict clippy (default target set)

```bash
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy (`--all-features`)

```bash
cargo clippy -p xiuxian-qianji --all-targets --all-features -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: prior `unwrap_used` / `expect_used` errors removed.

3. Targeted nextest (`llm` lane touched by expect cleanup)

```bash
cargo nextest run -p xiuxian-qianji --features llm --test llm_analyzer
```

- Exit code: `0`
- Result: `6 passed`, `0 failed`

4. Targeted mechanism/safety regression

```bash
cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test unit_qianji_safety --test unit_adversarial_loop --test test_qianji_trinity_integration
```

- Exit code: `0`
- Result: `5 passed`, `0 failed`

## Outcome

- Production cast-suppression allows in `xiuxian-qianji` are removed.
- `--all-features` clippy no longer fails on `unwrap_used`/`expect_used`.
- Touched execution, safety, and LLM analyzer lanes remain behaviorally green.
