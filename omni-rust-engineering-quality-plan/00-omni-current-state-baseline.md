# Omni Current-State Baseline (Rust + Python)

This baseline is derived from repository evidence captured on 2026-02-18.

## 1. What Is Strong Today

## Workspace And Language Baseline

- Root workspace already uses Rust 2024 and central lint policy in `Cargo.toml`.
- Critical lint guardrails already exist:
  - `unsafe_code = "deny"`
  - `unwrap_used = "deny"`
  - `expect_used = "deny"`
- Python stack has strict runtime test settings in `pyproject.toml` and clear package boundaries (`agent/core/foundation/mcp-server`).

## Testing Investment

- Strong Rust snapshot/data-contract testing in `omni-vector` tests.
- Cross-layer validation exists in CI:
  - Rust checks + contract tests in `.github/workflows/ci.yaml`.
- Python tests are broad across unit/integration/contracts in `packages/python/**/tests`.

## Cross-Language Direction Is Already Present

- Rust bindings are operational and used by Python bridge modules:
  - `packages/rust/bindings/python/src/vector/mod.rs`
  - `packages/python/foundation/src/omni/foundation/bridge/rust_vector.py`
- PRJ directory conventions are clearly documented and implemented:
  - `packages/python/foundation/src/omni/foundation/config/prj.py`
  - `AGENTS.md`.

## 2. Current Risks And Gaps

## Rust Quality-Gate Gaps

- CI currently uses `cargo check` and selected tests, but no explicit `cargo clippy -D warnings` lane.
- No `nextest` lane in workflow or local task entrypoint.
- No `cargo-deny`/`cargo-audit` workflow equivalent.

## Lint Policy Coverage Incomplete

- Crates missing `[lints] workspace = true`:
  - `packages/rust/crates/omni-executor/Cargo.toml`
  - `packages/rust/crates/omni-io/Cargo.toml`
  - `packages/rust/crates/omni-macros/Cargo.toml`
  - `packages/rust/crates/omni-sandbox/Cargo.toml`
  - `packages/rust/crates/omni-tui/Cargo.toml`

## Boundary Safety Not Enforced In Rust Libraries

- No crate-level `#![deny(clippy::print_stdout, clippy::print_stderr)]` found under `packages/rust`.
- This can create protocol or output-channel fragility in future tooling layers.

## Module Complexity Hotspots

- Rust hotspots:
  - `packages/rust/crates/omni-vector/src/skill/ops_impl.rs` (1271 lines)
  - `packages/rust/crates/omni-vector/src/search/search_impl.rs` (1143 lines)
  - `packages/rust/crates/omni-scanner/src/skills/tools.rs` (1223 lines)
  - `packages/rust/crates/omni-scanner/src/skills/metadata.rs` (1156 lines)
- Python hotspots:
  - `packages/python/agent/src/omni/agent/mcp_server/server.py` (1260 lines)
  - `packages/python/foundation/src/omni/foundation/bridge/rust_vector.py` (1138 lines)
  - `packages/python/core/src/omni/core/router/hybrid_search.py` (1058 lines)
  - `packages/python/core/src/omni/core/kernel/engine.py` (861 lines)

These are maintenance and regression risk multipliers.

## Rust/Python Boundary Complexity

- Two MCP server stacks co-exist:
  - `packages/python/mcp-server/src/omni/mcp/server.py`
  - `packages/python/agent/src/omni/agent/mcp_server/server.py`
- Runtime behavior ownership can drift unless contract and responsibility are explicit.

## 3. Quantitative Snapshot

| Metric | Omni | Codex Reference |
| --- | ---: | ---: |
| Rust crate directories with Cargo.toml | 21 | 45 |
| Crates with `[lints] workspace = true` | 16 | 38 |
| Rust files >1000 lines | 9 | 77 |
| Rust files >500 lines | 36 | 197 |
| Crates denying print stdout/stderr at crate root | 0 | 7 |
| Top-level workflow files | 3 | 18 |
| `nextest` usage in CI/just | none | present |
| `cargo-deny`/`cargo-audit` lane | none | present |

Note: this table is for calibration, not for copying Codex scale directly.

## 4. Immediate Interpretation

`omni-dev-fusion` already has good architectural intent and strong functional work.
The main weakness is engineering-system maturity around:

- enforceable quality gates,
- dependency security discipline,
- module complexity control,
- cross-language boundary governance.

These are solvable with incremental feature-based work and without disruptive rewrites.
