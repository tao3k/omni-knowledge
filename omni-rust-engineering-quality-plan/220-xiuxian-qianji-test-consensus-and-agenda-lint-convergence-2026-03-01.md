# 220. `xiuxian-qianji` Test Lint Convergence (Consensus + Agenda) (2026-03-01)

## Scope

Converge remaining strict-clippy debt in `xiuxian-qianji` test lanes discovered
during cross-crate revalidation:

- `unwrap_used` in `tests/test_consensus.rs`
- `clippy::too_many_lines` in
  `tests/test_agenda_validation_pipeline.rs` main manifest assertion test

Also fixed one follow-up `missing_docs` warning in
`xiuxian-skills/tests/test_frontmatter.rs`.

## Changes

### 1) `tests/test_consensus.rs`

- Added `must_ok` helper for explicit error handling in tests.
- Added `now_millis` helper to remove repeated time conversion and avoid
  `unwrap`.
- Replaced all async `Result` `unwrap()` calls with `must_ok(..., "...")`.
- Replaced timestamp `duration_since(...).unwrap()` with safe helper.

### 2) `tests/test_agenda_validation_pipeline.rs`

- Extracted manifest parsing and node lookup logic into focused helpers:
  - `parse_agenda_validation_manifest`
  - `node_by_id`
  - `qianhuan_binding`
- Reduced line complexity of
  `agenda_validation_manifest_contains_required_nodes_and_bindings`
  below `clippy::too_many_lines` threshold.
- Addressed helper-lifetime follow-up (`needless_lifetimes`) by eliding
  explicit lifetimes.

### 3) `xiuxian-skills` frontmatter test docs

- Added crate-level doc comment in
  `tests/test_frontmatter.rs` to satisfy strict `missing_docs` lane behavior.

## Verification Evidence

### Strict clippy

```bash
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::too_many_lines
cargo clippy -p xiuxian-skills --all-targets -- -W clippy::too_many_lines
```

Result: both pass.

### Targeted nextest

```bash
cargo nextest run -p xiuxian-qianji test_consensus_majority_logic
cargo nextest run -p xiuxian-qianji agenda_validation_manifest_contains_required_nodes_and_bindings
cargo nextest run -p xiuxian-skills split_frontmatter_returns_yaml_and_body
```

Result: all pass.

### Cross-crate strict revalidation

```bash
cargo clippy \
  -p omni-ast -p omni-events -p omni-executor -p omni-io -p omni-security \
  -p omni-tui -p xiuxian-vector -p xiuxian-memory -p xiuxian-qianji \
  -p xiuxian-skills -p xiuxian-wendao \
  --all-targets -- -W clippy::too_many_lines
```

Result: pass (exit code `0`).

## Outcome

`xiuxian-qianji` test-lane strict-clippy blockers from this wave are removed
without lint suppression, and cross-crate strict clippy validation is green.
