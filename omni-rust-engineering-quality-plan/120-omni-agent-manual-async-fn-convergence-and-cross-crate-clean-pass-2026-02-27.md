# 修仙道场 (Xiuxian Daochang) Manual-Async-Fn Convergence and Cross-Crate Clean Pass (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::manual_async_fn` file-level allow markers, fixing surfaced source
warnings, and restoring cross-crate strict-clippy clean status.

## Implemented Changes

1. Removed `clippy::manual_async_fn` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced `manual_async_fn` warnings in:
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_pool_hard_timeout.rs`
   - converted trait impl methods from manual `fn -> impl Future` wrappers to
     direct `async fn` bodies.
3. Follow-up cross-crate cleanup needed to keep strict runs clean:
   - `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/payload/core.rs`
     - completed static helper migration by updating both call sites to
       `Self::build_planned_payload(...)`.
   - `packages/rust/crates/xiuxian-wendao/src/enhancer/resource_registry.rs`
     - replaced redundant closure with method reference
       (`map(str::to_ascii_lowercase)`).
4. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::manual_async_fn" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-wendao
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::manual_async_fn` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`: pass (`0`).
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::manual_async_fn`, and dependent
strict-clippy lanes (`xiuxian-wendao` + `xiuxian-daochang`) returned to fully clean
status.
