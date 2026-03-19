# Omni-Memory Cast Precision Convergence (2026-02-26)

## Scope

This shard records a focused convergence pass to remove remaining
`cast_precision_loss` suppressions in `omni-memory` production sources.

Targets:

- `packages/rust/crates/omni-memory/src/episode.rs`
- `packages/rust/crates/omni-memory/src/gate.rs`

## Changes Implemented

### 1) Removed all remaining cast-precision suppressions

Actions:

- Deleted all `#[allow(clippy::cast_precision_loss)]` in the two target files.

### 2) Replaced direct numeric casts with explicit conversion helpers

Actions:

- Added `num-traits` dependency to `omni-memory`:
  - `packages/rust/crates/omni-memory/Cargo.toml`
- Introduced `ToPrimitive`-based conversion helpers:
  - `u32 -> f32` for usage/success/failure counters,
  - `i64 -> f32` saturation path for age-millis conversion.

Rationale:

- Keep float-based scoring behavior while making conversion intent explicit and
  eliminating pedantic cast warnings without suppression attributes.

### 3) Small refactor for consistency

Actions:

- Reused `age_hours()` inside `apply_time_decay()` to avoid duplicated
  conversion logic and keep one conversion path for time decay.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p omni-memory
cargo clippy -p omni-memory --all-targets -- -W clippy::pedantic
cargo test -p omni-memory --tests
```

Additional check:

```bash
rg -n "allow\\(clippy::" packages/rust/crates/omni-memory/src
```

Result:

- No remaining `allow(clippy::...)` attributes in `omni-memory/src`.

## Outcome

- `omni-memory` production source reached suppression-free pedantic convergence
  for this wave.
- Full test lane remains green after conversion cleanup.
