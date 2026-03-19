# 242 - `xiuxian-skills` `src` Path-Mount Zero Convergence and Harness Shim Stabilization - 2026-03-02

## Scope

This wave removes all remaining `src`-side test path mounts in `xiuxian-skills`
and migrates each lane to package-top integration harnesses while preserving
legacy `tests/unit/**` assertions.

## Structural Changes

### Removed `src` test path-mounts

- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths/mod.rs`

### Added package-top harnesses

- `packages/rust/crates/xiuxian-skills/tests/knowledge_scanner_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_tool_record_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_index_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_reference_record_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_sync_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_prompt_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_resource_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_scanner_references_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_annotations_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_category_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_parser_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_parser_parameters_model_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_tools_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_tools_scan_virtual_paths_unit.rs`

### Harness stabilization notes

- Legacy unit files under `tests/unit/**` were kept intact.
- Harnesses use module shims so `super::*` and `crate::*` imports in existing
  unit files continue to work without rewriting the test bodies.
- No lint suppression attributes were introduced.

## Validation Evidence

### Mandatory strict clippy for touched crate

```bash
cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines
```

Result: success.

### Targeted `nextest` proof

```bash
cargo nextest run -p xiuxian-skills \
  --test knowledge_scanner_unit \
  --test skills_metadata_tool_record_unit \
  --test skills_metadata_index_unit \
  --test skills_metadata_reference_record_unit \
  --test skills_metadata_sync_unit \
  --test skills_prompt_unit \
  --test skills_resource_unit \
  --test skills_scanner_references_unit \
  --test skills_skill_command_annotations_unit \
  --test skills_skill_command_category_unit \
  --test skills_skill_command_parser_unit \
  --test skills_skill_command_parser_parameters_model_unit \
  --test skills_tools_unit \
  --test skills_tools_scan_virtual_paths_unit
```

Result: `80 passed`, `0 failed`.

## Burndown Status

Global `src` path-mount scan:

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | wc -l
```

Current remaining count: `18`.

Remaining crates:

- `xiuxian-daochang`
- `omni-tui` (`src/main.rs`)
- `xiuxian-vector` (`match_util`, `search_impl`)

## Outcome

`xiuxian-skills/src` is now path-mount zero and fully aligned with the package-top
test layout standard, with strict clippy and targeted nextest evidence preserved.
