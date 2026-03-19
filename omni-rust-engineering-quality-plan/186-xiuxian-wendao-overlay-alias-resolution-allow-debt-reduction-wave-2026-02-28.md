# 186. Xiuxian-Wendao Overlay Alias-Resolution Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/overlay/promoted_overlay_resolves_mixed_alias_forms.rs`
  - shared overlay test helpers in `tests/test_wendao_cli/agentic/overlay/mod.rs`

## Why This Wave

The alias-resolution scenario file still relied on broad suppression and stayed
above the pedantic line-count limit after suppression removal.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `promoted_overlay_resolves_mixed_alias_forms.rs`

2. Added reusable helper utilities in overlay module:
   - `run_wendao_ok(...)`
   - `run_wendao_json(...)`
   These centralize command execution, success assertion, stderr diagnostics,
   and JSON parsing.

3. Refactored test to use helpers:
   - replaced repeated command blocks with helper calls
   - retained test semantics (log suggestion -> decide promoted -> verify
     neighbors overlay)
   - reduced file size from `125` lines to `94` lines

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
  - Before this wave: `11`
  - After this wave: `10`
  - Net reduction: `1` file

## Engineering Outcome

- Mixed-alias promoted-overlay scenario is now suppression-free.
- Remaining suppression debt is concentrated in larger CLI and agentic scenario
  modules.
