# 387. KnowledgeCategory sink to xiuxian-types and cross-crate alignment (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-types`
  - `packages/rust/crates/xiuxian-skills`
  - `packages/rust/crates/xiuxian-wendao`
- Goal:
  - remove duplicated `KnowledgeCategory` enums across crates,
  - centralize category contract ownership in `xiuxian-types`,
  - preserve existing runtime behavior in scanner/storage/Python bridge paths.

## Implementation

1. Added canonical `KnowledgeCategory` in `xiuxian-types`:
   - file:
     - `packages/rust/crates/xiuxian-types/src/lib.rs`
   - includes:
     - singular/plural serde compatibility (`pattern` + `patterns`, etc.),
     - `Unknown` fallback variant,
     - utility methods `as_str()` and `as_plural_str()`,
     - `FromStr` and `Display` implementations.

2. Registered the shared category type in schema registry:
   - added `"KnowledgeCategory"` support in:
     - `get_schema_json`
     - `get_registered_types`

3. Migrated `xiuxian-skills` to shared category contract:
   - `knowledge/types/mod.rs`: removed local `category` module, now re-exports
     `xiuxian_types::KnowledgeCategory`.
   - deleted:
     - `packages/rust/crates/xiuxian-skills/src/knowledge/types/category.rs`
   - `knowledge/types/entry.rs`: preserved previous missing-category behavior
     (`Unknown`) via explicit serde default function.
   - `knowledge/types/entry.rs` + `knowledge/types/metadata.rs`:
     used `#[schemars(with = ...)]` bridge so `xiuxian-skills` can keep its
     existing `schemars 1.2` contract while consuming `xiuxian-types`.

4. Migrated `xiuxian-wendao` to shared category contract:
   - removed local enum from:
     - `packages/rust/crates/xiuxian-wendao/src/types/entry.rs`
   - re-exported shared type from:
     - `packages/rust/crates/xiuxian-wendao/src/types/mod.rs`
   - updated category mapping in storage to shared canonical helper:
     - `category_to_str(...) -> category.as_plural_str()`
   - updated Python bridge to remain compatible with new enum ownership:
     - `knowledge_py/py_category.rs`
     - `knowledge_py/py_entry.rs`
     - `knowledge_py/py_functions.rs`

5. No broad lint-suppression attributes were added.

## Verification

- Type/syntax:
  - `cargo check -p xiuxian-types -p xiuxian-skills -p xiuxian-wendao`
  - result: pass
- Mandatory lint gates:
  - `cargo clippy -p xiuxian-types -p xiuxian-skills -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass
- Targeted regressions:
  - `cargo nextest run -p xiuxian-skills --test knowledge_scanner_unit --test test_schema_generation`
  - result: `9 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-wendao --test types_unit --test storage_unit --test test_knowledge`
  - result: `20 passed`, `0 failed`

## Outcome

- Knowledge category contracts are now centralized in `xiuxian-types`.
- `xiuxian-skills` and `xiuxian-wendao` no longer carry duplicated category
  enum definitions.
- Existing behavior was preserved:
  - scanner fallback remains `Unknown` for missing/invalid frontmatter
    category,
  - Wendao storage mappings still emit plural category keys for stats/filter.
