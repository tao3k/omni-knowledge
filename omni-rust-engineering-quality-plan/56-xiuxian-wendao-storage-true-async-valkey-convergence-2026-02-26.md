# Xiuxian-Wendao Storage True-Async Valkey Convergence (2026-02-26)

## Scope

This shard records root-cause cleanup for `xiuxian-wendao` storage-layer
`unused_async` suppressions by migrating the implementation from blocking Valkey
calls to true async Valkey operations.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/storage/crud.rs`
- `packages/rust/crates/xiuxian-wendao/src/storage/query.rs`
- `packages/rust/crates/xiuxian-wendao/src/storage/keyspace.rs`
- `packages/rust/crates/xiuxian-wendao/Cargo.toml`

## Changes Implemented

### 1) Enabled async Redis runtime support

Actions:

- Updated `redis` dependency in `xiuxian-wendao` to include:
  - `features = ["tokio-comp"]`

### 2) Replaced blocking Valkey operations with async operations

Actions:

- Added async connection helper in storage keyspace:
  - `redis_connection() -> redis::aio::MultiplexedConnection`
- Migrated storage CRUD commands from `query(...)` to
  `query_async(...).await`:
  - `PING`, `HGET`, `HSET`, `HLEN`, `HDEL`, `DEL`
- Migrated entry loading (`HVALS`) to async fetch path.

### 3) Removed suppression attributes by design

Actions:

- Removed `#[allow(clippy::unused_async)]` from storage CRUD/query methods.
- Removed `#[allow(clippy::unused_self)]` in keyspace client helper by making
  it an associated function (`redis_client()`).

Result:

- `storage` module no longer contains `allow(clippy::...)` attributes.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib storage::tests::
```

Additional check:

```bash
rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-wendao/src/storage
```

## Outcome

- Storage async API is now backed by true non-blocking Valkey operations.
- `unused_async` suppression debt in storage layer is fully removed.
- Targeted storage tests remain green (`4/4`).
