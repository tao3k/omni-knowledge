# 253. xiuxian-skills Test Include-Indirection Reduction Wave (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-skills`
- Objective:
  - reduce `include!("unit/...")` indirection in package-top test harnesses,
  - keep test behavior unchanged,
  - preserve strict lint policy (no suppression-first fixes).

## Changes

### 1) Converted 11 harness entries from `include!` to standard file modules

Updated harnesses:

- `packages/rust/crates/xiuxian-skills/tests/knowledge_scanner_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_index_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_reference_record_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_sync_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_metadata_tool_record_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_prompt_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_resource_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_annotations_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_category_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_parser_parameters_model_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_skill_command_parser_unit.rs`

Pattern change:

- from:
  - `mod tests { include!("unit/.../tests.rs"); }`
- to:
  - `mod tests;`

### 2) Moved test implementation files to module-local paths under package-top `tests/`

Moved to:

- `packages/rust/crates/xiuxian-skills/tests/knowledge_scanner_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/metadata_index_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/metadata_reference_record_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/metadata_sync_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/tool_record_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/prompt_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/resource_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/skill_command_annotations_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/skill_command_category_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/skill_command_parser_parameters_model_module/tests.rs`
- `packages/rust/crates/xiuxian-skills/tests/skill_command_parser_module/tests.rs`

Also cleaned empty legacy directories under:

- `packages/rust/crates/xiuxian-skills/tests/unit/**`

### 3) Remaining include-based harnesses (explicitly deferred)

Still include-based:

- `packages/rust/crates/xiuxian-skills/tests/skills_scanner_references_unit.rs`
- `packages/rust/crates/xiuxian-skills/tests/skills_tools_scan_virtual_paths_unit.rs`

Reason:

- these harnesses currently include internal `src` modules for crate-private
  testing seams; they require a separate design pass (API seam extraction or
  dedicated test-only adapters) to remove safely.

## Validation Evidence

### 1) Targeted nextest for touched harnesses

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-skills \
  --test knowledge_scanner_unit \
  --test skills_metadata_index_unit \
  --test skills_metadata_reference_record_unit \
  --test skills_metadata_sync_unit \
  --test skills_metadata_tool_record_unit \
  --test skills_prompt_unit \
  --test skills_resource_unit \
  --test skills_skill_command_annotations_unit \
  --test skills_skill_command_category_unit \
  --test skills_skill_command_parser_parameters_model_unit \
  --test skills_skill_command_parser_unit
```

Result:

- `51 passed`, `0 failed`, `0 skipped`

### 2) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines
```

Result:

- success (exit 0)

### 3) Post-wave include audit

```bash
rg -n 'include!\(' packages/rust/crates/xiuxian-skills/tests --glob '*.rs'
```

Result:

- only 2 harnesses remain (the deferred complex lanes listed above)

## Outcome

- `xiuxian-skills` package-top test harnesses now use a more standard module
  layout in most unit lanes.
- Include-indirection was significantly reduced without changing test behavior.
- Deferred lanes are explicitly isolated with a clear next engineering step.
