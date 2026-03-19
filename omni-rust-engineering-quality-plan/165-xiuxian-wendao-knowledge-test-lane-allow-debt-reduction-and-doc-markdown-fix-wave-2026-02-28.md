# 165. Xiuxian-Wendao Knowledge Test Lane Allow-Debt Reduction and Doc-Markdown Fix Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_knowledge/*.rs`
  - `tests/test_knowledge/mod.rs`

## Why This Wave

After `test_wendao_cli/search` convergence, the `test_knowledge` lane was the
next low-risk, high-yield slice: many short leaf tests still carried file-level
`#![allow(...)]` blocks.

## Changes Implemented

Removed file-level `#![allow(...)]` from all `test_knowledge` leaf tests:

- `knowledge_category_equality.rs`
- `knowledge_category_variants.rs`
- `knowledge_entry_clone.rs`
- `knowledge_entry_creation.rs`
- `knowledge_entry_default_category.rs`
- `knowledge_entry_equality.rs`
- `knowledge_entry_tag_operations.rs`
- `knowledge_entry_with_metadata.rs`
- `knowledge_entry_with_options.rs`
- `knowledge_stats_default.rs`
- `search_query_builder.rs`
- `search_query_creation.rs`
- `search_query_default.rs`

Root-cause fix applied after suppression removal:

- `tests/test_knowledge/mod.rs`: updated doc comment to use backticks
  (`KnowledgeCategory`) to satisfy `clippy::doc_markdown`.

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
- Run time: `~12.135s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `67`
  - After this wave: `54`
  - Net reduction: `13` files

## Engineering Outcome

- Entire `test_knowledge` lane is now suppression-free.
- Remaining suppression debt is increasingly concentrated in larger graph/sync
  and integration-heavy tests, which should be addressed in planned slices.

## Next Slice

- Continue with `tests/test_sync/*` small-to-medium files.
