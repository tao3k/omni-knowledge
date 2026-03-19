# Omni Vector Skill Ops Doc-Markdown Cleanup (2026-02-26)

## Objective

Reduce lint-suppression debt in `xiuxian-vector` skill operation modules by
removing unnecessary `doc_markdown` suppressions and validating strict pedantic
compliance.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/listing.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/registry.rs`

### What changed

1. Removed file-level `#[allow(clippy::doc_markdown)]` in both modules.
2. Preserved behavior and interfaces; this was suppression cleanup with full
   regression verification.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

Skill operation modules now carry lower suppression debt while remaining fully
green under strict pedantic lint and full test gates.
