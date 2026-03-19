# 277. xiuxian-daochang Memory Decay/Recall-Credit/Feedback Test Remap Reduction Wave (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove source include remaps from memory-focused top-level tests,
  - replace with stable `test_support` APIs,
  - keep behavior and quality gates green.

## Changes

### 1) Added stable memory test-support surface

Added:

- `packages/rust/crates/xiuxian-daochang/src/test_support/memory_credit.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/memory_feedback.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_feedback.rs`

Actions:

- introduced stable wrappers for:
  - memory decay decisions (`should_apply_decay`, `sanitize_decay_factor`),
  - recall-credit candidate selection and Q-update application,
  - memory feedback parsing/classification/outcome resolution/plan adjustment,
- exposed crate-local test bridge structs and mappings for recall-credit updates,
- promoted `memory_recall_feedback` internals from `pub(super)` to `pub(crate)` where needed for test-support wiring (no public API surface change).

### 2) Rewrote top-level tests to consume `xiuxian_daochang::test_support`

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_decay_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_credit_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_feedback.rs`

Deleted obsolete include-driven test fragments:

- `packages/rust/crates/xiuxian-daochang/tests/agent/memory/decay_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/memory/recall_credit_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_feedback_impl/tests.rs`

Actions:

- removed `include!("../src/agent/memory/decay.rs")`,
- removed `include!("../src/agent/memory/recall_credit.rs")`,
- removed `include!("../src/agent/memory_recall_feedback.rs")`,
- preserved assertions/behavior by porting tests to stable wrappers.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_memory_decay_unit --test agent_memory_recall_credit_unit --test agent_memory_recall_feedback
```

Result:

- `16 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced in touched `xiuxian-daochang` files for this slice,
- transitive `xiuxian-llm` warnings surfaced separately and are outside this slice boundary.

### 3) Structural remap count snapshot

```bash
rg -n "include!\\(\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/" \
  packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l
```

Result:

- `35` remaining matches (down from `38` at start of this wave).

## Outcome

- three memory-focused test lanes now run against stable crate-owned interfaces,
- `xiuxian-daochang` source-remap debt reduced by three include paths in this wave,
- test and lint gates remain green for touched scope.
