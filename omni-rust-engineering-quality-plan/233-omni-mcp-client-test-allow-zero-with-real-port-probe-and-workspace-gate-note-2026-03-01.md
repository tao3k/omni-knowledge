# 233. `omni-mcp-client` Test `allow`-Zero Convergence with Real-Port Probe and Workspace Gate Note (2026-03-01)

## Scope

- Remove the last known `#[allow(dead_code)]` debt under `packages/rust/crates/*/tests`.
- Keep real-server integration coverage meaningful without suppression-based shortcuts.
- Record current validation blocker for `omni-mcp-client` crate-local execution.

## Changes

1. Removed stale dead-code suppressions in streamable HTTP integration test
- File: `packages/rust/crates/omni-mcp-client/tests/streamable_http_integration.rs`
- Removed:
  - `#[allow(dead_code)]` on `REAL_PORT`
  - `#[allow(dead_code)]` on `port_open`

2. Replaced suppression with active usage in real-server test lane
- File: `packages/rust/crates/omni-mcp-client/tests/streamable_http_integration.rs`
- Added explicit preflight assertion in `test_connect_real_server`:
  - `assert!(port_open(REAL_PORT).await, ...)`
- Result: both previously suppressed symbols are now first-class test inputs.

## Validation Evidence

1. Repository-level test-allow scan

```bash
rg -n "#\\[allow\\(" packages/rust/crates/*/tests --glob '*.rs'
```

- Exit code: `1` (no matches)
- Result: zero `#[allow(...)]` attributes in Rust `tests/` tree.

2. `omni-mcp-client` strict clippy attempt

```bash
cargo clippy --manifest-path packages/rust/crates/omni-mcp-client/Cargo.toml --all-targets -- -W clippy::too_many_lines
```

- Exit code: `101`
- Blocking reason: crate/workspace mismatch during `cargo metadata`
  (`omni-mcp-client` is currently not an active workspace member).

3. `omni-mcp-client` nextest attempt

```bash
cargo nextest run --manifest-path packages/rust/crates/omni-mcp-client/Cargo.toml --all-targets
```

- Exit code: `102` (wrapper), underlying cargo metadata failure `101`.
- Blocking reason: same workspace mismatch as above.

## Outcome

- Rust test-tree suppression debt remains zero after this wave.
- `omni-mcp-client` real-server lane now uses explicit runtime probes rather than
  dead-code exceptions.
- Follow-up needed: either add `omni-mcp-client` back to workspace quality lanes
  or define an isolated crate validation workflow outside workspace metadata.
