# 修仙道场 (Xiuxian Daochang) Format-Collect Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::format_collect` file-level allow markers and fixing surfaced source
warnings with idiomatic construction patterns.

## Implemented Changes

1. Removed `clippy::format_collect` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced real warning:
   - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_delivery.rs`
   - replaced iterator `map(format!).collect::<String>()` with
     `fold(String::new(), ...)` + `write!` accumulation.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::format_collect" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::format_collect` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::format_collect` with strict pedantic
and `too_many_lines` validation still green.
