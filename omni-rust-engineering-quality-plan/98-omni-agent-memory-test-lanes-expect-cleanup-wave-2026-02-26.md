# 修仙道场 (Xiuxian Daochang) Memory Test Lanes Expect Cleanup Wave (2026-02-26)

## Scope

Drive down the next residual test-lane suppression set by converging memory
gate and persistence backend tests.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` and
   replaced extraction paths with explicit handling in:
   - `tests/agent_memory_gate_flow.rs`
   - `tests/agent_memory_persistence_backend.rs`
2. Shifted async setup/append assertions from `expect` to `Result`-first and
   helper-based handling (`require_ok` / `require_some`) where signatures did
   not return `Result`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --tests --no-run
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | wc -l
```

Result:

- `xiuxian-daochang` tests still compile under pedantic verification.
- marker-file count in `xiuxian-daochang/tests` dropped from `4` to `2`.
- workspace sibling warnings remained in non-target crates
  (`xiuxian-wendao`, `xiuxian-zhixing`, `xiuxian-qianji`).

## Outcome

Memory gate and persistence test lanes are suppression-clean for
`expect_used`/`unwrap_used`; residual marker baseline reduced to `2`.
