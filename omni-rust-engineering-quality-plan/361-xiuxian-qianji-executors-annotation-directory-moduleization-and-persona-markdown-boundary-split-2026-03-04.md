# 361. xiuxian-qianji executors annotation directory moduleization and persona-markdown boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/executors/annotation.rs`
  - added:
    - `src/executors/annotation/mod.rs`
    - `src/executors/annotation/context.rs`
    - `src/executors/annotation/persona_markdown.rs`
- Goal:
  - replace mixed-responsibility annotation executor file with concern-split
    modules while preserving public `ContextAnnotator` API and runtime behavior.

## Implementation

1. Converted `executors::annotation` to directory module:
   - `mod.rs` is interface-only and re-exports `ContextAnnotator`.
2. Split responsibilities by concern:
   - `context.rs`:
     - `ContextAnnotator` model and mechanism execution path.
     - narrative collection, history merge, metadata shaping.
     - registry/wendao persona resolution orchestration.
   - `persona_markdown.rs`:
     - persona profile parsing from markdown/frontmatter.
     - markdown helper functions (`frontmatter`, heading, bullet sections,
       identifier/name normalization, dedup).
3. API compatibility preserved:
   - external import path remains:
     `xiuxian_qianji::executors::annotation::ContextAnnotator`.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/executors/annotation/mod.rs packages/rust/crates/xiuxian-qianji/src/executors/annotation/context.rs packages/rust/crates/xiuxian-qianji/src/executors/annotation/persona_markdown.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Annotation-focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test executors_annotation --test executors_formal_audit --test test_qianji_qianhuan_binding`
  - result: `8 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Annotation executor now follows interface-only module entry and explicit
  boundary split between runtime mechanism flow and persona-markdown parsing.
- Code navigation and future evolution points are cleaner with behavior intact.
