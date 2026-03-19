# 211. Xiuxian-Skills `test_skill_scanner` Warning-Zero Convergence Wave (2026-02-28)

## Scope

This wave continued warning burndown by cleaning
`tests/test_skill_scanner.rs`, previously the largest remaining warning lane.

## What Changed

1. Fixed documentation style warnings (`clippy::doc_markdown`) by adding
   backticks for module/type/API names:
   - `skill_scanner`
   - `SKILL.md`
   - `SkillScanner`
   - `SnifferRule`
   - `build_index_entry`
   - `build_canonical_payload`
   - `for_tools`
   - `skill_tool_references`
2. Removed needless borrows in `fs::write(...)` path arguments.
3. Replaced `"" .to_string()` patterns with `String::new()`.
4. Replaced `Default::default()` with `ToolAnnotations::default()` in
   `ToolRecord` fixtures (`clippy::default_trait_access`).
5. Removed the last unnecessary raw-string hash in frontmatter fixture text.
6. Added explicit `ToolAnnotations` import for clarity and type-local default.

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-skills`
2. `CARGO_TARGET_DIR=target/clippy-xiuxian-skills cargo clippy -p xiuxian-skills --test test_skill_scanner -- -W clippy::too_many_lines`
3. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills -E 'binary(test_skill_scanner)' --no-tests=pass`
4. `CARGO_TARGET_DIR=target/clippy-xiuxian-skills cargo clippy -p xiuxian-skills --all-targets -- -W clippy::too_many_lines`
5. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills`

Outcomes:

- `test_skill_scanner` strict-clippy lane is warning/error free.
- `test_skill_scanner` `nextest` lane passed: `17 passed`, `0 failed`,
  `0 skipped`.
- Full crate `nextest` passed: `189 passed`, `0 failed`, `0 skipped`.
- Full crate strict-clippy remained pass (`exit code 0`).
- Aggregate strict-clippy warning count reduced from `94` to `60` (`-34`).

## Result

`tests/test_skill_scanner.rs` reached warning-zero convergence under strict
clippy, and overall crate warning debt was further reduced with purely
root-cause fixes and no suppression attributes.
