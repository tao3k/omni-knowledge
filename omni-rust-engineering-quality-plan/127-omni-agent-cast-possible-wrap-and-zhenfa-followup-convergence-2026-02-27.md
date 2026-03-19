# 修仙道场 (Xiuxian Daochang) Cast-Possible-Wrap and Zhenfa Follow-Up Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::cast_possible_wrap` file-level allow markers, fixing surfaced cast
warnings, and closing follow-up pedantic warnings in zhenfa bootstrap/bridge
paths.

## Implemented Changes

1. Removed `clippy::cast_possible_wrap` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced cast warnings:
   - `tests/telegram_runtime/mod.rs`
     - replaced `u64 -> i64` `as` cast with `i64::try_from(...).unwrap_or(i64::MAX)`.
   - `tests/channels_webhook_stress.rs`
     - replaced repeated `usize -> i64` casts with checked conversion helper
       (`usize_to_i64`) returning `anyhow::Result<i64>`.
3. Closed follow-up pedantic warnings in zhenfa integration:
   - `src/agent/zhenfa/bridge.rs`
     - changed `from_xiuxian_config` from `Result<Option<_>>` to `Option<_>`
       (`clippy::unnecessary_wraps`).
     - changed `build_native_tool` to take `&ZhenfaToolSpec`
       (`clippy::needless_pass_by_value`).
   - `src/agent/bootstrap/zhenfa.rs`
     - replaced single-pattern `match` with `if let`
       (`clippy::single_match_else`).
   - `tests/zhenfa_tool_bridge.rs`
     - switched global serialization lock to `tokio::sync::Mutex` and async lock
       acquisition (`clippy::await_holding_lock`).
4. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::cast_possible_wrap" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::cast_possible_wrap` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::cast_possible_wrap`, and zhenfa
runtime/bridge follow-up warnings were resolved while keeping strict clippy
lanes clean.
