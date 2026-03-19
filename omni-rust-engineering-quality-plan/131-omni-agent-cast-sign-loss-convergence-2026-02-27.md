# 修仙道场 (Xiuxian Daochang) Cast-Sign-Loss Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::cast_sign_loss` file-level allow markers and fixing surfaced
float-to-index conversion warnings.

## Implemented Changes

1. Removed `clippy::cast_sign_loss` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced cast warnings:
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_discover_cache.rs`
     - replaced `f64 -> usize` percentile rank conversion with integer math:
       `sorted.len().saturating_mul(95).div_ceil(100)`.
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`
     - replaced `rank.floor()/ceil() as usize` with index selection over range
       iterators, avoiding direct signed-loss float casts.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::cast_sign_loss" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::cast_sign_loss` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::cast_sign_loss` with strict pedantic
and `too_many_lines` lanes remaining clean.
