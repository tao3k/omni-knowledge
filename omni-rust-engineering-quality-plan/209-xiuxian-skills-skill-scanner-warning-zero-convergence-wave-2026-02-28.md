# 209. Xiuxian-Skills `skill_scanner` Warning-Zero Convergence Wave (2026-02-28)

## Scope

This wave continued post-`tools_scanner` cleanup by targeting the remaining
warning-heavy `skill_scanner` test lane in `xiuxian-skills`.

## What Changed

1. Cleaned module/test documentation for `clippy::doc_markdown`:
   - added backticks for `SkillScanner`, `SKILL.md`,
     `build_index_entry`, and `rules.toml`.
2. Removed `clippy::needless_borrows_for_generic_args` patterns:
   - replaced `fs::write(&path.join(...), ...)` with
     `fs::write(path.join(...), ...)` in all flagged sites.
3. Removed `clippy::items_after_statements` triggers:
   - deleted function-local `use` statements in tests and relied on
     module-level imports.
4. Added `ToolsScanner` to module-level imports where needed by
   `test_build_index_entry_with_sniffer_rules`.
5. Resolved final `clippy::needless_raw_string_hashes` warning in a
   plain markdown fixture literal.

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-skills`
2. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills -E 'binary(skill_scanner)' --no-tests=pass`
3. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills`
4. `CARGO_TARGET_DIR=target/clippy-xiuxian-skills cargo clippy -p xiuxian-skills --test skill_scanner -- -W clippy::too_many_lines`
5. `CARGO_TARGET_DIR=target/clippy-xiuxian-skills cargo clippy -p xiuxian-skills --all-targets -- -W clippy::too_many_lines`

Outcomes:

- `skill_scanner` lane `nextest` passed: `15 passed`, `0 failed`, `0 skipped`.
- Full crate `nextest` passed: `189 passed`, `0 failed`, `0 skipped`.
- `skill_scanner` strict-clippy lane now reports zero warnings/errors.
- Full crate strict-clippy remains pass (`exit code 0`).
- Aggregate strict-clippy warning count reduced from `153` to `132`
  after this wave (`-21`), matching removal of the prior `skill_scanner`
  warning bucket (`20`) plus one related pedantic warning cleanup.

## Result

`xiuxian-skills/tests/skill_scanner.rs` is now warning-clean under strict
clippy policy, and the crate-level warning surface has been further reduced
without introducing any lint suppression.
