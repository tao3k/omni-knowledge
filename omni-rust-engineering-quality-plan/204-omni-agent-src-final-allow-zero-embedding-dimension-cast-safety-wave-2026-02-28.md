# 204. 修仙道场 (Xiuxian Daochang) Src Final Allow-Zero (Embedding-Dimension Cast Safety) Wave (2026-02-28)

## Scope

This wave removed the final remaining `#[allow(...)]` attribute in
`xiuxian-daochang/src`, completing source-level allow-zero convergence for the crate.

Target lane:

- `packages/rust/crates/xiuxian-daochang/src/agent/embedding_dimension.rs`

## What Changed

1. Removed cast-related clippy suppression on
   `repair_embedding_dimension`.
2. Replaced `as` numeric casts with explicit conversion paths using
   `num_traits::ToPrimitive`:
   - `usize -> f32` via `to_f32()`
   - `f32 -> usize` via `to_usize()`
3. Added bounds-safe fallback behavior for conversion edge cases and index
   clamping, preserving interpolation semantics.

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-daochang`
2. `CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines`
3. `CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang`

Outcomes:

- Strict clippy passed (exit code `0`).
- Full nextest passed (`653 passed`, `0 failed`, `30 skipped`).

## Result

`rg -n "#\\[allow\\(" packages/rust/crates/xiuxian-daochang/src` returns no matches.

This marks full allow-zero convergence in `xiuxian-daochang/src` for the current
modernization wave.
