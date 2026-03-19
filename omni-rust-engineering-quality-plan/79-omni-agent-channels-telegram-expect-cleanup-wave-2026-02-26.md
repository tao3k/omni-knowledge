# 修仙道场 (Xiuxian Daochang) `channels_telegram` Expect-Cleanup Wave (2026-02-26)

## Scope

Converge the large `channels_telegram` integration test target by removing
`expect`/`expect_err` usage and retiring stale `expect_used`/`unwrap_used`
file-level suppressions.

## Why

`channels_telegram.rs` is a broad regression surface (parsing, partitioning,
Markdown fallback, rate limiting, timeout behavior). Keeping panic-style
assertions hidden behind file-level suppressions weakens strict-gate quality.

## Implemented Changes

1. Removed file-level:
   - `clippy::expect_used`
   - `clippy::unwrap_used`
2. Replaced all `parse_update_message(...).expect(...)` paths by introducing a
   dedicated local helper (`require_some`) to enforce explicit option checks.
3. Replaced `find_map(...).expect(...)` in async send-rate-limit test with
   `ok_or_else(anyhow!(...))?`.
4. Replaced `expect_err(...)` in timeout test with explicit `let Err(error) =`
   branch handling and error-return path.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test channels_telegram
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `channels_telegram` test target passed: `29 passed; 0 failed`.
- `xiuxian-daochang` remained green under pedantic clippy.
- `xiuxian-daochang/tests` allow-marker file count dropped from `107` to `106`.

## Outcome

The highest-coverage Telegram channel test target now avoids hidden panic-style
assertions and contributes cleaner signal under strict lint gates.
