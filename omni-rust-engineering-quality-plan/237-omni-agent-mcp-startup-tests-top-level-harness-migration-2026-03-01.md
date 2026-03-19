# 237. `xiuxian-daochang` MCP Startup Tests Top-Level Harness Migration (2026-03-01)

## Scope

- Remove `src`-side test path mount for `agent/mcp_startup`.
- Keep the lane executable from package-top `tests/`.
- Maintain strict clippy warning-zero with no suppression attributes.

## Changes

1. Removed `src`-side test mount
- File:
  - `packages/rust/crates/xiuxian-daochang/src/agent/mcp_startup.rs`
- Removed:
  - `#[cfg(test)] #[path = "../../tests/agent/mcp_startup.rs"] mod tests;`

2. Added package-top harness
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_mcp_startup.rs`
- Added a dedicated integration harness that mounts:
  - source module: `src/agent/mcp_startup.rs`
  - minimal dependencies via harness-local `config` and `mcp` modules
- Inlined the two startup-connect policy tests in the harness lane to avoid
  module-path/doc-comment include issues while preserving behavior.

3. Removed orphaned old test file after migration
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/mcp_startup.rs`
- Deleted because it is no longer referenced after introducing
  `tests/agent_mcp_startup.rs`.

4. Added explicit symbol probes
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_mcp_startup.rs`
- Added `let _ = ...;` probes for `connect_mcp_pool_if_configured` and
  `startup_connect_config` to keep the harness clippy-clean without `allow`.

## Validation Evidence

1. Migrated target strict clippy

```bash
cargo clippy -p xiuxian-daochang --test agent_mcp_startup -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

2. Migrated target nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_mcp_startup
```

- Exit code: `0`
- Result: `2 passed`, `0 failed`.

3. Combined migrated-lane revalidation

```bash
cargo clippy -p xiuxian-daochang --test embedding --test nodes_warmup --test config_xiuxian --test agent_mcp_startup -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-daochang --test embedding --test nodes_warmup --test config_xiuxian --test agent_mcp_startup
```

- Exit code: `0`
- Result: clippy warning-zero; nextest `15 passed`, `0 failed`.

4. Mandatory touched-crate strict clippy

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

## Outcome

- `agent/mcp_startup` test execution now follows package-top harness structure.
- Remaining `xiuxian-daochang/src` path-mounted test hooks reduced further (from `5` to `4`).
- Migration remains consistent with no-suppression policy and strict verification gates.
