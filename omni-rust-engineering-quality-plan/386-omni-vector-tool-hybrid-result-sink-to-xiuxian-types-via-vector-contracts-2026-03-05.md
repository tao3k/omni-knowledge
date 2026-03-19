# 386. xiuxian-vector tool/hybrid result sink to xiuxian-types via vector contracts (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-types`
  - `packages/rust/crates/xiuxian-vector`
- Goal:
  - eliminate duplicate `ToolSearchResult` and `HybridSearchResult` struct
    definitions in `xiuxian-vector` by sinking canonical vector-internal contracts
    into `xiuxian-types`,
  - preserve `xiuxian-vector` external API paths and avoid broad behavioral churn.

## Implementation

1. Added canonical vector-internal contracts in `xiuxian-types`:
   - `VectorToolSearchResult`
   - `VectorHybridSearchResult`
   - file:
     - `packages/rust/crates/xiuxian-types/src/lib.rs`

2. Registered these types in schema registry:
   - added to `get_schema_json` match arms:
     - `"VectorToolSearchResult"`
     - `"VectorHybridSearchResult"`
   - added to `get_registered_types`.

3. Replaced duplicate structs in `xiuxian-vector` with aliases:
   - `packages/rust/crates/xiuxian-vector/src/skill/mod.rs`
     - `pub type ToolSearchResult = xiuxian_types::VectorToolSearchResult;`
   - `packages/rust/crates/xiuxian-vector/src/keyword/fusion/types.rs`
     - `pub type HybridSearchResult = xiuxian_types::VectorHybridSearchResult;`

4. Compatibility strategy:
   - kept existing `xiuxian-types::ToolSearchResult` /
     `xiuxian-types::HybridSearchResult` payload contracts untouched,
   - introduced dedicated `Vector*` contract names for `xiuxian-vector` internal
     ranking/search pipelines to avoid cross-domain semantic collision.

5. No lint-suppression attributes were introduced.

## Verification

- Type/syntax:
  - `cargo check -p xiuxian-types -p xiuxian-vector`
  - result: pass
- Mandatory lint gates:
  - `cargo clippy -p xiuxian-types -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
  - result: pass
- Regression lanes:
  - `cargo nextest run -p xiuxian-vector --test test_fusion --test test_search --test search_impl_unit --test keyword_entity_aware`
  - result: `43 passed`, `0 failed`

## Outcome

- `xiuxian-vector` no longer owns duplicate `ToolSearchResult` /
  `HybridSearchResult` struct definitions.
- Canonical vector search/fusion contract ownership is now centralized under
  `xiuxian-types`.
- Public `xiuxian-vector` API names remain stable through type aliases.
