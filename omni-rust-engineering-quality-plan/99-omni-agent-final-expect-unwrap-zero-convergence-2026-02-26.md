# 修仙道场 (Xiuxian Daochang) Final Expect/Unwrap Zero Convergence (2026-02-26)

## Scope

Complete the final residual suppression cleanup in `xiuxian-daochang/tests` and
reach zero marker files for `clippy::expect_used|clippy::unwrap_used`.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced extraction paths with explicit handling in:
   - `tests/channels_telegram_group_policy.rs`
   - `tests/agent_injection.rs`
2. Follow-up cleanup:
   - fixed a post-refactor type mismatch in `agent_injection` by using
     `require_some` for optional budget snapshot extraction.
   - removed a follow-up `clippy::single_match` warning in
     `agent_injection` (`if is_ok` guard instead of empty `match` arm).

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | wc -l
```

Result:

- `xiuxian-daochang` test targets compile and pedantic verification remains green.
- marker-file count in `xiuxian-daochang/tests` dropped from `2` to `0`.
- workspace sibling warnings remain in non-target crates:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianji`
- note: `xiuxian-daochang` still has non-blocking test-target warnings in
  `config_settings`/`discord_acl_overrides`/`telegram_acl_overrides`
  (`clippy::collapsible_if`), unrelated to `expect/unwrap` suppression debt.

## Outcome

`xiuxian-daochang/tests` is now at zero occurrences for file-level
`clippy::expect_used|clippy::unwrap_used` suppressions.
