# 255. xiuxian-skills References Harness Public-API Rewrite and Include-Zero (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-skills`
- Objective:
  - remove the final include-heavy references harness,
  - replace crate-internal source includes with tests over public APIs,
  - achieve include-zero in `xiuxian-skills/tests`.

## Changes

### 1) Rewrote references harness to public API tests

Updated:

- `packages/rust/crates/xiuxian-skills/tests/skills_scanner_references_unit.rs`

Replaced include-based harness (which pulled `../src/.../references/*.rs`) with
direct integration tests that use:

- `SkillScanner::build_index_entry`
- `SkillScanner::scan_skill`
- `SkillMetadata::with_name`

Coverage retained for:

- reference record assembly from frontmatter (`title`, `for_tools`, keywords),
- scalar/sequence parsing behavior for `metadata.for_tools`,
- unique+sorted `for_skills` derivation from mixed tool lists,
- strict validation failures for missing `type`,
- strict validation failures for `type=persona` without `metadata.role_class`.

### 2) Removed final legacy unit include file

Deleted:

- `packages/rust/crates/xiuxian-skills/tests/unit/skills/scanner/references/tests.rs`

Cleanup:

- removed now-empty `tests/unit/**` directories.

## Validation Evidence

### 1) Regression sweep across affected and adjacent harnesses

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
  --test skills_scanner_references_unit \
  --test skills_skill_command_annotations_unit \
  --test skills_skill_command_category_unit \
  --test skills_skill_command_parser_parameters_model_unit \
  --test skills_skill_command_parser_unit
```

Result:

- `77 passed`, `0 failed`, `0 skipped`

### 2) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines
```

Result:

- success (exit 0)

### 3) Include audit

```bash
rg -n 'include!\(' packages/rust/crates/xiuxian-skills/tests --glob '*.rs'
```

Result:

- zero matches

## Outcome

- `xiuxian-skills/tests` now has no `include!` indirection.
- References behavior is now validated through stable public API contracts
  instead of test-side source embedding.
