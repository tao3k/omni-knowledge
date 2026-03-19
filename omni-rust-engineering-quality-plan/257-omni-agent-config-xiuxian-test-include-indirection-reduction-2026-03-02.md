# 257. xiuxian-daochang config_xiuxian Test Include-Indirection Reduction (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target: `tests/config_xiuxian.rs`
- Objective:
  - remove `include!("unit/...")` mounts in this harness,
  - normalize to module-local test files under package-top `tests/config/`,
  - validate behavior and touched-crate lint gate.

## Changes

### 1) Harness normalization

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/config_xiuxian.rs`

Change:

- from:
  - `mod tests { include!("unit/config/config_tests.rs"); }`
  - `mod xiuxian_overlay_tests { include!("unit/config/xiuxian_overlay_tests.rs"); }`
- to:
  - `mod tests;`
  - `mod xiuxian_overlay_tests;`

### 2) Test source relocation

Moved:

- `packages/rust/crates/xiuxian-daochang/tests/unit/config/config_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/config/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/config/xiuxian_overlay_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/config/xiuxian_overlay_tests.rs`

Cleanup:

- removed empty `tests/unit/config` directories during empty-dir cleanup.

## Validation Evidence

### 1) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test config_xiuxian
```

Result:

- `12 passed`, `0 failed`, `0 skipped`

### 2) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- success (exit 0)

Note:

- command reports existing warnings in upstream/touched-adjacent modules
  (`xiuxian-llm` docs + `xiuxian-daochang` provider-mode style warnings), but no
  compilation failure and no new harness-structure warnings introduced by this
  change.

## Outcome

- `config_xiuxian` harness no longer depends on `unit/...` include indirection.
- test sources are now module-local under package-top `tests/config/`.
