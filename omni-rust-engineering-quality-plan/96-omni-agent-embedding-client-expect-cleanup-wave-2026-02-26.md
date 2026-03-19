# 修仙道场 (Xiuxian Daochang) Embedding Client Expect Cleanup Wave (2026-02-26)

## Scope

Converge `tests/embedding_client.rs` by removing remaining
`expect_used`/`unwrap_used` suppression debt.

## Implemented Changes

1. Removed file-level `clippy::expect_used` / `clippy::unwrap_used` in:
   - `tests/embedding_client.rs`
2. Replaced all `expect`-based `Option` extraction in embedding assertions with
   explicit helper-based handling:
   - added `require_vectors(...)` for readable and explicit failure branches.

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
- marker-file count in `xiuxian-daochang/tests` dropped from `7` to `6`.
- workspace sibling warnings remained unchanged in:
  - `xiuxian-wendao`
  - `xiuxian-zhixing`
  - `xiuxian-qianhuan`

## Outcome

Embedding client test lane is suppression-free for
`expect_used`/`unwrap_used`; marker baseline reduced to `6`.
