# 162. Xiuxian-Wendao Wendao-CLI Related and Directive Leaf Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/related/related_command_accepts_ppr_flags.rs`
  - `tests/test_wendao_cli/related/related_verbose_includes_diagnostics.rs`
  - `tests/test_wendao_cli/search/directives/search_rejects_legacy_sort_flag.rs`

## Why This Wave

Following wave `161`, the next low-risk slice was a set of small CLI leaf tests
with file-level suppression markers. These tests validate externally visible CLI
behavior and are good targets for suppression-free convergence.

## Changes Implemented

Removed file-level `#![allow(...)]` from the three files listed in scope.

No behavior logic changes were required; strict clippy passed after suppression
removal without additional source edits.

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

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`
- Run time: `~62.995s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `80`
  - After this wave: `77`
  - Net reduction: `3` files

## Engineering Outcome

- Additional user-facing CLI contract tests now run suppression-free.
- This keeps momentum on small, provable slices while preserving full crate
  validation stability.

## Next Slice

- Continue with:
  - `tests/test_wendao_cli/search/basic/*`
  - `tests/test_wendao_cli/search/directives/*` (remaining files)
