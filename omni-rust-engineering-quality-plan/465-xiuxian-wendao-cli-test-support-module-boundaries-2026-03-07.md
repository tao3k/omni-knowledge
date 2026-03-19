# 465. Xiuxian Wendao CLI Test Support Module Boundaries

Date: 2026-03-07

## Scope

This shard records the structural cleanup of the `test_wendao_cli` family so
its module roots are interface-only and child tests no longer depend on ambient
`use super::*;` imports.

## Why This Change Was Needed

The CLI integration suite had accumulated several hidden dependency layers:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/mod.rs`

Those module roots mixed declarations with helper implementations and ambient
imports. Child files inherited `serde_json::Value`, `TempDir`, CLI command
builders, Valkey cleanup helpers, and parsing helpers through `super::*` rather
than declaring their own dependencies.

## What Changed

### 1) Restored CLI module roots to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/mod.rs`

These files now declare child modules only.

### 2) Extracted shared CLI helpers into dedicated support modules

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/support.rs`

These support modules now own:

- CLI process bootstrapping (`wendao_cmd`)
- file-writing helpers used by fixture setup
- Valkey key-prefix cleanup helpers
- agentic command runners and config builders
- CLI JSON parsing for command-output assertions

### 3) Localized imports in every CLI child test file

Updated representative child files:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/ambiguity.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/attachments.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/allows_global_root_after_subcommand.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/hmas_validate_command.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/stats_reports_note_counts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/log_flow.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/planning.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/agentic_run_can_persist_suggestions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/agentic_run_emits_discovery_quality_signals.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/agentic_run_verbose_emits_monitor_dashboard.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/promoted_links_materialize_in_neighbors_and_related.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/promoted_overlay_is_isolated_by_key_prefix.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/promoted_overlay_resolves_mixed_alias_forms.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/provisional_links_are_isolated_before_promotion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/related_command_accepts_ppr_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/related_verbose_includes_diagnostics.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_path_fuzzy_emits_section_context.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_returns_matches.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_strategy_and_path_sort_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_verbose_includes_monitor_summary.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_directives_apply_without_cli_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_limit_directive_overrides_cli_limit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_rejects_legacy_sort_flag.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_semantic_filter_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_temporal_flags_filter_results.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay.rs`

The child tests now import only the support helpers and domain types they
actually use.

### 4) Kept fixture-contract helpers beside their contract lanes

Updated fixture helpers:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/related/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters_fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay_fixture_contract_support.rs`

These fixture modules now import `TempDir`, `Value`, and filesystem utilities
explicitly instead of inheriting them from parent modules.

## Architectural Takeaways

- CLI tests need the same module-boundary discipline as production code.
- `mod.rs` should not act as a command-helper bucket or import bucket.
- Nested CLI suites benefit from layered support modules when those helpers are
  scoped to an execution lane (`agentic`, `overlay`, `cli_commands`) instead of
  a global prelude.
- Fixture-contract helpers can remain local to a lane as long as they declare
  their own imports explicitly.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_wendao_cli --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` completed with zero warnings.
- The focused CLI integration suite passed (`36 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- New support modules:
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/cli_commands/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/execution/support.rs`
  - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/agentic/overlay/support.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/465-xiuxian-wendao-cli-test-support-module-boundaries-2026-03-07.md`
