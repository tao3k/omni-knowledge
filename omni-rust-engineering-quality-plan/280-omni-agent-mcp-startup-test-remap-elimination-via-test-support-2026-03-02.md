# 280. xiuxian-daochang MCP Startup Test Remap Elimination via test_support (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove `agent_mcp_startup` source include remap,
  - migrate to stable test-support boundary,
  - preserve startup-connect configuration behavior.

## Changes

Added:

- `packages/rust/crates/xiuxian-daochang/src/test_support/mcp_startup.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mcp_startup.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_mcp_startup.rs`

Actions:

- exposed `agent::mcp_startup` at crate-internal visibility for test-support
  wiring (`pub(crate) mod mcp_startup`),
- promoted `startup_connect_config` to `pub(crate)` and wrapped it through
  `xiuxian_daochang::test_support::startup_connect_config`,
- rewrote top-level `agent_mcp_startup` tests to call test-support directly,
- removed include-driven harness pattern from this lane.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_mcp_startup --test gateway_http_runtime_unit --test gateway_http_llm_proxy_unit --test agent_admission
```

Result:

- `27 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced by this slice.

### 3) Structural remap count snapshot

```bash
rg -n "include!\\(\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/" \
  packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l
```

Result:

- `31` remaining matches (down from `32` before this wave).

## Outcome

- `agent_mcp_startup` no longer path-compiles source internals,
- MCP startup connect-config behavior remains verified through stable test-support,
- remap debt continues to converge with measured count reduction.
