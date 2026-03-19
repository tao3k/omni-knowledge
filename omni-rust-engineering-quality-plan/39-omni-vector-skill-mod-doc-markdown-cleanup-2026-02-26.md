# Omni Vector Skill Module Doc-Markdown Cleanup (2026-02-26)

## Objective

Reduce lint-suppression debt in `xiuxian-vector` skill module entry by removing
unnecessary `doc_markdown` suppression and keeping strict pedantic checks green.

## Scope

### Changed file

- `packages/rust/crates/xiuxian-vector/src/skill/mod.rs`

### What changed

1. Removed crate-level `#![allow(clippy::doc_markdown)]` from the skill module.
2. Updated module-level docs to use explicit code formatting for terms such as
   `@skill_command`, `SKILL.md`, `SkillScanner`, and `ToolsScanner`.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

The skill module entry now has lower suppression debt and remains fully clean
under strict lint and test gates.
