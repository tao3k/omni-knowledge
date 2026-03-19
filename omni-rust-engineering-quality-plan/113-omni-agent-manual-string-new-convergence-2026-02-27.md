# 修仙道场 (Xiuxian Daochang) Manual-String-New Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt reduction by removing
`clippy::manual_string_new` allow markers and fixing surfaced source warnings
without adding new suppression attributes.

## Implemented Changes

1. Removed `clippy::manual_string_new` allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed the two surfaced real warnings:
   - `packages/rust/crates/xiuxian-daochang/tests/gateway/http/runtime.rs`
     - replaced `\"\".to_string()` with `String::new()` in test settings args.
   - `packages/rust/crates/xiuxian-daochang/tests/channels_idempotency.rs`
     - replaced `\"\".to_string()` with `String::new()` for Redis key prefix.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::manual_string_new" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::manual_string_new` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::manual_string_new` with strict
pedantic and `too_many_lines` validation still green.
