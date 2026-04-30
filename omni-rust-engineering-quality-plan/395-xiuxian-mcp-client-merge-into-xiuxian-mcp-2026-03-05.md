# 395) Merge Completion: Remove `xiuxian-mcp-client`, Converge on `xiuxian-mcp`

Date: 2026-03-05

## Decision

`xiuxian-mcp-client` is deprecated and no longer maintained as a separate package surface.
All MCP client capability remains in `xiuxian-mcp`.

## Audit Result

`packages/rust/crates/xiuxian-mcp-client` was an empty shell:

- `src/` empty
- `resources/` empty
- `tests/` empty
- no workspace membership and no references

## Action

- Removed `packages/rust/crates/xiuxian-mcp-client` directory.
- Verified no remaining references to `xiuxian-mcp-client`/`xiuxian_mcp_client`.

## Validation Evidence

- `cargo check -p xiuxian-mcp`: PASS
- `cargo clippy -p xiuxian-mcp -- -W clippy::too_many_lines`: PASS
- `cargo nextest run -p xiuxian-mcp`: PASS (17 passed, 1 skipped)

## Outcome

MCP client ownership is now singular and explicit under `xiuxian-mcp`.
