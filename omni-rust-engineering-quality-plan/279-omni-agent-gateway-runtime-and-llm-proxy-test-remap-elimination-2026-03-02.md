# 279. xiuxian-daochang Gateway Runtime and LLM Proxy Test Remap Elimination (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove source include remaps from gateway runtime and LLM-proxy test lanes,
  - migrate tests to stable `test_support` contracts,
  - keep targeted regression and clippy gates green.

## Changes

### 1) Added gateway test-support adapter layer

Added:

- `packages/rust/crates/xiuxian-daochang/src/test_support/gateway_http.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/gateway/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/gateway/http/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/gateway/http/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/gateway/http/llm_proxy.rs`

Actions:

- exposed crate-internal gateway/http modules for test-support access
  (`pub(crate)` visibility),
- promoted helper functions used by tests to `pub(crate)` where needed,
- added stable wrappers for:
  - embedding model/base-url resolution,
  - runtime embedding bootstrap from explicit settings,
  - LLM proxy target URL/env resolution and API-key/model helpers.

### 2) Replaced include-driven gateway tests

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_runtime_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_llm_proxy_unit.rs`

Deleted obsolete include-driven fragments:

- `packages/rust/crates/xiuxian-daochang/tests/gateway/http/runtime/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/gateway/http/llm_proxy/tests.rs`

Actions:

- removed `include!("../src/gateway/http/runtime.rs")`,
- removed `include!("../src/gateway/http/llm_proxy.rs")`,
- preserved assertions and network-stub behavior by porting tests to
  `xiuxian_daochang::test_support`.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test gateway_http_runtime_unit --test gateway_http_llm_proxy_unit
```

Result:

- `17 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced by this slice.

### 3) Structural remap count snapshot

```bash
rg -n "include!\\(\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/" \
  packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l
```

Result:

- `32` remaining matches (down from `34` before this wave).

## Outcome

- gateway runtime/proxy test lanes now rely on stable crate-owned test-support APIs,
- include remap debt reduced by two more paths,
- targeted tests and clippy gate remain green.
