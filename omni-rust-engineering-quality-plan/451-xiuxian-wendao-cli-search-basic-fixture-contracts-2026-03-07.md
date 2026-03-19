# 451. Xiuxian Wendao CLI Search Basic Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of the `test_wendao_cli/search/basic` lane from
inline notebook setup to fixture-backed `input/expected` contracts.

## Why This Change Was Needed

The `search/basic` CLI lane still built every notebook corpus inline with
`TempDir` and `write_file`, even though the behaviors under test were stable CLI
contracts:

- default search results,
- path-fuzzy section context,
- explicit strategy and path sorting,
- verbose monitor and retrieval-plan reporting.

That structure made the command surface harder to inspect and left the expected
payload shape undocumented outside imperative test code.

## What Changed

### 1) Added a lane-specific fixture support module

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/fixture_contract_support.rs`

This support file handles:

- materializing scenario input trees,
- projecting standard search payloads,
- projecting verbose search payloads,
- normalizing monitor and retrieval-plan assertions into stable contracts.

### 2) Converted the basic search module to use the support layer

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/mod.rs`

The lane now has one explicit support module and four focused scenario tests.

### 3) Migrated the four basic CLI scenarios onto fixture-backed roots

Added scenario trees under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/returns_matches/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/path_fuzzy_section_context/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/strategy_and_path_sort/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/verbose_monitor_summary/`

Each scenario now keeps its notebook input beside one expected contract file.

### 4) Replaced imperative asserts with stable CLI contracts

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_returns_matches.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_path_fuzzy_emits_section_context.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_strategy_and_path_sort_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_verbose_includes_monitor_summary.rs`

The tests now assert the CLI response through fixture-backed projections instead
of local field-by-field assertions.

## Architectural Takeaways

- CLI search behaviors should be treated like any other contract surface: keep
  the notebook input and the expected JSON payload together in fixtures.
- Verbose command output should project unstable observability data into stable
  semantic signals such as phase labels, validated policy flags, and monitor
  presence.
- A lane-specific fixture support file is the right abstraction when several CLI
  tests share the same corpus-materialization and payload-projection needs.
- Once this pattern exists for `search/basic`, the next search lanes can extend
  it without reintroducing inline notebook setup.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_returns_matches.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_path_fuzzy_emits_section_context.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_strategy_and_path_sort_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_verbose_includes_monitor_summary.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/returns_matches/input/notes/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/returns_matches/input/notes/beta.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/returns_matches/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/path_fuzzy_section_context/input/docs/architecture/graph.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/path_fuzzy_section_context/input/docs/misc.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/path_fuzzy_section_context/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/strategy_and_path_sort/input/notes/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/strategy_and_path_sort/input/notes/zeta.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/strategy_and_path_sort/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/verbose_monitor_summary/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/verbose_monitor_summary/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/basic/verbose_monitor_summary/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_wendao_cli --message-format short
cargo nextest run -p xiuxian-wendao --test test_wendao_cli test_wendao_search_returns_matches test_wendao_search_path_fuzzy_emits_section_context test_wendao_search_strategy_and_path_sort_flags test_wendao_search_verbose_includes_monitor_summary --no-fail-fast
cargo clippy -p xiuxian-wendao --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- The targeted `cargo nextest run ...` passed (`4 passed, 32 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/450-xiuxian-wendao-seed-and-priors-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/451-xiuxian-wendao-cli-search-basic-fixture-contracts-2026-03-07.md`
