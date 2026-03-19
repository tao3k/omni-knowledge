# 205. Package Rename Follow-Up Audit and Dependency-Chain Verification Wave (2026-02-28)

## Scope

This wave follows up the crate rename migration and verifies that the workspace
now consistently uses:

- `xiuxian-config-core`
- `xiuxian-macros`
- `xiuxian-skills`
- `xiuxian-wendao`

## Audit Findings

1. Old crate names are no longer referenced in active Rust/package wiring:
   - `omni-config-core`
   - `omni-macros`
   - `omni-scanner`
2. Remaining old-name occurrences are documentation-only references in this
   knowledge-plan index/history, which are intentional.
3. Workspace membership and package metadata both include the renamed crate
   names.

## Validation Evidence

Commands executed:

1. `rg -n "\\bomni-(config-core|macros|scanner)\\b" Cargo.toml packages/rust packages/python nix scripts .github justfile`
2. `rg -n "packages/rust/crates/(xiuxian-config-core|xiuxian-macros|xiuxian-skills|xiuxian-wendao)" Cargo.toml`
3. `cargo metadata --no-deps --format-version=1 | rg -o "\"name\":\"xiuxian-(config-core|macros|skills|wendao)\"" | sort -u`
4. `cargo check -p xiuxian-config-core -p xiuxian-macros -p xiuxian-skills -p xiuxian-wendao -p xiuxian-qianhuan -p xiuxian-qianji -p xiuxian-zhixing -p xiuxian-daochang -p omni-core-rs`

Outcomes:

- Old-name scan in active code/config/task paths returns no matches.
- `Cargo.toml` workspace members point to renamed crate directories.
- `cargo metadata` lists renamed crate package names.
- Dependency-chain compile check passed for renamed crates and key dependents.

## Result

Rename migration is functionally converged for Rust package wiring and dependent
compile paths. The historical mapping note is preserved as documentation-only
context and does not affect build/test behavior.
