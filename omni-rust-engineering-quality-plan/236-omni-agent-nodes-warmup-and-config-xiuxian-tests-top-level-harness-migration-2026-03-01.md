# 236. `xiuxian-daochang` Nodes Warmup and Config Xiuxian Tests Top-Level Harness Migration (2026-03-01)

## Scope

- Continue removing `src`-side `#[cfg(test)]` path mounts in `xiuxian-daochang`.
- Migrate `nodes/warmup` and `config/xiuxian` test lanes to package-top `tests/`.
- Keep strict clippy warning-zero without adding lint suppressions.

## Changes

1. Migrated `nodes/warmup` lane to top-level harness
- Source change:
  - `packages/rust/crates/xiuxian-daochang/src/nodes/warmup.rs`
  - Removed `#[cfg(test)] #[path = "../../tests/nodes/warmup.rs"] mod tests;`
- Harness added:
  - `packages/rust/crates/xiuxian-daochang/tests/nodes_warmup.rs`
  - Added minimal local `resolve` helpers required by `warmup` source module.
  - Mounted source via `include!` and mounted tests from `tests/nodes/warmup.rs`.
  - Added symbol probes (`let _ = ...;`) to keep the harness warning-zero.

2. Migrated `config` xiuxian test lane to top-level harness
- Source change:
  - `packages/rust/crates/xiuxian-daochang/src/config/mod.rs`
  - Removed `#[cfg(test)] #[path = "../../tests/unit/config/config_tests.rs"] mod tests;`
- Harness added:
  - `packages/rust/crates/xiuxian-daochang/tests/config_xiuxian.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/config/xiuxian.rs`
  - Kept focus narrow by mounting only the `xiuxian` configuration module required by
    `tests/unit/config/config_tests.rs`.
  - Added a small wrapper function to expose `load_xiuxian_config_from_bases` through the harness
    boundary while preserving source visibility constraints.
  - Added symbol probes to avoid dead-code warnings in the harness context.

## Validation Evidence

1. Warmup lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test nodes_warmup -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

2. Warmup lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test nodes_warmup
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`.

3. Config xiuxian lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test config_xiuxian -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

4. Config xiuxian lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test config_xiuxian
```

- Exit code: `0`
- Result: `2 passed`, `0 failed`.

5. Mandatory touched-crate strict clippy

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

## Outcome

- `nodes/warmup` and `config/xiuxian` lanes now follow the package-top `tests/` execution model.
- Two additional `src`-side path mounts have been removed from `xiuxian-daochang`.
- Migration stays aligned with the no-suppression rule and is backed by strict-clippy/nextest evidence.
