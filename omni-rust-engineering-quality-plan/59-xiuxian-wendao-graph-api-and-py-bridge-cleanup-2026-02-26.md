# Xiuxian-Wendao Graph API And Py-Bridge Cleanup (2026-02-26)

## Scope

This shard records a follow-up quality pass on graph core APIs and Python
bridge boundaries in `xiuxian-wendao`.

Targets:

- `packages/rust/crates/xiuxian-wendao/src/graph/errors.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/entity_ops.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph/relation_ops.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph_py/py_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/graph_py/py_graph/skill_methods.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph_py/engine/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph_refs_py/py_functions.rs`
- `packages/rust/crates/xiuxian-wendao/src/storage_py.rs`

## Changes Implemented

### 1) Removed `unnecessary_wraps` by introducing real validation error path

Actions:

- Added `GraphError::InvalidEntity(String)`.
- Updated `KnowledgeGraph::add_entity` to validate required fields and return
  `InvalidEntity` for empty `id`/`name`.
- Removed `#[allow(clippy::unnecessary_wraps)]` from `add_entity`.

### 2) Removed `needless_pass_by_value` in relation core

Actions:

- Changed relation insertion API to borrow:
  - `add_relation(&self, relation: &Relation) -> Result<(), GraphError>`
- Updated all call sites in graph persistence, valkey load path, skill
  registration, and `PyO3` core methods.

### 3) Removed bridge-level suppressions

Actions:

- `PyLinkGraphEngine::narrate_hits_json` converted to `#[staticmethod]`
  (removed `unused_self` suppression).
- `link_graph_find_referencing_notes` now consumes `contents` with
  `into_iter()` (removed `needless_pass_by_value` suppression).
- `PyKnowledgeStorage::vector_search` now consumes input vector through boxed
  slice conversion before search (removed `needless_pass_by_value` suppression).
- `PyKnowledgeGraph::query_tool_relevance` now passes ownership to helper;
  helper normalizes/filters terms before graph query (removed suppression).

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

- `xiuxian-wendao/src` suppression count reduced from `6` to `4`.
- Remaining suppressions are structural and require larger refactors:
  - `struct_excessive_bools` (`SearchArgs`),
  - `struct_field_names` (`Entity`, `Relation`, parser directive state).
