# 191. Xiuxian-Wendao Related Module Split and Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/related/mod.rs`
  - extracted helpers:
    - `tests/test_wendao_cli/related/diagnostics_assertions.rs`
    - `tests/test_wendao_cli/related/monitor_assertions.rs`

## Why This Wave

`related/mod.rs` carried broad suppression and mixed module wiring with large
assertion helpers, violating the project modularization preference and
`clippy::too_many_lines` constraints when suppression is removed.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_wendao_cli/related/mod.rs`

2. Split assertion responsibilities by domain:
   - diagnostics assertions moved into
     `related/diagnostics_assertions.rs`
   - monitor assertions moved into
     `related/monitor_assertions.rs`

3. Kept `mod.rs` as interface-only:
   - declares child modules
   - re-exports assertion helpers for existing `super::*` call sites

4. Visibility fix:
   - assertion helpers marked `pub(crate)` so re-exports are legal and clippy
     compile path remains clean.

No suppressions were reintroduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `6`
  - After this wave: `5`
  - Net reduction: `1` file

## Engineering Outcome

- `related` test module now follows interface-only `mod.rs` + focused child
  modules.
- Remaining suppression debt is concentrated in five largest CLI/overlay files.
