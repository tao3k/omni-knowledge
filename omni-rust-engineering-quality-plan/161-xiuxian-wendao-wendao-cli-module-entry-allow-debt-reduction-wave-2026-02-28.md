# 161. Xiuxian-Wendao Wendao-CLI Module-Entry Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus: `tests/test_wendao_cli` module-entry files

## Why This Wave

`test_wendao_cli` still had multiple lightweight module-entry files with
file-level `#![allow(...)]`. These are low-risk and high-yield targets for
continued suppression burndown.

## Changes Implemented

Removed file-level `#![allow(...)]` from:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/mod.rs`

No behavior logic changes were required in this wave; these files are primarily
module wiring surfaces.

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

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `85`
  - After this wave: `80`
  - Net reduction: `5` files

## Engineering Outcome

- `wendao_cli` module entry surfaces are now suppression-free under strict
  pedantic linting.
- Remaining debt is increasingly concentrated in heavy scenario files where
  root-cause cleanup can be planned in larger slices.

## Next Slice

- Continue with small/medium `test_wendao_cli` leaf tests:
  - `related/related_command_accepts_ppr_flags.rs`
  - `related/related_verbose_includes_diagnostics.rs`
  - `search/directives/search_rejects_legacy_sort_flag.rs`
