# 249. xiuxian-daochang `src` `#[cfg(test)]` Zero Convergence and Config Test Migration (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Objective:
  - eliminate all `#[cfg(test)]` entries under `src/**`,
  - move remaining `src`-bound config test module to package-top `tests/`,
  - keep validation evidence aligned with repository gate policy.

## Structural Changes

1. Removed `#[cfg(test)]` gates across `xiuxian-daochang/src` and replaced test access with:

- direct module-path imports from package-top harnesses,
- harness-side wrappers where needed,
- production visibility that matches real call boundaries (without global lint suppression).

2. Migrated config test module from source tree:

- moved `packages/rust/crates/xiuxian-daochang/src/config/tests_xiuxian.rs`
  to `packages/rust/crates/xiuxian-daochang/tests/unit/config/xiuxian_overlay_tests.rs`,
- removed `mod tests_xiuxian;` from `packages/rust/crates/xiuxian-daochang/src/config/mod.rs`,
- wired the migrated tests into `packages/rust/crates/xiuxian-daochang/tests/config_xiuxian.rs`.

3. Removed source-side test-only helper artifacts that became dead after top-level harness migration, and updated test modules to consume internal APIs via explicit paths.

## Evidence Commands

### 1) Source `#[cfg(test)]` audit

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[cfg\(test\)\]' | sort
```

Result:

- no matches

### 2) Targeted nextest validation

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang \
  --test agent_admission \
  --test agent_memory_stream_consumer_unit \
  --test channels_telegram_runtime_unit \
  --test config_xiuxian
```

Result:

- `108 passed`, `0 failed`, `4 skipped`

Additional touched-lane validation:

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang \
  --test channels_discord_runtime_unit \
  --test llm \
  --test embedding \
  --test agent_reflection_unit \
  --test agent_memory_recall_unit \
  --test agent_memory_recall_state_unit
```

Result:

- `65 passed`, `0 failed`, `2 skipped`

### 3) Mandatory clippy gate for touched Rust crate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- success (exit 0)

## Notes

- This wave enforces the project direction of keeping tests in package-top `tests/` instead of source-local test gating patterns.
- Existing warnings in large include-driven test harness targets were pre-existing and non-blocking for this gate sequence; no `#[allow(...)]` suppression was introduced.
