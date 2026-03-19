# 修仙道场 (Xiuxian Daochang) Cast-Conversion Convergence (2026-02-26)

## Scope

This shard records the cast-related suppression cleanup wave in `xiuxian-daochang`,
focused on removing:

- `clippy::cast_precision_loss`
- `clippy::cast_sign_loss`
- `clippy::cast_possible_truncation`

Targets:

- `packages/rust/crates/xiuxian-daochang/Cargo.toml`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_metrics.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/ranking.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/planning.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_feedback.rs`

## Changes Implemented

### 1) Introduced explicit numeric conversion strategy

Actions:

- Added `num-traits` dependency to `xiuxian-daochang`.
- Switched lossy integer/float conversions from `as` casts to
  `ToPrimitive`-based conversions with bounded fallback behavior.

### 2) Replaced cast-heavy ratio/pressure calculations

Actions:

- `memory_recall_metrics.rs`:
  - Replaced `numerator as f32 / denominator as f32` with
    `to_f32().unwrap_or(f32::MAX)` conversion flow in `ratio_as_f32`.
- `planning.rs`:
  - Replaced direct `usize -> f32` casts with a helper
    `ratio_usize_as_f32(...)` using `ToPrimitive`.
- `ranking.rs`:
  - Replaced `i64 -> f32` cast in recency computation with
    `to_f32().unwrap_or(f32::MAX)`.

### 3) Removed float-to-usize cast warnings in feedback tuning

Actions:

- `memory_recall_feedback.rs`:
  - Replaced `(240.0 * strength) as usize` and
    `(160.0 * strength) as usize` with
    rounded `to_usize().unwrap_or(usize::MAX)` deltas.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-daochang --lib
rg -n "allow\\(clippy::(cast_precision_loss|cast_sign_loss|cast_possible_truncation|struct_field_names)\" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests
rg -o "allow\\(clippy::[a-z0-9_]+(?:, clippy::[a-z0-9_]+)*\\)" \
  packages/rust/crates/xiuxian-daochang/src \
  packages/rust/crates/xiuxian-daochang/tests \
| sed -E 's/.*allow\\(//; s/\\)//' | tr ',' '\\n' \
| sed -E 's/^\\s*clippy:://; s/^\\s+|\\s+$//g' | sort | uniq -c | sort -nr
```

Results:

- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass for
  `xiuxian-daochang` (workspace still reports other-crate warnings outside this scope).
- `cargo test -p xiuxian-daochang --lib`: pass (`224 passed`, `0 failed`, `8 ignored`).
- Cast-related suppression categories in `xiuxian-daochang`: eliminated.
- Remaining suppression inventory in `xiuxian-daochang`:
  - `struct_field_names`: 3

## Outcome

- Numeric conversion paths are now explicit, bounded, and reviewable.
- Cast-related suppression debt in `xiuxian-daochang` is fully removed.
- `xiuxian-daochang` suppression debt is now concentrated only in naming-shape
  compatibility (`struct_field_names`).
