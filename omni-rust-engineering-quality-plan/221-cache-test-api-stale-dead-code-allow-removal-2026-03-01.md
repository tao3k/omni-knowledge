# 221. Cache Test API Stale `dead_code` Allow Removal (2026-03-01)

## Scope

Remove stale suppression markers in cache test helper APIs where integration
tests already provide real usage evidence.

Targeted crates:

- `xiuxian-vector`
- `xiuxian-wendao`

## Changes

1. `xiuxian-vector` search cache helper cleanup
- File: `packages/rust/crates/xiuxian-vector/src/search_cache.rs`
- Removed stale `#[allow(dead_code)]` from:
  - `pub fn clear_cache()`
- Rationale: function is used by integration tests in
  `packages/rust/crates/xiuxian-vector/tests/test_search_cache.rs`; suppression was
  no longer justified.

2. `xiuxian-wendao` KG cache helper cleanup
- File: `packages/rust/crates/xiuxian-wendao/src/kg_cache.rs`
- Removed stale `#[allow(dead_code)]` from:
  - `pub fn invalidate_all()`
  - `pub fn cache_len() -> usize`
- Rationale: both helpers are used by integration tests in
  `packages/rust/crates/xiuxian-wendao/tests/test_kg_cache.rs`.

## Validation Evidence

1. Strict clippy (touched crates)

```bash
cargo clippy -p xiuxian-vector -p xiuxian-wendao --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Targeted nextest (`xiuxian-vector`)

```bash
cargo nextest run -p xiuxian-vector --test test_search_cache
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`

3. Targeted nextest (`xiuxian-wendao`)

```bash
cargo nextest run -p xiuxian-wendao --test test_kg_cache
```

- Exit code: `0`
- Result: `4 passed`, `0 failed`

4. Structural guard recheck

```bash
bash scripts/rust/check_test_layout.sh
```

- Exit code: `0`
- Result: passed

## Outcome

- Removed three stale suppression points without behavior changes.
- Preserved strict clippy clean status for touched crates.
- Preserved targeted cache-lane integration behavior under `cargo nextest`.
