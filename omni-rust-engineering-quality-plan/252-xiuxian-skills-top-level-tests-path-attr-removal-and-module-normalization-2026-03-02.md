# 252. xiuxian-skills Top-Level Tests Path-Attribute Removal and Module Normalization (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-skills`
- Target lane: `skills_tools_unit`
- Objective:
  - remove package-top test entry `#[path = "..."]` indirection,
  - align with standard package-top test module layout,
  - keep behavior unchanged and validate with targeted tests + mandatory clippy.

## Changes

### 1) Removed path-attribute mount from top-level harness

Updated:

- `packages/rust/crates/xiuxian-skills/tests/skills_tools_unit.rs`

Change:

- replaced:
  - `#[path = "unit/skills/tools/tests/mod.rs"]`
  - `mod skills_tools_tests;`
- with:
  - `mod skills_tools_tests;`

This keeps the harness explicit and avoids non-standard file remapping in
package-top tests.

### 2) Normalized test module location under package-top tests

Moved files:

- `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/tests/mod.rs`
  -> `packages/rust/crates/xiuxian-skills/tests/skills_tools_tests/mod.rs`
- `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/tests/parse_content.rs`
  -> `packages/rust/crates/xiuxian-skills/tests/skills_tools_tests/parse_content.rs`
- `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/tests/scan_paths.rs`
  -> `packages/rust/crates/xiuxian-skills/tests/skills_tools_tests/scan_paths.rs`
- `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/tests/scan_scripts.rs`
  -> `packages/rust/crates/xiuxian-skills/tests/skills_tools_tests/scan_scripts.rs`

No test logic was changed; this is a structural normalization only.

Cleanup:

- removed now-empty legacy directory:
  - `packages/rust/crates/xiuxian-skills/tests/unit/skills/tools/tests`

## Validation Evidence

### 1) Targeted test lane

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-skills --test skills_tools_unit
```

Result:

- `21 passed`, `0 failed`, `0 skipped`

### 2) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines
```

Result:

- success (exit 0)

### 3) Package-top tests path-remap audit

```bash
rg -n '#\[path\s*=\s*".*tests/.*"\]' packages/rust/crates/*/tests --glob '*.rs'
```

Result:

- zero matches (no remaining package-top test remaps to `tests/...`)

## Outcome

- `xiuxian-skills` top-level test harness no longer depends on `#[path = "..."]`
  remapping for `skills_tools_unit`.
- Test module layout is now directly rooted under package-top `tests/` for this
  lane, matching project test-structure direction.
