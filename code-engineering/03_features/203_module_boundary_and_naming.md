# Module Boundary And Naming

:PROPERTIES:
:ID: code-engineering-module-boundary-naming
:PARENT: [[index]]
:TAGS: feature, modularization, naming, rust, architecture
:STATUS: ACTIVE
:END:

## Goal

Module and naming rules should encode intent, ownership, and responsibility
rather than historical accidents.

## Boundary Rules

1. Split by responsibility, not by arbitrary line counts alone.
2. For medium or complex Rust features, prefer a dedicated feature folder.
3. Keep `mod.rs` interface-only.
4. Use `pub(crate)` by default for internal collaboration surfaces.
5. Expose `pub` only when a real package boundary requires it.

## Naming Rules

1. Namespaces should reflect the feature.
2. Avoid hierarchical naming redundancy in child folders, files, types, and
   modules unless repetition resolves a real collision.
3. Prefer concise leaf names inside already-specific parents.
4. Keep orchestrator names stable and leaf names role-specific.

## Examples

- Prefer `graph/query/plan.rs` over `graph/query/query_plan.rs`.
- Prefer `gateway/search/response.rs` over `gateway/search/search_response.rs`.
- Prefer `runtime/config/loader.rs` over `runtime/config/runtime_config_loader.rs`.

## Non-Goals

- Preserving monoliths for familiarity.
- Using compatibility wrappers as a permanent substitute for real structure.
- Exporting entire internal trees to avoid local refactors.

:RELATIONS:
:LINKS: [[01_core/101_engineering_harness_kernel]], [[03_features/202_test_harness_contracts]], [[06_roadmap/401_code_engineering_harness_expansion]]
:END:

---

:FOOTER:
:STANDARDS: v1.0
:END:
