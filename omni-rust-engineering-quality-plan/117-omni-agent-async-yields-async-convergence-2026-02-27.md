# 修仙道场 (Xiuxian Daochang) Async-Yields-Async Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::async_yields_async` file-level allow markers and fixing surfaced
runtime-task composition issues at source level.

## Implemented Changes

1. Removed `clippy::async_yields_async` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced real error in reconnect test:
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_pool_reconnect.rs`
   - replaced `tokio::spawn(async { ... spawn_mock_server(...).await })`
     (async yielding awaitable handle) with:
     - delayed spawn task that sends the server handle through
       `tokio::sync::oneshot`,
     - explicit receive path for the returned handle before cleanup.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::async_yields_async" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::async_yields_async` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::async_yields_async` with strict
pedantic and `too_many_lines` validation still green.
