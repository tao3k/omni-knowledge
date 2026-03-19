# 389. `omni-mcp-client` physical sink into `xiuxian-mcp` (2026-03-05)

## Scope

- Goal:
  - complete package-boundary convergence by physically removing the legacy
    non-workspace `omni-mcp-client` crate,
  - keep a single authoritative MCP client/runtime package:
    `xiuxian-mcp`.

## Implementation

1. Removed legacy crate files under:
   - `packages/rust/crates/omni-mcp-client/`
   - deleted:
     - `Cargo.toml`
     - `resources/omni.mcp.tool_result.v1.schema.json`
     - `src/client.rs`
     - `src/config.rs`
     - `src/lib.rs`
     - `tests/README.md`
     - `tests/client.rs`
     - `tests/config.rs`
     - `tests/streamable_http_integration.rs`

2. Residual audit:
   - no remaining `omni-mcp-client` / `omni_mcp_client` references in
     code/workspace manifests/lockfile outside historical knowledge records.

## Verification

- Residual reference check:
  - `rg -n "omni-mcp-client|omni_mcp_client" --glob '!assets/knowledge/**' .`
  - result: no matches
  - `rg -n "omni-mcp-client|omni_mcp_client" Cargo.lock`
  - result: no matches

- Compile gates:
  - `cargo check -p xiuxian-mcp -p xiuxian-llm -p xiuxian-daochang -p xiuxian-wendao`
  - result: pass

- Mandatory lint gate (touched Rust crates):
  - `cargo clippy -p xiuxian-mcp -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass

- MCP regression lane:
  - `cargo nextest run -p xiuxian-mcp --test client --test config --test streamable_http_integration --test tool_call --test tool_policy --test tool_schema`
  - result: `17 passed`, `1 skipped`, `0 failed`

## Outcome

- Repository now has one MCP client authority (`xiuxian-mcp`) without a
  duplicate legacy package path.
- Build/test surface is simpler and avoids accidental drift between two
  parallel MCP client crates.
