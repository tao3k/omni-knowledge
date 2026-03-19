# 245. xiuxian-daochang `gateway/http/runtime` `src` Path-Mount Elimination (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Lane: `gateway::http::runtime`
- Goal: remove remaining `src`-side test mount and keep coverage from package-top `tests/`.

## Code Changes

### Removed `src` mount

- `packages/rust/crates/xiuxian-daochang/src/gateway/http/runtime.rs`

### Added package-top harness

- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_runtime_unit.rs`

### Compatibility adjustment

- Updated module doc style for include-based harness loading:
  - `packages/rust/crates/xiuxian-daochang/tests/gateway/http/runtime.rs`

## Validation Evidence

### 1) Lane validation

```bash
cargo nextest run -p xiuxian-daochang --test gateway_http_runtime_unit
```

Result:

- `9 passed`, `0 failed`.

### 2) Aggregated migrated-lane validation

```bash
cargo nextest run -p xiuxian-daochang --test agent_memory_decay_unit --test agent_memory_recall_credit_unit --test agent_memory_recall_unit --test agent_reflection_unit --test gateway_http_runtime_unit
```

Result:

- `26 passed`, `0 failed`.

### 3) Remaining `src` path-mount scan

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | sort
```

Result:

- Remaining matches: `10` (all in `xiuxian-daochang`).

### 4) Mandatory clippy command for touched Rust crate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- Completed successfully (`exit code 0`).

## Delta Summary

- Previous global residual count: `11`
- New global residual count: `10`
- Net reduction in this lane: `1`
