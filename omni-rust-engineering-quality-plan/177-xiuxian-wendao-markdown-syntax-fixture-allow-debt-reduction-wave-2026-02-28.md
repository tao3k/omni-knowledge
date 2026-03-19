# 177. Xiuxian-Wendao Markdown Syntax Fixture Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_markdown_syntax_algorithm_fixtures.rs`

## Why This Wave

This fixture test file is a focused, low-coupling target for incremental
suppression-debt reduction in the `xiuxian-wendao` test lane.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_markdown_syntax_algorithm_fixtures.rs`

2. Root-cause fixes after removal:
   - replaced five `map_err(|e| e.to_string())` occurrences with
     `map_err(|e| e.clone())` to satisfy `clippy::implicit_clone`.

No lint suppressions were reintroduced.

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
- Note: initial run reported five `implicit_clone` warnings; fixed and rerun to
  clean pass.

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `24`
  - After this wave: `23`
  - Net reduction: `1` file

## Engineering Outcome

- Markdown syntax and graph-algorithm fixture suite now runs suppression-free
  under strict pedantic checks.
- Remaining debt is concentrated in heavier benchmark/topology and CLI agentic
  scenario suites.
