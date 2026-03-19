# 228. `omni-memory` Pybindings Allow Removal and `src` Allow-Zero Milestone (2026-03-01)

## Scope

- Remove the final broad suppression block in
  `packages/rust/crates/omni-memory/src/pymodule_impl.rs`.
- Replace suppression-dependent behavior with explicit documentation and
  structural fixes.
- Revalidate `omni-memory` in both default and `pybindings` lanes.
- Confirm repository-wide `src` allow-zero status for Rust crates.

## Changes

1. Removed broad suppression block in Python bindings module
- File: `packages/rust/crates/omni-memory/src/pymodule_impl.rs`
- Removed module-level allow list:
  - `missing_docs`
  - `clippy::doc_markdown`
  - `clippy::module_inception`
  - `clippy::needless_pass_by_value`
  - `clippy::must_use_candidate`
  - `clippy::semicolon_if_nothing_returned`

2. Structural and documentation fixes in `pymodule_impl`
- Renamed nested module from `pymodule_impl` to `pybindings_impl` and re-exported
  at file end to remove `module_inception`.
- Added doc comments for previously undocumented public fields and public
  factory functions.
- Normalized doc markdown with backticks for type names and argument labels.
- Added `#[must_use]` for pure factory-style `#[pyfunction]` APIs returning new
  wrappers.
- Removed pass-by-value warnings through ownership-aware conversion:
  - consumed vector inputs with `into_boxed_slice()` before forwarding as slices
  - removed redundant clone in `PyEpisodeStore::store` by moving `episode.inner`

3. Follow-up clippy cleanup in test lane
- File: `packages/rust/crates/omni-memory/tests/test_feedback_tracking.rs`
- Replaced direct float equality checks with epsilon-based helper assertions to
  resolve `clippy::float_cmp`.

## Validation Evidence

1. Strict clippy (required touched-crate gate)

```bash
cargo clippy -p omni-memory -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy with Python bindings and all targets

```bash
cargo clippy -p omni-memory --all-targets --features pybindings -- -W clippy::too_many_lines
```

- Exit code: `0`

3. Targeted nextest lane (feedback tracking)

```bash
cargo nextest run -p omni-memory --features pybindings --test test_feedback_tracking
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`

4. Full crate nextest lane (`pybindings`)

```bash
cargo nextest run -p omni-memory --features pybindings
```

- Exit code: `0`
- Result: `69 passed`, `0 failed`

5. Repository-wide source suppression scan

```bash
rg -n "#\\[allow\\(" packages/rust/crates/*/src --glob '*.rs'
```

- Exit code: `1` (no matches)

## Outcome

- `omni-memory` no longer depends on broad pybindings suppression.
- All touched lanes for `omni-memory` are green under strict validation.
- Current milestone: no `#[allow(...)]` attributes remain under
  `packages/rust/crates/*/src`.
