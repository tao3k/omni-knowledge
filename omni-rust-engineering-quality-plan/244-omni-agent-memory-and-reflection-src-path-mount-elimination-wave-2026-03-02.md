# 244. xiuxian-daochang Memory/Reflection `src` Path-Mount Elimination Wave (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Objective: continue removing `src`-side `#[cfg(test)] #[path = "...tests/..."]` mounts by
  moving coverage entry to package-top `tests/` harnesses.
- This slice targets:
  - `agent::memory::decay`
  - `agent::memory::recall_credit`
  - `agent::memory_recall`
  - `agent::reflection`

## Code Changes

### Removed `src`-side test mounts

- `packages/rust/crates/xiuxian-daochang/src/agent/memory/decay.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory/recall_credit.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/reflection/mod.rs`

### Added package-top harnesses

- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_decay_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_credit_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_reflection_unit.rs`

### Compatibility adjustments

- Adjusted module-doc style in legacy test leaf files from inner to outer docs to support
  include-based harness loading:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory/decay.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory/recall_credit.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/agent/reflection.rs`

## Validation Evidence

### 1) Residual `src` path-mount scan

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | sort
```

Result:

- Remaining matches: `11`
- Remaining lanes are all in `xiuxian-daochang` and no longer include the four migrated modules above.

### 2) Targeted test execution (mandatory fast regression)

```bash
cargo nextest run -p xiuxian-daochang --test agent_memory_decay_unit --test agent_memory_recall_credit_unit --test agent_memory_recall_unit --test agent_reflection_unit
```

Result:

- `17 passed`, `0 failed`.

### 3) Mandatory clippy command for touched Rust crate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- **Blocked by external compile errors in dependency crate `xiuxian-qianji`** (unrelated to this
  migration slice), including:
  - missing module file under `swarm/engine` (`worker`)
  - unresolved `SwarmEngine` associated methods in `orchestrator.rs`

Status note:

- This slice keeps zero suppression changes (no broad `#[allow(...)]` added).
- Clippy evidence is recorded as blocked by current workspace state, while targeted migrated-lane
  tests are fully green.

## Delta Summary

- Previous global residual count: `15`
- New global residual count: `11`
- Net reduction in this wave: `4`
