# 163. Xiuxian-Wendao Wendao-CLI Search Basic and Directive Lanes Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/search/basic/search_path_fuzzy_emits_section_context.rs`
  - `tests/test_wendao_cli/search/basic/search_returns_matches.rs`
  - `tests/test_wendao_cli/search/basic/search_strategy_and_path_sort_flags.rs`
  - `tests/test_wendao_cli/search/basic/search_verbose_includes_monitor_summary.rs`
  - `tests/test_wendao_cli/search/directives/search_query_directives_apply_without_cli_flags.rs`
  - `tests/test_wendao_cli/search/directives/search_query_limit_directive_overrides_cli_limit.rs`
  - `tests/test_wendao_cli/search/directives/search_semantic_filter_flags.rs`
  - `tests/test_wendao_cli/search/directives/search_temporal_flags_filter_results.rs`

## Why This Wave

After wave `162`, the highest-value low-risk lane was the remaining
`test_wendao_cli/search/basic` and `test_wendao_cli/search/directives` leaf
tests. These files exercise user-visible CLI behavior and were still carrying
file-level suppression blocks.

## Changes Implemented

Removed file-level `#![allow(...)]` from all eight files listed in scope.

No fallback suppression was added. This wave intentionally keeps all checks in
strict mode and fixes are suppression-free.

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
- Run time: `~8.888s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `77`
  - After this wave: `69`
  - Net reduction: `8` files

## Engineering Outcome

- `wendao_cli/search` basic and directive lanes now run suppression-free.
- Remaining suppression debt is now concentrated in heavier integration and
  scenario files, which can be handled in larger root-cause slices.

## Next Slice

- Continue with the remaining `test_wendao_cli/search` leaf files:
  - `tests/test_wendao_cli/search/link_filters.rs`
  - `tests/test_wendao_cli/search/provisional_overlay.rs`
