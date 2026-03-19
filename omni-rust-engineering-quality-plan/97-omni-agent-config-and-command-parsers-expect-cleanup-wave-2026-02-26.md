# 修仙道场 (Xiuxian Daochang) Config and Command Parsers Expect Cleanup Wave (2026-02-26)

## Scope

Continue suppression-debt reduction from the `6`-file residual baseline in
`xiuxian-daochang/tests`.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   converted to explicit handling in:
   - `tests/config_settings.rs`
   - `tests/channels_commands.rs`
2. Added small local helpers for explicit parse/tempdir/error handling without
   reintroducing panic-style extraction APIs.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | wc -l
```

Result:

- test target compilation and pedantic clippy remain green for `xiuxian-daochang`.
- marker-file count in `xiuxian-daochang/tests` dropped from `6` to `4`.
- workspace sibling warnings remained in non-target crates
  (`xiuxian-wendao`, `xiuxian-zhixing`, `xiuxian-qianji`).

## Outcome

Config/command parser test lanes are suppression-clean for
`expect_used`/`unwrap_used`, with residual marker baseline reduced to `4`.
