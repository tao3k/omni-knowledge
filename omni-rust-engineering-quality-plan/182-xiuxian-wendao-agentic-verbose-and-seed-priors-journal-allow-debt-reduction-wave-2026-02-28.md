# 182. Xiuxian-Wendao Agentic-Verbose and Seed-Priors Journal Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/execution/agentic_run_verbose_emits_monitor_dashboard.rs`
  - `tests/test_link_graph_seed_and_priors/link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`

## Why This Wave

These files were medium/small scenario leaves where suppression removal can be
completed without changing production code behavior.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `agentic_run_verbose_emits_monitor_dashboard.rs`
   - `link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`

2. Root-cause cleanup:
   - `agentic_run_verbose_emits_monitor_dashboard.rs`
     - inlined assertion message formatting by binding `stderr` and using
       `{stderr}`.
   - `link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`
     - removed unnecessary raw-string hashes in fixture markdown literals
     - replaced `map_err(|e| e.to_string())` with `map_err(|e| e.clone())`
     - inlined assertion formatting (`{best_section:?}`, `{stems:?}`)

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
  - Before this wave: `16`
  - After this wave: `14`
  - Net reduction: `2` files

## Engineering Outcome

- Agentic verbose execution telemetry and seed-priors journal-linked scenario
  tests are now suppression-free.
- Remaining debt is concentrated in large CLI modules and two seed-priors
  scenario leaves.
