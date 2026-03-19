# 246. xiuxian-daochang LLM Proxy + Redis Message Store `src` Path-Mount Elimination (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Lanes:
  - `gateway::http::llm_proxy`
  - `session::redis_backend::message_store`
- Goal: remove `src`-side `#[path = "...tests/..."]` mounts and keep coverage via package-top
  harnesses under `tests/`.

## Code Changes

### Removed `src` mounts

- `packages/rust/crates/xiuxian-daochang/src/gateway/http/llm_proxy.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/mod.rs`

### Added package-top harnesses

- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_llm_proxy_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/session_redis_backend_unit.rs`

### Structural notes

- `llm_proxy` harness uses local config shim for `load_xiuxian_config` shape so helper tests stay
  focused on resolution logic.
- `redis_backend` harness uses a minimal backend shim to compile and verify
  `message_store::{encode_chat_message_payload, decode_chat_message_payload}` behavior without
  touching networked backend paths.
- Converted `llm_proxy.rs` header from inner doc comments to regular comments to support stable
  include-based harness compilation.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test gateway_http_llm_proxy_unit --test session_redis_backend_unit
```

Result:

- `11 passed`, `0 failed`.

### 2) Mandatory clippy for touched Rust crate

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

- Remaining matches: `8` (all in `xiuxian-daochang`).

## Delta Summary

- Previous global residual count: `10`
- New global residual count: `8`
- Net reduction in this wave: `2`
