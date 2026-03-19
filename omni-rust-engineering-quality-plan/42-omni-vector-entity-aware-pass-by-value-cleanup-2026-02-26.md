# Omni Vector Entity-Aware Pass-By-Value Cleanup (2026-02-26)

## Objective

Remove the remaining `clippy::needless_pass_by_value` suppression in
`xiuxian-vector` entity-aware ranking while preserving behavior and test outcomes.

## Scope

### Changed file

- `packages/rust/crates/xiuxian-vector/src/keyword/entity_aware.rs`

### What changed

1. Removed `#[allow(clippy::needless_pass_by_value)]` from
   `apply_entity_boost`.
2. Refactored cached-entity representation from borrowed to owned:
   - `CachedEntity<'a> { original: &'a EntityMatch, ... }`
   - to `CachedEntity { original: EntityMatch, ... }`.
3. Updated entity-cache construction to consume `entities` with `into_iter()`,
   making by-value API ownership explicit and lint-clean.
4. Updated helper signatures to use `&[CachedEntity]` without lifetime wiring.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`
- `rg -n "allow\\(clippy::needless_pass_by_value\\)" packages/rust/crates/xiuxian-vector/src -g '*.rs'`

Result: all checks passed; no `needless_pass_by_value` suppressions remain in
`xiuxian-vector` source modules.

## Outcome

Entity-aware fusion code now follows explicit ownership semantics and reduced
suppression debt, aligned with the project’s high-quality Rust baseline.
