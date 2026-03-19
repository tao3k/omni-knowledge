# 修仙道场 (Xiuxian Daochang) Float-Cmp Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::float_cmp` file-level allow markers and replacing direct float equality
assertions with tolerance-based checks.

## Implemented Changes

1. Removed `clippy::float_cmp` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Replaced direct float equality assertions with epsilon/tolerance checks:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory/decay.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_metrics.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/config_and_session.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_discover_cache.rs`
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::float_cmp" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::float_cmp` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::float_cmp` while keeping strict
pedantic and `too_many_lines` lanes clean.
