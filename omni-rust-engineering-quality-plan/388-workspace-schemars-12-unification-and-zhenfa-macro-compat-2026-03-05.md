# 388. Workspace `schemars 1.2` unification and `zhenfa_tool` macro compatibility (2026-03-05)

## Scope

- Goal:
  - unify schema dependency policy to workspace-managed `schemars 1.2`,
  - remove direct per-crate drift (`=1.2.0` / `1.0`) for workspace crates,
  - fix migration breakage in macro-generated Zhenfa tool definitions.
- Touched crates:
  - `xiuxian-types`
  - `xiuxian-skills`
  - `xiuxian-mcp`
  - `xiuxian-macros`
  - workspace dependency surface (`Cargo.toml`)

## Implementation

1. Unified workspace dependency version:
   - `Cargo.toml`:
     - `schemars = "0.8.21"` -> `schemars = "1.2.0"`

2. Removed per-crate explicit versions in workspace members:
   - `packages/rust/crates/xiuxian-skills/Cargo.toml`
     - `schemars = "=1.2.0"` -> `schemars = { workspace = true }`
   - `packages/rust/crates/xiuxian-mcp/Cargo.toml`
     - `schemars = "1.0"` -> `schemars = { workspace = true }`

3. Macro-compatibility fix for `schemars 1.2`:
   - `packages/rust/crates/xiuxian-macros/src/zhenfa_tool.rs`
   - Root cause:
     - old code expected `schema_for!` result to expose `.schema` field.
   - Fix:
     - serialize the returned schema object directly:
       - `serde_json::to_value(schema)` (instead of `to_value(schema.schema)`).

4. Type-schema derive compatibility in `xiuxian-types`:
   - `packages/rust/crates/xiuxian-types/src/lib.rs`
   - Added `JsonSchema` derive to internal helper used by
     `#[serde(from = "...", into = "...")]`:
     - `SkillDefinitionHelper`

## Verification

- Compile gates:
  - `cargo check -p xiuxian-types -p xiuxian-skills -p xiuxian-wendao -p xiuxian-qianhuan -p xiuxian-zhenfa -p omni-memory -p xiuxian-mcp`
  - `cargo check -p xiuxian-macros -p xiuxian-zhenfa -p xiuxian-wendao`
  - `cargo check -p xiuxian-daochang`
  - result: pass (note: existing unrelated `missing_docs` warnings in `xiuxian-qianji`)

- Mandatory lint gates:
  - `cargo clippy -p xiuxian-macros -p xiuxian-types -p xiuxian-skills -p xiuxian-zhenfa -p xiuxian-wendao -p xiuxian-mcp -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-types -p xiuxian-skills -p xiuxian-wendao -p xiuxian-qianhuan -p xiuxian-zhenfa -p omni-memory -p xiuxian-mcp -- -W clippy::too_many_lines`
  - result: pass

- Regression lanes:
  - `cargo nextest run -p xiuxian-zhenfa --test test_zhenfa_tool_macro`
    - result: `3 passed`
  - `cargo nextest run -p xiuxian-skills --test test_schema_generation`
    - result: `3 passed`
  - `cargo nextest run -p xiuxian-wendao --features zhenfa-router --test zhenfa_router_rpc --test zhenfa_router_xml_lite_unit`
    - result: `7 passed`

## Outcome

- Workspace policy is now coherent around `schemars 1.2.0` for workspace crates.
- The `xiuxian-wendao` Zhenfa native tool path is compatible with
  `schemars 1.2` again (no `schema.schema` field access).
- Prior runtime error reported during `agent-channel-webhook` compilation is resolved at source.
