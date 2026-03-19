# 修仙道场 (Xiuxian Daochang) Uninlined-Format-Args Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::uninlined_format_args` file-level allow markers and fixing surfaced
format-argument warnings.

## Implemented Changes

1. Removed `clippy::uninlined_format_args` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced pedantic warnings after marker removal:
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram.rs`
     - updated assertion message from positional formatting to inline capture:
       `"expected timeout error, got: {error}"`.
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_role_perf_smoke.rs`
     - updated `bail!` message from `{:?}` positional formatting to inline
       capture: `"gateway health not ready within {timeout:?}: {health_url}"`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::uninlined_format_args" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::uninlined_format_args` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::uninlined_format_args` with strict
pedantic and `too_many_lines` lanes remaining clean.
