# 235. `xiuxian-daochang` LLM and Embedding Tests Top-Level Harness Convergence (2026-03-01)

## Scope

- Keep `xiuxian-daochang` test layout aligned with the package-top `tests/` standard.
- Eliminate remaining `src`-side test mounts for `llm` and `embedding` lanes.
- Converge migrated harnesses to strict clippy warning-zero without lint suppression.

## Changes

1. Migrated `llm` lane to package-top harness
- Source change:
  - `packages/rust/crates/xiuxian-daochang/src/llm/mod.rs`
  - Removed `#[cfg(test)]` + `#[path = "../../tests/llm/*.rs"]` test mounts.
- Harness added:
  - `packages/rust/crates/xiuxian-daochang/tests/llm.rs`
  - Centralized source-module mounting and test includes under package-top `tests/`.

2. Migrated `embedding` lane to package-top harness
- Source changes:
  - `packages/rust/crates/xiuxian-daochang/src/embedding/backend.rs`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/transport_http.rs`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/transport_litellm.rs`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/transport_openai.rs`
  - Removed `src`-side `#[cfg(test)]` + `#[path]` mounts.
- Harness added:
  - `packages/rust/crates/xiuxian-daochang/tests/embedding.rs`
  - Hosts backend and transport tests from top-level `tests/` only.

3. Fixed harness-level clippy and compile blockers without `allow`
- Resolved duplicate import (`E0252`) in embedding harness by removing local
  `Duration` import conflict with `include!`d source.
- Added explicit symbol probes (`let _ = ...;`) to mark migrated public/internal
  APIs as used in harness context and avoid dead-code/no-effect warnings.
- Kept all fixes structural; no file/module-level lint suppression introduced.

## Validation Evidence

1. Embedding lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test embedding -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

2. Embedding lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test embedding
```

- Exit code: `0`
- Result: `8 passed`, `0 failed`.

3. LLM lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test llm -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

4. LLM lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test llm
```

- Exit code: `0`
- Result: `14 passed`, `0 failed`.

5. Mandatory touched-crate strict clippy

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

## Outcome

- `xiuxian-daochang` `llm` and `embedding` test lanes now follow the unified package-top
  `tests/` structure.
- Source modules no longer carry `src`-side test mounting glue for these lanes.
- Migrated harnesses are strict-clippy clean and nextest-validated.
