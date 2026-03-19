# 264. xiuxian-daochang config_xiuxian Test Remap Elimination and Public Loader Alignment (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove `config_xiuxian` test harness dependency on
    `#[path = "../../src/config/xiuxian.rs"]`,
  - align test access with crate public loaders,
  - keep strict test/lint gates green without suppression.

## Changes

### 1) Promoted xiuxian loader helpers to public config API

Updated:

- `packages/rust/crates/xiuxian-daochang/src/config/xiuxian.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/lib.rs`

Actions:

- changed visibility:
  - `load_xiuxian_config_from_paths(...)` -> `pub`
  - `load_xiuxian_config_from_bases(...)` -> `pub`
- added `config`-level re-exports for both functions.
- added crate-root re-exports for both functions.
- added public docs for both new public functions to remove
  `missing-docs` warnings.

### 2) Migrated config_xiuxian tests to public crate imports

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/config/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/config/xiuxian_overlay_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/config_xiuxian.rs`

Deleted:

- `packages/rust/crates/xiuxian-daochang/tests/config/xiuxian.rs`

Actions:

- switched test imports to:
  - `xiuxian_daochang::load_xiuxian_config_from_bases`
  - `xiuxian_daochang::load_xiuxian_config_from_paths`
- removed legacy wrapper/module shim and path remap usage.
- simplified top harness module tree to focused test files only.

## Validation Evidence

### 1) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test config_xiuxian
```

Result:

- `13 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors for touched files.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/config/xiuxian\\.rs\\\"" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs'
```

Result:

- no matches.

## Outcome

- `config_xiuxian` test lane now validates via public API contract only,
- obsolete shim file removed,
- touched crate remains test-green and lint-green under required gates.
