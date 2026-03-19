# 187. Xiuxian-Wendao Attachments CLI Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/attachments.rs`

## Why This Wave

This CLI attachment scenario suite still used file-level suppression and was
above the pedantic file-length threshold after suppression removal.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_wendao_cli/attachments.rs`

2. Root-cause cleanup:
   - introduced shared local helper `run_attachments_query(...)` for:
     - command execution
     - success assertion with stderr diagnostics
     - JSON payload parsing
   - replaced three duplicated command blocks with helper calls
   - reduced file size from `130` lines to `100` lines

No suppressions were reintroduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `10`
  - After this wave: `9`
  - Net reduction: `1` file

## Engineering Outcome

- Attachment filtering/normalization CLI tests are now suppression-free.
- Remaining debt is concentrated in larger CLI and agentic scenario modules.
