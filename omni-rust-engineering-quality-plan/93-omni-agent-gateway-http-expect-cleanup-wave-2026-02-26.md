# 修仙道场 (Xiuxian Daochang) Gateway HTTP Expect Cleanup Wave (2026-02-26)

## Scope

Continue queue-driven suppression cleanup by converging
`tests/gateway_http.rs`.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used`.
2. Replaced all `expect`/`unwrap` call sites in
   `tests/gateway_http.rs` with explicit error handling.
3. Added local test helpers to avoid repetition while keeping failure paths
   explicit:
   - `build_agent_or_panic`
   - `request_or_panic`
   - `oneshot_or_panic`

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
- marker-file count in `xiuxian-daochang/tests` dropped from `11` to `10`.
- workspace sibling warnings remained unchanged in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`

## Outcome

Gateway HTTP tests are now suppression-free for
`expect_used`/`unwrap_used`; remaining marker files reduced to `10`.
