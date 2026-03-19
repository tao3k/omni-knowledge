# 193. Xiuxian-Qianhuan Contracts Missing-Docs Suppression Removal Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianhuan`
- Focus:
  - `tests/contracts.rs`

## Why This Wave

`tests/contracts.rs` used file-level `#![allow(missing_docs)]` at the test
entrypoint. This wave removes the suppression and keeps the file compliant by
adding explicit crate-level documentation.

## Changes Implemented

1. Removed file-level suppression:
   - `#![allow(missing_docs)]`

2. Replaced with explicit module documentation:
   - Added `//! Contract test entrypoints for \`xiuxian-qianhuan\`.`

No additional suppressions were introduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-qianhuan
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-qianhuan cargo clippy -p xiuxian-qianhuan --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-qianhuan cargo nextest run -p xiuxian-qianhuan
```

- Result: pass
- Summary: `51 passed`, `0 failed`, `0 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-qianhuan/tests -g '*.rs'`
  - Before this wave: `1` file
  - After this wave: `0` files
  - Net reduction: `1` file

## Engineering Outcome

- `xiuxian-qianhuan/tests` no longer depends on file-level `missing_docs`
  suppression.
- Contract test entrypoint follows documentation-first lint compliance.
