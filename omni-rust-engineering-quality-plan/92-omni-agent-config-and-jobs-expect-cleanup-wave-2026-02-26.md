# 修仙道场 (Xiuxian Daochang) Config and Jobs Expect Cleanup Wave (2026-02-26)

## Scope

Remove `expect/unwrap` suppression debt from the next focused pair of
`xiuxian-daochang` test files after the eight-occurrence wave.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   migrated to explicit `match`-based error handling in:
   - `tests/config_mcp.rs`
   - `tests/jobs_manager.rs`
2. Added small local test helpers in `config_mcp.rs` to keep setup readable
   while preserving explicit failure paths:
   - temp-dir creation
   - JSON fixture write
3. Fixed route-loop compile regression exposed during revalidation:
   - `src/agent/turn_execution/react_loop/agenda_validation.rs`
   - restored `should_run_agenda_validation` symbol used by
     `react_loop/tests.rs`.

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
- marker-file count in `xiuxian-daochang/tests` dropped from `13` to `11`.
- workspace sibling warnings remained unchanged in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`

## Outcome

Config lane and jobs lane are now suppression-free for
`expect_used`/`unwrap_used`; remaining marker files reduced to `11`.
