# High-Quality Rust Engineering Scorecard

## Goal

Provide an objective tracking system to verify whether
`omni-dev-fusion` reaches a high-quality Rust engineering standard.

## Scoring Model (100 Points)

| Area | Weight | Pass Rule |
| --- | ---: | --- |
| Workspace lint inheritance | 15 | 100% crates include `[lints] workspace = true` |
| CI lint/test quality gates | 20 | CI enforces `clippy -D warnings` and `nextest` |
| Dependency security | 15 | `cargo-deny` and `cargo-audit` mandatory in CI |
| Runtime output boundary safety | 10 | protocol/runtime library crates deny stdout/stderr prints |
| Reliability test depth | 15 | integration tests cover lifecycle/cleanup/failure edges |
| Module complexity governance | 15 | hotspot split policy active, with measurable reduction |
| Release engineering discipline | 10 | release preflight checks and target validation in place |

## High-Quality Gate Definition

A release can be labeled as high-quality Rust engineering only if:

1. Total score is at least 85/100.
2. No P0 item from `01-gap-matrix-and-priorities.md` is failing.
3. Quality evidence is attached for each scored area.

## Baseline Snapshot (2026-02-18)

| Area | Baseline Signal | Status |
| --- | --- | --- |
| Workspace lint inheritance | 16/21 crates inherit workspace lints | Fail |
| CI lint/test quality gates | no clippy `-D warnings` lane; no nextest lane | Fail |
| Dependency security | no cargo-deny/cargo-audit mandatory lane | Fail |
| Runtime output boundary safety | 0 crate roots deny print stdout/stderr | Fail |
| Reliability test depth | mixed; exists in selected crates, incomplete for all critical paths | Partial |
| Module complexity governance | 9 Rust files >1000 lines, 36 >500 lines | Partial |
| Release engineering discipline | release hardening not yet modernized | Fail |

## Feature Tracking Grid

| Feature Name | Primary Areas | Owner | Target Date | Evidence Links | Status |
| --- | --- | --- | --- | --- | --- |
| Rust Quality Gate Modernization | CI lint/test quality gates | TBD | TBD | `.github/workflows/ci.yaml`, `.github/workflows/checks.yaml`, `justfile`, `assets/nix/modules/rust.nix` | In progress |
| Dependency Security Guardrails | Dependency security | TBD | TBD | TBD | Not started |
| Workspace Lint Completion | Workspace lint inheritance | TBD | TBD | `packages/rust/crates/*/Cargo.toml`, `justfile` (`rust-lint-inheritance-check`) | Completed |
| MCP Client Reliability Hardening | Reliability test depth | TBD | TBD | TBD | Not started |
| Module Complexity Reduction | Module complexity governance | TBD | TBD | TBD | Not started |
| Rust/Python Boundary Clarification | Reliability test depth | TBD | TBD | TBD | Not started |
| Release Engineering Modernization | Release engineering discipline | TBD | TBD | TBD | Not started |
| Crate Documentation Completion | Governance support | TBD | TBD | TBD | Not started |

## Monthly Review Template

Use this template once per month:

1. Recompute metrics from `05-evidence-metrics-snapshot-2026-02-18.md`.
2. Update each scorecard area with new evidence.
3. Recompute total score.
4. Record blockers and next top-2 priorities.

## Progress Notes

- 2026-02-18: Enabled `rust-quality-gate` in CI and checks workflows.
  Current gate = `cargo check` + strict clippy baseline (`omni-types`,
  `omni-events`, `omni-tokenizer`, `omni-window`, `omni-security`,
  `omni-io`, `omni-executor`, `omni-mcp-client`, `omni-ast`) + `cargo nextest`.
- 2026-02-18: Completed workspace lint inheritance coverage.
  Current state = 21/21 crates include `[lints] workspace = true`.
  Added `rust-lint-inheritance-check` to prevent future regressions.
- 2026-02-18: Expanded strict clippy baseline to include `omni-executor`.
  Crate now passes `cargo clippy -p omni-executor -- -D warnings` and
  `cargo test -p omni-executor`.
- 2026-02-18: Expanded strict clippy baseline to include `omni-mcp-client`.
  Crate now passes `cargo clippy -p omni-mcp-client -- -D warnings`.
  Unit and config tests pass; streamable HTTP integration test needs a
  non-sandbox network bind environment.
- 2026-02-18: Completed dedicated `omni-ast` warning cleanup batch.
  Crate now passes `cargo clippy -p omni-ast -- -D warnings`.
  Functional/unit tests pass.
- 2026-02-18: Restored `omni-ast` benchmark stability without relaxing thresholds.
  Fixed invalid benchmark generator syntax and optimized Python tree-sitter path
  (cached function query + early decorator filtering).
  `cargo test -p omni-ast --test test_ast_benchmark` now passes all 7 tests.
- 2026-02-18: Completed dedicated `omni-macros` strict cleanup batch.
  Crate now passes `cargo clippy -p omni-macros -- -D warnings`.
  Replaced panic-style proc-macro parsing with compile-error diagnostics and
  updated doctest/documentation style for strict lint compliance.
- 2026-02-18: Completed dedicated `omni-lance` strict cleanup batch.
  Crate now passes `cargo clippy -p omni-lance -- -D warnings` and
  `cargo test -p omni-lance`.
  Added checked dimension conversion (`usize -> i32`) with explicit
  `ArrowError` handling and tightened API documentation/must-use coverage.
- 2026-02-18: Started `omni-scanner` historical warning reduction (in progress).
  Current reduction: strict clippy debt dropped from 232 to 107 errors.
  Completed slices:
  `src/frontmatter.rs`, `src/knowledge/scanner.rs`, `src/knowledge/types.rs`,
  `src/skills/tools.rs`, `src/skills/skill_command/{parser,category,annotations,mod}.rs`,
  `src/skills/{mod,canonical,metadata,scanner}.rs`.
- 2026-02-18: Completed dedicated `omni-scanner` strict cleanup batch.
  Crate now passes `cargo clippy -p omni-scanner -- -D warnings`.
  This batch reduced scanner strict debt from 232 to 0.
- 2026-02-18: Expanded strict clippy baseline to include `omni-macros`,
  `omni-lance`, and `omni-scanner` in `just rust-clippy`.
  Baseline command now passes with warnings denied for 12 crates.
- 2026-02-18: Started strict cleanup for `omni-vector` as the next
  high-debt crate in the `omni-knowledge` dependency chain.
  Current strict debt reduced from 548 to 531 errors.
- 2026-02-18: Next strict-expansion candidates:
  run a fresh workspace debt scan and select the next crate by warning count.
