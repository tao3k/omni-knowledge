# 272. xiuxian-daochang llm Test Remap Elimination and Zero Convergence (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove the final `#[path = "../../src/llm/..."]` remaps in `tests/llm.rs`,
  - migrate this lane to stable test-support contracts,
  - close remap debt across all `xiuxian-daochang/tests`.

## Changes

### 1) Added stable LLM test API and test-support boundary

Added:

- `packages/rust/crates/xiuxian-daochang/src/llm/test_api.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/llm.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/llm/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`

Actions:

- introduced test-facing wrappers for:
  - backend mode parsing and inference-url base extraction,
  - tools JSON parsing,
  - chat-completion request serialization shape checks,
  - provider-mode resolution with env override injection,
  - litellm compatibility helpers (`anthropic` endpoint/host checks),
  - multimodal converter entrypoint (`chat_message_to_litellm_message`).

### 2) Rewrote llm harness to consume test-support only

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/llm.rs`
- `packages/rust/crates/xiuxian-daochang/tests/llm/backend.rs`
- `packages/rust/crates/xiuxian-daochang/tests/llm/http_request.rs`
- `packages/rust/crates/xiuxian-daochang/tests/llm/provider_mode.rs`
- `packages/rust/crates/xiuxian-daochang/tests/llm/litellm_compat.rs`
- `packages/rust/crates/xiuxian-daochang/tests/llm/converters_multimodal.rs`

Actions:

- removed all direct path remaps to `src/llm/*`,
- imported test-facing API from `xiuxian_daochang::test_support::*`,
- kept existing test semantics and assertions intact.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test llm
```

Result:

- `23 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings in `xiuxian-daochang` from this migration.

Note:

- pre-existing warnings surfaced from `xiuxian-llm` transitive checks remain
  outside this slice scope.

### 3) Structural proof commands

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests/llm.rs
```

Result:

- no matches.

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests
```

Result:

- no matches.

## Outcome

- `llm` harness no longer path-compiles internal source files,
- all `xiuxian-daochang/tests` remap debt converged to zero,
- migration remains green under required test and clippy gates.
