# 224. `xiuxian-qianji` Error Enum De-suffix and Final `enum_variant_names` Allow Removal (2026-03-01)

## Scope

- Remove the last `#[allow(clippy::enum_variant_names)]` in
  `xiuxian-qianji/src`.
- Keep API behavior equivalent while making error variant naming modern and
  clippy-compliant.
- Revalidate strict clippy and LLM integration test lanes.

## Changes

1. Error enum normalization
- File: `packages/rust/crates/xiuxian-qianji/src/error.rs`
- Removed:
  - `#[allow(clippy::enum_variant_names)]`
- Renamed `QianjiError` variants:
  - `TopologyError` -> `Topology`
  - `ExecutionError` -> `Execution`
  - `DriftError` -> `Drift`
  - `CapacityError` -> `Capacity`
  - `CheckpointError` -> `Checkpoint`
- Preserved `thiserror` display messages, so external string-level behavior
  remains stable.

2. Cross-crate call-site synchronization inside `xiuxian-qianji`
- Updated all references in source and docs/comments where the old variant
  names were used, including:
  - `src/engine/compiler.rs`
  - `src/bootcamp.rs`
  - `src/scheduler/core.rs`
  - `src/scheduler/checkpoint.rs`
  - `src/manifest.rs`
  - `src/safety/mod.rs`
- No legacy variant reference remains in `packages/rust/crates/xiuxian-qianji`.

## Validation Evidence

1. Strict clippy (default target set)

```bash
cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy (`--all-features`)

```bash
cargo clippy -p xiuxian-qianji --all-targets --all-features -- -W clippy::too_many_lines
```

- Exit code: `0`

3. Targeted LLM integration regression lane

```bash
cargo nextest run -p xiuxian-qianji --features llm --test llm_analyzer --test test_bootcamp_api --test llm_multi_tenancy --test llm_augmented_formal_audit --test test_qianji_master_research --test test_agenda_validation_pipeline
```

- Exit code: `0`
- Result: `22 passed`, `0 failed`

## Outcome

- The final `enum_variant_names` suppression in `xiuxian-qianji/src` is removed.
- Error naming is cleaner and consistent with modern Rust style.
- Strict lint and relevant integration regression lanes remain green.
