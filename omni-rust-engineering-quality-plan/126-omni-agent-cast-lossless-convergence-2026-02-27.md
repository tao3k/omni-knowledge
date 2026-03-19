# 修仙道场 (Xiuxian Daochang) Cast-Lossless Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::cast_lossless` file-level allow markers and fixing surfaced cast
warnings.

## Implemented Changes

1. Removed `clippy::cast_lossless` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced real warning:
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client.rs`
   - replaced lossless cast `*byte as u32` with `u32::from(*byte)`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::cast_lossless" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::cast_lossless` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::cast_lossless` with strict pedantic
and `too_many_lines` validation still green.
