# 254. xiuxian-skills Virtual-Path Filter Harness Retirement and Include Reduction (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-skills`
- Objective:
  - retire a redundant include-heavy harness that depended on direct `src`
    file inclusion,
  - keep behavior coverage via existing higher-level tools scan tests,
  - further reduce include indirection in package-top tests.

## Changes

### 1) Removed redundant harness

Deleted:

- `packages/rust/crates/xiuxian-skills/tests/skills_tools_scan_virtual_paths_unit.rs`

Rationale:

- this lane only validated virtual-path skip behavior
  (`__init__.py`/private/non-`.py`), which is already covered by
  `skills_tools_unit` scan-path tests:
  - `test_scan_paths_skips_init`
  - `test_scan_paths_skips_private_files`
  - `test_scan_paths_skips_non_python`

### 2) Removed now-unused legacy unit file

Deleted:

- `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/scan/virtual_paths/tests.rs`

Cleanup:

- removed empty legacy directories under `tests/unit/**` after deletion.

## Validation Evidence

### 1) Regression sweep for affected and adjacent lanes

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-skills \
  --test skills_tools_unit \
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
  --test skills_skill_command_parser_unit \
  --test skills_scanner_references_unit
```

Result:

- `77 passed`, `0 failed`, `0 skipped`

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

- only one include-based harness remains:
  - `packages/rust/crates/xiuxian-skills/tests/skills_scanner_references_unit.rs`

## Outcome

- include-heavy virtual-path filter harness is retired with coverage preserved
  in existing tools scan lane.
- `xiuxian-skills/tests/unit` is reduced to one remaining file tied to the
  single deferred complex harness.
