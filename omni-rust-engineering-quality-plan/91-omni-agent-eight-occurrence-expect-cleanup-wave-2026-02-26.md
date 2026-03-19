# 修仙道场 (Xiuxian Daochang) Eight-Occurrence Expect Cleanup Wave (2026-02-26)

## Scope

Continue suppression-debt convergence by cleaning the eight-occurrence queue in
`xiuxian-daochang` tests.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced panic-style extraction with explicit handling in:
   - `tests/discord_acl_overrides.rs`
   - `tests/embedding_client_cache.rs`
   - `tests/telegram_acl_overrides.rs`
2. Fixed blocking source issues surfaced by full-lane verification:
   - `src/agent/turn_execution/react_loop/agenda_validation.rs`
   - replaced `and_then(Value::as_str)` with explicit closures
     (`and_then(|value| value.as_str())`) to avoid invalid trait-path
     resolution under the current build context.

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
- marker-file count in `xiuxian-daochang/tests` dropped from `16` to `13`.
- workspace sibling warnings remained unchanged in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`

## Outcome

Eight-occurrence queue is cleared; remaining marker files reduced to `13`.
