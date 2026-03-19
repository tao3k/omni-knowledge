# 267. xiuxian-daochang runtime_agent_factory Test Remap Elimination and Lib-Boundary Alignment (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove `runtime_agent_factory` test dependency on `#[path = "../src/..."]`,
  - align runtime-agent construction logic with library boundary reuse,
  - keep touched crate green under required nextest/clippy gates.

## Changes

### 1) Eliminated `runtime_agent_factory` source remapping in tests

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory.rs`
- `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/inference.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/runtime_agent_factory.rs`

Actions:

- removed direct remaps to `src/runtime_agent_factory/*` from integration tests,
- removed now-obsolete helper shim file:
  - `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/memory.rs`,
- added stable wrappers under `xiuxian_daochang::test_support::*` for:
  - inference URL resolution,
  - embedding backend/base-url resolution,
  - inference origin guard validation,
  - runtime model resolution,
  - runtime memory option resolution.

### 2) Moved runtime_agent_factory ownership to library boundary

Updated:

- `packages/rust/crates/xiuxian-daochang/src/lib.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/inference.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mcp.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/embedding.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/env_overrides.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/session.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/main.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/gateway.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/repl.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/schedule.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/stdio.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/telegram.rs`

Actions:

- registered `runtime_agent_factory` in library module graph and re-exported:
  - `xiuxian_daochang::build_agent`,
- switched runtime_agent_factory internals from `crate::resolve::*` parsing helpers
  to `crate::env_parse::*` helpers,
- updated binary nodes to call `xiuxian_daochang::build_agent` directly,
- removed no-longer-needed duplicated env parse functions from:
  - `packages/rust/crates/xiuxian-daochang/src/resolve.rs`,
- expanded `env_parse` with missing typed parsers used by runtime-agent factory:
  - positive `u32`,
  - positive `f32`,
  - unit-range `f32`.

## Validation Evidence

### 1) Targeted nextest (runtime-agent factory lane)

```bash
cargo nextest run -p xiuxian-daochang --test runtime_agent_factory
```

Result:

- `20 passed`, `0 failed`, `0 skipped`.

### 2) Regression check for touched LLM lane

```bash
cargo nextest run -p xiuxian-daochang --test llm
```

Result:

- `23 passed`, `0 failed`, `0 skipped`.

### 3) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 4) Structural proof command (remap removal in runtime-agent lane)

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory.rs \
  packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/inference.rs
```

Result:

- no matches.

## Outcome

- `runtime_agent_factory` test lane no longer depends on private source remap,
- runtime agent build path is now library-owned and reusable,
- touched crate remains green under required nextest and clippy quality gates.
