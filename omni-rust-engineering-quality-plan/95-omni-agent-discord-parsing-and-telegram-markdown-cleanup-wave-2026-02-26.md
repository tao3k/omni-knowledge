# 修仙道场 (Xiuxian Daochang) Discord Parsing and Telegram Markdown Cleanup Wave (2026-02-26)

## Scope

Continue suppression convergence in message-format parsing/rendering tests.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced parse/error extraction with explicit handling in:
   - `tests/channels_discord_parsing.rs`
2. Removed now-stale `clippy::expect_used` / `clippy::unwrap_used` entries in:
   - `tests/channels_telegram_markdown.rs`
3. Resolved pedantic follow-up in `channels_discord_parsing`:
   - replaced `Ok(_)` with `Ok(())` in unit-result matching.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | wc -l
```

Result:

- `xiuxian-daochang` test targets compile and pedantic clippy stays green.
- marker-file count in `xiuxian-daochang/tests` dropped from `8` to `7`.
- workspace sibling warnings remained unchanged in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`

## Outcome

Discord parsing and Telegram markdown lanes are now suppression-clean for
`expect_used`/`unwrap_used`; marker baseline reduced to `7`.
