# 修仙道场 (Xiuxian Daochang) Manual-Assert Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::manual_assert` file-level allow markers and fixing all surfaced
source warnings without adding new suppression attributes.

## Implemented Changes

1. Removed `clippy::manual_assert` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced real warnings:
   - `packages/rust/crates/xiuxian-daochang/tests/agent_injection.rs`
     - replaced `if ... { panic!(...) }` with `assert!(...)`.
   - `packages/rust/crates/xiuxian-daochang/tests/channels_idempotency.rs`
     - replaced loop guard `if ... { panic!(...) }` with `assert!(...)`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::manual_assert" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::manual_assert` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::manual_assert` with strict pedantic
and `too_many_lines` validation still green.
