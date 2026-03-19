# 修仙道场 (Xiuxian Daochang) Cast-Precision-Loss Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::cast_precision_loss` file-level allow markers and fixing surfaced
integer-to-float conversion warnings.

## Implemented Changes

1. Removed `clippy::cast_precision_loss` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced precision-loss warnings in test lanes:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/embedding_dimension.rs`
     - switched generator range to `u16` and used `f32::from(idx)`.
   - `packages/rust/crates/xiuxian-daochang/tests/agent/reflection.rs`
     - replaced float coverage ratio with integer percentage threshold checks.
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client_cache.rs`
     - narrowed `usize` lengths/indexes with checked `u16::try_from(...)`,
       then converted via `f32::from(...)`.
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client.rs`
     - narrowed modulo score to `u16` before float conversion.
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`
     - introduced `usize_to_f64` helper (`u32` checked narrowing + `f64::from`)
       and removed direct `usize as f64` casts in percentile/report paths.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::cast_precision_loss" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::cast_precision_loss` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::cast_precision_loss` while keeping
strict pedantic and `too_many_lines` lanes clean.
