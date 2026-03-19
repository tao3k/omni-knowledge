# 291. xiuxian-daochang mcp dispatch modularization and too-many-lines convergence (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target file: `src/agent/mcp.rs`
- Goal: remove `clippy::too_many_lines` debt in
  `call_mcp_tool_with_diagnostics` without suppression, while preserving
  native/zhenfa/mcp dispatch behavior and telemetry semantics.

## Implementation

1. Split monolithic dispatch function into focused stages:
   - `call_native_tool_with_diagnostics(...)`
   - `call_zhenfa_tool_with_diagnostics(...)`
   - `call_external_mcp_tool_with_diagnostics(...)`
2. Kept the public dispatcher as orchestration-only:
   - `call_mcp_tool_with_diagnostics(...)` now delegates per stage and returns
     early on first handler match.
3. Extracted shared dispatch logging helpers to remove repeated tracing blocks:
   - `log_tool_dispatch_success(...)`
   - `log_tool_dispatch_error_with_detail(...)`
   - `log_tool_dispatch_error(...)`
4. Closed pedantic API warning by replacing `&Option<T>` with `Option<&T>`
   and cloning only at call boundaries.
5. Preserved fallback semantics:
   - native and zhenfa tool errors stay soft (`ToolCallOutput { is_error: true }`)
   - MCP transport errors still return `Err(...)`
   - MCP response errors still return tool output with `is_error = true`.

## Verification

- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass (`0` warnings)
- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test agent_mcp_startup --test test_native_tools --test zhenfa_tool_bridge --test multiple_mcp --test config_mcp`
  - result: `15 passed`, `0 skipped`, `0 failed`

## Outcome

- `src/agent/mcp.rs` no longer emits `too_many_lines`.
- The previous warning debt at `src/agent/mcp.rs:67` is closed via structural
  decomposition, not lint suppression.
