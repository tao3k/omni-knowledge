# 468. Xiuxian Daochang Telegram Runtime Test Support Modularization

Date: 2026-03-08

## Scope

This shard records the structural cleanup of the Telegram runtime integration
suite under `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/`.

## Why This Change Was Needed

The suite root `mod.rs` had grown into a 390-line harness bucket that mixed:

- module declarations,
- channel mocks,
- inbound-message factories,
- polling/webhook transport fixtures,
- agent/job-manager builders,
- runtime dispatch helpers,
- partition-flow assertions.

That layout violated the repository rule that `mod.rs` must stay interface-only.
It also forced child tests to depend on ambient parent-module leakage via
`use super::{...}`.

## What Changed

### Interface Restoration

Reduced `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/mod.rs`
to an interface-only module graph.

### Support Layer Extraction

Replaced the old single-file harness with a dedicated `support/` module tree:

- `support/mod.rs`
- `support/agent.rs`
- `support/channel.rs`
- `support/messages.rs`
- `support/runtime.rs`
- `support/partition.rs`
- `support/transport.rs`

This keeps each helper concern isolated:

- agent/bootstrap helpers live in `agent.rs`;
- shared mock channels live in `channel.rs`;
- inbound/sample update factories live in `messages.rs`;
- runtime dispatch helpers live in `runtime.rs`;
- cross-partition flow assertions live in `partition.rs`;
- polling/webhook transport fixtures live in `transport.rs`.

### Child Test Import Cleanup

Updated the child test modules to import explicit helpers from
`super::support::{...}` instead of depending on parent-module implementation
state.

## Structural Outcomes

Observed after the cleanup:

- `tests/.../mod.rs` is now 24 lines and interface-only.
- The support layer is decomposed into focused files instead of a single mixed
  harness bucket.
- There are no remaining `use super::{...}` parent-import buckets in the
  Telegram runtime test tree.

Support module sizes after the split:

- `support/mod.rs`: 17 lines
- `support/agent.rs`: 34 lines
- `support/channel.rs`: 39 lines
- `support/runtime.rs`: 48 lines
- `support/messages.rs`: 76 lines
- `support/partition.rs`: 89 lines
- `support/transport.rs`: 104 lines

## Validation Evidence

Executed during the cleanup wave:

```bash
cargo check -p xiuxian-daochang --tests
cargo nextest run -p xiuxian-daochang --test channels_telegram_runtime_unit --no-fail-fast
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-daochang --tests` passed before the second-stage
  support-module split.
- `cargo nextest run -p xiuxian-daochang --test channels_telegram_runtime_unit --no-fail-fast`
  passed (`77 passed, 0 skipped`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` completed with
  one pre-existing warning in `packages/rust/crates/xiuxian-daochang/src/main.rs:41`
  for `clippy::large_futures`; the test-structure cleanup introduced no new
  clippy failures.

## Current Blocker

A follow-up `cargo check -p xiuxian-daochang --tests` after the deeper
`support/` submodule split is currently blocked by unrelated compile failures in
user-touched runtime sources:

- `packages/rust/crates/xiuxian-daochang/src/llm/compat/litellm_ocr.rs`
- `packages/rust/crates/xiuxian-daochang/src/model_host/ocr.rs`

Those failures concern missing OCR imports and are outside the Telegram runtime
test tree. This shard intentionally does not modify those files, because they
belong to a separate active workstream.

## Architectural Takeaways

- Moving helpers out of `mod.rs` is necessary but not sufficient; large support
  buckets should be decomposed again by responsibility.
- Integration-test support code should follow the same modularity rules as
  production Rust code.
- Explicit helper imports from `support` produce a more stable test namespace
  than ambient parent-module leakage.
- Validation records must separate structural wins from unrelated compile debt
  elsewhere in a dirty worktree.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/mod.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/agent.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/channel.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/messages.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/partition.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/support/transport.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/jobs_logging.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/partition_modes.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_admin.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_budget.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_control_admin.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_feedback.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_help.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_injection.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_jobs.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_memory.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_partition.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_preemption.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_reset.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_resume_flow.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_slash_acl.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_status.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/session_stop.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/telemetry.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/transport_command_flow.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels/telegram/runtime/tests/webhook_security.rs`
