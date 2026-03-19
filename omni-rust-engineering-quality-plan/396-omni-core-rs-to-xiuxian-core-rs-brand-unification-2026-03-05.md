# 396) Rename `omni-core-rs` to `xiuxian-core-rs`

Date: 2026-03-05
Scope: Rust/Python bindings package identity convergence.

## Goal

Unify binding package branding from `omni-*` to `xiuxian-*` for the Rust Python bridge.

## Changes

- Rust crate/package rename:
  - `packages/rust/bindings/python/Cargo.toml`
    - `[package].name`: `omni-core-rs` -> `xiuxian-core-rs`
    - `[lib].name`: `omni_core_rs` -> `xiuxian_core_rs`
- PyO3 module init rename:
  - `packages/rust/bindings/python/src/lib.rs`
    - `fn omni_core_rs(...)` -> `fn xiuxian_core_rs(...)`
- Python packaging metadata rename:
  - `packages/rust/bindings/python/pyproject.toml`
    - project name + maturin module-name migrated to `xiuxian-*`
  - workspace-level `pyproject.toml`, `uv.lock`, and dependent package pyproject references updated.
- Repo-wide usage migration:
  - textual references/imports `omni_core_rs` -> `xiuxian_core_rs`
  - package name references `omni-core-rs` -> `xiuxian-core-rs`
- File/path rename convergence:
  - `nix/packages/omni-core-rs.nix` -> `nix/packages/xiuxian-core-rs.nix`
  - `nix/modules/flake-parts/omni-core-rs.nix` -> `nix/modules/flake-parts/xiuxian-core-rs.nix`
  - `scripts/rust/test_omni_core_rs.sh` -> `scripts/rust/test_xiuxian_core_rs.sh`

## Additional Fixes

- Fixed a failing integration test fixture in
  `packages/rust/bindings/python/tests/test_skill_index.rs`:
  - SKILL frontmatter updated to current nested `metadata` schema contract.

## Validation Evidence

- `cargo pkgid -p xiuxian-core-rs`: PASS
- `cargo pkgid -p omni-core-rs`: expected FAIL (package no longer exists)
- `cargo check -p xiuxian-core-rs`: PASS
- `cargo clippy -p xiuxian-core-rs -- -W clippy::too_many_lines`: PASS
- `cargo nextest run -p xiuxian-core-rs --test test_skill_index`: PASS (10/10)

## Environment Note

- Full `cargo nextest run -p xiuxian-core-rs` fails at test listing stage in current macOS runner with:
  - `dyld: symbol not found in flat namespace '_PyBool_Type'`
- This is runtime dynamic-link behavior for PyO3 test enumeration in this environment, not a compile failure.

