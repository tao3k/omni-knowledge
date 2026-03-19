# 184. Xiuxian-Wendao Seed-Priors Related-Filter Accuracy Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_seed_and_priors/link_graph_related_filter_seed_accuracy_is_cluster_grounded.rs`

## Why This Wave

This remaining seed-priors scenario file was still suppression-based and had
clear local refactor opportunities to satisfy strict pedantic lints.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `link_graph_related_filter_seed_accuracy_is_cluster_grounded.rs`

2. Root-cause cleanup:
   - replaced repeated fixture `write_file` calls with a compact fixture loop to
     reduce file size and maintainability overhead
   - removed unnecessary string cloning in error mapping:
     `map_err(|e| e.to_string())` -> `map_err(|e| e.clone())`
   - inlined assertion formatting for stem diagnostics messages

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
  - Before this wave: `13`
  - After this wave: `12`
  - Net reduction: `1` file

## Engineering Outcome

- Seed-grounded related-filter cluster-accuracy scenario test is now
  suppression-free.
- Remaining debt is now fully concentrated in large CLI/agentic scenario files.
