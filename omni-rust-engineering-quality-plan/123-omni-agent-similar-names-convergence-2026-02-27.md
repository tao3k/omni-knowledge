# 修仙道场 (Xiuxian Daochang) Similar-Names Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::similar_names` file-level allow markers and fixing surfaced naming
conflicts.

## Implemented Changes

1. Removed `clippy::similar_names` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed the surfaced real warning:
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs`
   - renamed paired bindings from `msg_a_shared`/`msg_b_shared` to
     `shared_from_alice`/`shared_from_bob`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::similar_names" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::similar_names` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::similar_names` with strict pedantic
and `too_many_lines` validation still green.
