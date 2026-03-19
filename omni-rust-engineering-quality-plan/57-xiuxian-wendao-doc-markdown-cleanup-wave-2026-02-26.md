# Xiuxian-Wendao Doc-Markdown Cleanup Wave (2026-02-26)

## Scope

This shard records a focused `doc_markdown` cleanup wave in `xiuxian-wendao`
after removing module-level suppression attributes.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/link_graph_refs/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/skill_registry.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph_py/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/kg_cache.rs`
- `packages/rust/crates/xiuxian-wendao/src/dep_indexer_py/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph_refs_py/mod.rs`
- Follow-up warning fixes:
  - `packages/rust/crates/xiuxian-wendao/src/graph_py/py_graph/mod.rs`
  - `packages/rust/crates/xiuxian-wendao/src/graph_py/py_query_intent.rs`
  - `packages/rust/crates/xiuxian-wendao/src/graph_py/py_skill_doc.rs`
  - `packages/rust/crates/xiuxian-wendao/src/link_graph_refs/extract.rs`

## Changes Implemented

### 1) Removed module-level `doc_markdown` suppressions

Actions:

- Deleted six `#![allow(clippy::doc_markdown)]` attributes from module roots.

### 2) Fixed root-cause documentation style issues

Actions:

- Added backticks around API/type names and protocol-like tokens
  (for example `KnowledgeGraph`, `PyO3`, `SKILL`, `RELATED_TO`,
  `[[EntityName]]`, function names, tuple field names).
- Normalized several relation-format doc lines to code-style notation.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib
```

Result:

- Library tests passed (`53/53`).

## Outcome

- `xiuxian-wendao` suppression count reduced from `16` to `10` in this wave.
- Remaining suppressions are now concentrated in semantic/interop-heavy areas
  (`needless_pass_by_value`, `struct_field_names`, `unnecessary_wraps`,
  `struct_excessive_bools`, `unused_self`).
