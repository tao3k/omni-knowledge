# 247. xiuxian-daochang Managed Runtime + Memory Stream Consumer `src` Path-Mount Elimination (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Lanes:
  - `channels::managed_runtime`
  - `agent::memory_stream_consumer`
- Goal: remove `src`-side test mounts and keep unit coverage via package-top harnesses.

## Code Changes

### Removed `src` mounts

- `packages/rust/crates/xiuxian-daochang/src/channels/managed_runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/mod.rs`

### Added package-top harnesses

- `packages/rust/crates/xiuxian-daochang/tests/channels_managed_runtime_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_stream_consumer_unit.rs`

### Compatibility adjustment

- Updated include-loaded test module doc style:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_stream_consumer.rs`

### Harness design notes

- `channels_managed_runtime_unit` uses a minimal local `agent::Agent` and `config` shim to compile
  utility-focused modules:
  - `turn.rs`
  - `session_partition_persistence.rs`
- `agent_memory_stream_consumer_unit` uses minimal shims for:
  - `config::MemoryConfig`
  - `session::RedisSessionRuntimeSnapshot`
  - `observability::SessionEvent`
  and reuses real `agent/logging/repeated_failure.rs`.

No broad `#[allow(...)]` suppression was introduced.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test channels_managed_runtime_unit --test agent_memory_stream_consumer_unit
```

Result:

- `21 passed`, `0 failed`, `4 skipped` (live Valkey-dependent tests are ignored by design).

### 2) Mandatory clippy for touched crate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- Completed successfully (`exit code 0`).

### 3) Remaining `src` path-mount scan

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | sort
```

Result:

- Remaining matches: `6` (all in `xiuxian-daochang`).

## Delta Summary

- Previous global residual count: `8`
- New global residual count: `6`
- Net reduction in this wave: `2`
