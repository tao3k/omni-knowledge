# 259. xiuxian-daochang tests Local Include-Indirection Zero Convergence (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Objective:
  - eliminate remaining local test include indirection in package-top harnesses
    (patterns such as `include!("agent/...")`, `include!("gateway/...")`,
    `include!("nodes/...")`, `include!("discord_runtime/mod.rs")`,
    `include!("telegram_runtime/mod.rs")`),
  - standardize to module-local `mod ...;` resolution under package-top `tests/`,
  - complete include-indirection zero for non-`src` test sources.

## Changes

### 1) Agent/domain harness normalization

Updated harnesses:

- `packages/rust/crates/xiuxian-daochang/tests/agent_admission.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_decay_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_credit_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_feedback.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_metrics.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_state_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_stream_consumer_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_reflection_unit.rs`

Key pattern:

- from inline include blocks:
  - `mod tests { include!("agent/..."); }`
- to module declarations:
  - `mod tests;`
  - or named module entries (`mod decay_tests;`, `mod recall_credit_tests;`)
    where two harnesses shared the same parent module name.

Test source relocations (representative):

- `tests/agent/admission.rs`
  -> `tests/agent/admission_impl/tests.rs`
- `tests/agent/memory_recall.rs`
  -> `tests/agent/memory_recall/tests.rs`
- `tests/agent/memory_stream_consumer.rs`
  -> `tests/agent/memory_stream_consumer/tests.rs`
- `tests/agent/reflection.rs`
  -> `tests/agent/reflection/tests.rs`
- `tests/agent/memory/decay.rs`
  -> `tests/agent/memory/decay_tests.rs`
- `tests/agent/memory/recall_credit.rs`
  -> `tests/agent/memory/recall_credit_tests.rs`

### 2) Gateway/nodes harness normalization

Updated harnesses:

- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_runtime_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/gateway_http_llm_proxy_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/nodes_warmup.rs`

Relocations:

- `tests/gateway/http/runtime.rs`
  -> `tests/gateway/http/runtime/tests.rs`
- `tests/gateway/http/llm_proxy.rs`
  -> `tests/gateway/http/llm_proxy/tests.rs`
- `tests/nodes/warmup.rs`
  -> `tests/nodes/warmup_impl/tests.rs`

### 3) Discord/Telegram runtime harness convergence

Updated harnesses:

- `packages/rust/crates/xiuxian-daochang/tests/channels_discord_runtime_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_runtime_unit.rs`

Change:

- from include mounts:
  - `include!("discord_runtime/mod.rs")`
  - `include!("telegram_runtime/mod.rs")`
- to:
  - `mod tests;`

Folder migrations:

- `tests/discord_runtime/*`
  -> `tests/channels/discord/runtime/tests/*`
- `tests/telegram_runtime/*`
  -> `tests/channels/telegram/runtime/tests/*`

## Structural Verification

### 1) No `unit` include indirection

```bash
rg -n "include!\\(\"unit/|#\\[path = \"unit/|mod tests \\{\\s*include!" packages/rust/crates/xiuxian-daochang/tests
```

Result:

- no matches.

### 2) No local include indirection in package-top tests

```bash
rg -n "include!\\(\"" packages/rust/crates/xiuxian-daochang/tests | rg -v "\\.\\./src|concat!"
```

Result:

- no matches.

## Validation Evidence

### 1) Agent/domain migration lanes

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test agent_admission --test agent_memory_decay_unit --test agent_memory_recall_credit_unit --test agent_memory_recall_feedback --test agent_memory_recall_metrics --test agent_memory_recall_state_unit --test agent_memory_recall_unit --test agent_memory_stream_consumer_unit --test agent_reflection_unit
```

Result:

- `57 passed`, `0 failed`, `6 skipped`.

### 2) Gateway/nodes migration lanes

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test gateway_http_runtime_unit --test gateway_http_llm_proxy_unit --test nodes_warmup
```

Result:

- `19 passed`, `0 failed`, `0 skipped`.

### 3) Discord/Telegram runtime migration lanes

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test channels_discord_runtime_unit --test channels_telegram_runtime_unit
```

Result:

- `102 passed`, `0 failed`, `0 skipped`.

### 4) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- success (exit 0).

Note:

- command still reports pre-existing warnings outside this structural migration
  scope (notably `xiuxian-llm` doc-markdown and `xiuxian-daochang` provider-mode
  style/dead-code warnings), with no new compile failures introduced.

## Outcome

- `xiuxian-daochang/tests` now has zero local include-indirection for package-top
  harness test sources.
- test source layout fully aligns to module-local files under package-top
  domain directories.
