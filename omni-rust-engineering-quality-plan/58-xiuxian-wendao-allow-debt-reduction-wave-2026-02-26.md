# Xiuxian-Wendao Allow-Debt Reduction Wave (2026-02-26)

## Scope

This shard records a focused suppression-debt reduction wave in
`xiuxian-wendao` after storage and doc-markdown convergence work.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/graph/relation_ops.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph_py/py_graph/core_methods.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/valkey_persistence.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/persistence/save_load.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/skill_registry.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph_py/engine/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph_refs_py/py_functions.rs`
- `packages/rust/crates/xiuxian-wendao/src/storage_py.rs`

## Changes Implemented

### 1) Removed `needless_pass_by_value` in graph relation ops

Actions:

- Changed graph core API:
  - `KnowledgeGraph::add_relation(&self, relation: &Relation) -> Result<..., ...>`
- Updated all internal call sites to pass borrowed relations.

### 2) Removed `unused_self` in link-graph Python engine

Actions:

- Converted `narrate_hits_json` to an explicit static Python method:
  - `#[staticmethod] fn narrate_hits_json(hits_json: &str) -> PyResult<String>`

### 3) Removed `needless_pass_by_value` in Python bridge helpers

Actions:

- `link_graph_find_referencing_notes` now consumes `contents` with
  `into_iter()` instead of borrowing.
- `PyKnowledgeStorage::vector_search` now consumes `query_vector` by converting
  to boxed slice before search.

## Verification Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --lib -- -W clippy::pedantic
cargo test -p xiuxian-wendao --lib
```

## Outcome

- `xiuxian-wendao/src` suppression count reduced from `10` to `6`.
- Remaining suppressions are now concentrated in structural policy areas:
  - `struct_field_names` (3),
  - `struct_excessive_bools` (1),
  - `unnecessary_wraps` (1),
  - `needless_pass_by_value` (1).
