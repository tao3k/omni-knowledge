# 修仙道场 (Xiuxian Daochang) Small-Test Marker Burndown and Docs Replacement (2026-02-27)

## Scope

Continue historical-marker reduction in `xiuxian-daochang/tests` for
`clippy::too_many_lines` and `clippy::too_many_arguments`, focusing on low-risk
small test modules, and replace one remaining `allow` with explicit docs.

## Implemented Changes

1. Removed stale `clippy::too_many_lines` / `clippy::too_many_arguments` marker
   entries from small `<=100` line test files in
   `packages/rust/crates/xiuxian-daochang/tests`.
2. Removed the last `<=100` line stale marker in:
   - `packages/rust/crates/xiuxian-daochang/tests/gateway/http/llm_proxy.rs`
3. Replaced `#![allow(missing_docs)]` in
   `packages/rust/crates/xiuxian-daochang/tests/gateway/http/llm_proxy.rs` with
   explicit doc comments on each test function.
4. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -o "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' \
  | sed 's/.*://g' | sort | uniq -c | sort -nr
rg -n "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- Remaining marker counts in `xiuxian-daochang/tests`:
  - `too_many_lines`: `89`
  - `too_many_arguments`: `88`
  - total marker entries: `177`
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

This wave reduced historical marker debt in small test files and replaced a
test-module `missing_docs` allow with explicit documentation, while keeping
strict clippy gates green.
