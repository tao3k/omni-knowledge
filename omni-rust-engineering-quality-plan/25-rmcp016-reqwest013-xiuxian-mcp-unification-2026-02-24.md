# RMCP 0.16 + Reqwest 0.13 + Xiuxian-MCP Unification (2026-02-24)

## Objective

Advance MCP/runtime dependency modernization while aligning package boundaries:

1. Upgrade workspace MCP stack to `rmcp 0.16`.
2. Upgrade workspace direct HTTP stack to `reqwest 0.13`.
3. Use `xiuxian-mcp` as the active MCP package for `xiuxian-daochang` (single-package direction).

## Scope

- Workspace root dependency definitions (`Cargo.toml`).
- `xiuxian-daochang` MCP dependency wiring.
- New package activation: `packages/rust/crates/xiuxian-mcp`.
- Compile-time route/state compatibility fix in gateway/webhook embedding route merges.

## Implemented Changes

1. Workspace dependency upgrades:
   - `rmcp`: `0.15` -> `0.16`
   - `reqwest`: `0.12` -> `0.13`
   - `reqwest` TLS feature updated to `rustls` for `0.13`.

2. Workspace package membership:
   - Added active member: `packages/rust/crates/xiuxian-mcp`.
   - Removed active member: `packages/rust/crates/omni-mcp-client`.

3. `xiuxian-daochang` MCP dependency switch:
   - `omni-mcp-client` path dependency replaced by `xiuxian-mcp`.
   - MCP import path in pool runtime switched to `xiuxian_mcp`.

4. Embedding route state compatibility fixes (Axum state merge):
   - Introduced generic `embedding_routes<S>() -> Router<S>`.
   - In gateway router: merged as `embedding_routes::<GatewayState>()`.
   - In Telegram webhook builder: merged as
     `embedding_routes::<TelegramWebhookState>()`.
   - Shared embedding runtime is layered via `Extension` for both paths.

5. Schema include path update:
   - `xiuxian-wendao` schema include switched from
     `../../omni-mcp-client/resources/...` to
     `../../xiuxian-mcp/resources/...`.

## Validation Evidence

All commands were executed from repository root on 2026-02-24.

1. MCP package compile:
   - `cargo check -p xiuxian-mcp`
   - Result: pass.

2. Agent compile (default):
   - `cargo check -p xiuxian-daochang`
   - Result: pass.

3. Agent compile (reduced profile):
   - `cargo check -p xiuxian-daochang --no-default-features`
   - Result: pass.

4. Agent test build (default):
   - `cargo test -p xiuxian-daochang --no-run`
   - Result: pass.

5. Agent test build (reduced profile):
   - `cargo test -p xiuxian-daochang --no-run --no-default-features`
   - Result: pass.

6. Schema consumer compile verification:
   - `cargo check -p xiuxian-wendao`
   - Result: pass.

## Notes

- `reqwest 0.11` still exists transitively from upstream dependencies in default
  profile (not introduced by this change); this record only upgrades direct
  workspace HTTP/MCP lanes and package structure.
- `omni-mcp-client` source directory remains in repository as non-workspace
  legacy material; active runtime now resolves through `xiuxian-mcp`.

## Next Slice

1. Decide whether to fully delete or archive `omni-mcp-client` directory after
   downstream references are fully retired.
2. Execute targeted API migration for `jsonschema` (`0.18.3` -> `0.42.x`) in
   Rust Python bindings (`checkpoint.rs`) as a separate, test-backed lane.
3. Continue dependency-graph assertion updates to include `xiuxian-mcp`
   package-boundary invariants.
