# 189. Xiuxian-Wendao Overlay Key-Prefix Isolation Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/overlay/promoted_overlay_is_isolated_by_key_prefix.rs`
  - shared overlay helper usage in `tests/test_wendao_cli/agentic/overlay/mod.rs`

## Why This Wave

This isolation scenario remained suppression-based and had a large amount of
repeated command boilerplate.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `promoted_overlay_is_isolated_by_key_prefix.rs`

2. Refactored to reuse overlay helpers:
   - `write_agentic_config(...)`
   - `run_wendao_json(...)`
   - `run_wendao_ok(...)`

3. Structural cleanup:
   - collapsed repeated config-writing and command execution sections
   - preserved scenario flow:
     - log suggestion in prefix A
     - promote suggestion in prefix A
     - verify overlay applies in prefix A but not in prefix B
   - reduced file size from `143` lines to `93` lines

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
  - Before this wave: `8`
  - After this wave: `7`
  - Net reduction: `1` file

## Engineering Outcome

- Key-prefix isolation overlay scenario is now suppression-free.
- Remaining debt is concentrated in the largest CLI orchestration files.
