# 278. xiuxian-daochang Admission Test Remap Elimination via test_support (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove `agent_admission` source include remap,
  - move this lane to stable `xiuxian_daochang::test_support` APIs,
  - keep behavior and quality gates green.

## Changes

### 1) Added admission test-support adapter

Added:

- `packages/rust/crates/xiuxian-daochang/src/test_support/admission.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/admission.rs`

Actions:

- introduced stable wrapper types and conversions for downstream admission:
  - policy, decision, reject reason, runtime snapshot, metrics snapshot,
    metrics accumulator,
- exposed crate-internal admission module (`pub(crate) mod admission`) to allow
  test-support delegation,
- relaxed `DownstreamAdmissionPolicy::from_lookup` visibility to `pub(crate)`
  so test-support can call production parsing logic directly.

### 2) Rewrote top-level `agent_admission` tests

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/agent_admission.rs`

Deleted:

- `packages/rust/crates/xiuxian-daochang/tests/agent/admission_impl/tests.rs`

Actions:

- removed `include!("../src/agent/admission.rs")` test remap pattern,
- rewired assertions to stable test-support API,
- preserved original policy/metrics behavioral assertions.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_admission --test agent_memory_decay_unit --test agent_memory_recall_credit_unit --test agent_memory_recall_feedback
```

Result:

- `24 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced in touched `xiuxian-daochang` files for this slice.

### 3) Structural remap count snapshot

```bash
rg -n "include!\\(\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/" \
  packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l
```

Result:

- `34` remaining matches (down from `35` before this wave).

## Outcome

- `agent_admission` lane no longer path-compiles internal source files,
- admission policy/metrics tests now run against stable crate-owned boundaries,
- remap debt continues converging with measurable count reduction.
