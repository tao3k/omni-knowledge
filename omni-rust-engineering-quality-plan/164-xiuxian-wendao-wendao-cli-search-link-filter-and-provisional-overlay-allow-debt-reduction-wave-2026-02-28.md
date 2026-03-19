# 164. Xiuxian-Wendao Wendao-CLI Search Link-Filter and Provisional-Overlay Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/search/link_filters.rs`
  - `tests/test_wendao_cli/search/provisional_overlay.rs`

## Why This Wave

After wave `163`, the only remaining suppression debt inside
`test_wendao_cli/search` was concentrated in `link_filters` and
`provisional_overlay`. Closing these two files completes suppression-free
coverage for the full `search` lane.

## Changes Implemented

Removed file-level `#![allow(...)]` from both scope files.

No new suppression was introduced. No behavior changes were required for this
wave.

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

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`
- Run time: `~9.070s`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `69`
  - After this wave: `67`
  - Net reduction: `2` files

## Engineering Outcome

- `test_wendao_cli/search` is now fully suppression-free.
- Remaining suppression markers are outside the `search` lane and can be
  handled by feature-aligned slices (e.g., graph, sync, knowledge, and
  heavyweight integration tests).

## Next Slice

- Continue with small/medium files in:
  - `tests/test_knowledge/*`
  - `tests/test_sync/*`
