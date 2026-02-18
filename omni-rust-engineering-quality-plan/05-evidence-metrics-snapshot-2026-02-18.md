# Evidence Metrics Snapshot (2026-02-18)

This snapshot records key metrics used by the playbook.

## 1. Structural Metrics

| Metric | Value |
| --- | ---: |
| Omni Rust crate dirs (`packages/rust/crates/*`) | 21 |
| Codex Rust crate dirs with `Cargo.toml` | 45 |
| Omni crates with `[lints] workspace = true` | 16 |
| Codex crates with `[lints] workspace = true` | 38 |
| Omni Rust files >500 lines | 36 |
| Codex Rust files >500 lines | 197 |
| Omni Rust files >1000 lines | 9 |
| Codex Rust files >1000 lines | 77 |
| Omni Python files >500 lines | 68 |
| Omni top-level workflow files | 3 |
| Codex top-level workflow files | 18 |

## 2. Selected Hotspots

## Rust Hotspots (Omni)

- `packages/rust/crates/omni-vector/src/skill/ops_impl.rs` (1271)
- `packages/rust/crates/omni-vector/src/search/search_impl.rs` (1143)
- `packages/rust/crates/omni-vector/src/checkpoint/store.rs` (1116)
- `packages/rust/crates/omni-scanner/src/skills/tools.rs` (1223)
- `packages/rust/crates/omni-scanner/src/skills/metadata.rs` (1156)

## Python Hotspots (Omni)

- `packages/python/agent/src/omni/agent/mcp_server/server.py` (1260)
- `packages/python/foundation/src/omni/foundation/bridge/rust_vector.py` (1138)
- `packages/python/core/src/omni/core/router/hybrid_search.py` (1058)
- `packages/python/core/src/omni/core/kernel/engine.py` (861)

## 3. Policy Coverage Deltas

## Omni crates missing `[lints] workspace = true`

- `packages/rust/crates/omni-executor/Cargo.toml`
- `packages/rust/crates/omni-io/Cargo.toml`
- `packages/rust/crates/omni-macros/Cargo.toml`
- `packages/rust/crates/omni-sandbox/Cargo.toml`
- `packages/rust/crates/omni-tui/Cargo.toml`

## Codex-style print boundary lint

- Codex has 7 crate roots with explicit `deny(print_stdout/print_stderr)`.
- Omni currently has 0.

## Security lane coverage

- Codex has dedicated `cargo-deny` and `cargo-audit` workflows.
- Omni currently has no equivalent mandatory lane.

## 4. Reproduction Command Fragments

```bash
# Count crates with lint inheritance
rg -l '^\[lints\]' packages/rust/crates/*/Cargo.toml | wc -l
rg -l '^\[lints\]' .cache/researcher/openai/codex/codex-rs/*/Cargo.toml | wc -l

# Count large Rust files
find packages/rust -name '*.rs' -type f -print0 | xargs -0 wc -l | awk '$1>1000{c++} END{print c+0}'
find .cache/researcher/openai/codex/codex-rs -name '*.rs' -type f -print0 | xargs -0 wc -l | awk '$1>1000{c++} END{print c+0}'

# Count large Python files
find packages/python -name '*.py' -type f -print0 | xargs -0 wc -l | awk '$1>500{c++} END{print c+0}'

# Check workflow volume
ls .github/workflows/*.yaml | wc -l
ls .cache/researcher/openai/codex/.github/workflows | wc -l
```

## 5. Interpretation Note

These numbers do not imply “copy Codex scale.”
They are used to locate governance gaps and prioritize modernization effort.
