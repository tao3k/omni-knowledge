# 修仙道场 (Xiuxian Daochang) Memory-Recall Docs + Test Function Split Follow-up Wave (2026-02-27)

## Scope

Continue warning-driven quality convergence in `xiuxian-daochang` with focus on:

1. Public API docs (`missing_docs`) in memory-recall agent operations.
2. Structural reduction of a long test function without lint suppression.
3. `doc_markdown` follow-up cleanup in Omega contract docs.

## Implemented Changes

1. Added public API docs:
   - `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/agent_ops.rs`
   - Documented `inspect_memory_recall_snapshot` behavior and return semantics.
2. Split long promoted-queue test logic into focused helpers:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_stream_consumer.rs`
   - Introduced `PromotedQueueTestContext`, event append/assert/cleanup helpers.
   - Kept behavior intact while reducing `memory_promoted_events_are_queued_once_for_knowledge_ingest`
     complexity and line span.
3. Cleaned Omega terminology docs for markdown lint:
   - `packages/rust/crates/xiuxian-daochang/src/contracts/omega.rs`
   - Replaced plain `ReAct` mentions with backticked `` `ReAct` `` wording.
4. Removed residual suppression attribute in Xiuxian config:
   - `packages/rust/crates/xiuxian-daochang/src/config/xiuxian.rs`
   - Deleted remaining field-level `#[allow(dead_code)]` on `wendao.link_graph`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::doc_markdown
rg -o "clippy::too_many_lines|clippy::too_many_arguments" \
  packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' \
  | sed 's/.*://g' | sort | uniq -c | sort -nr
```

Results:

- `cargo fmt -p xiuxian-daochang`: pass.
- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`: pass (exit `0`), warnings only.
- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::doc_markdown`: pass (exit `0`).
- `omega.rs` no longer emits `clippy::doc_markdown` warnings in this wave.
- No new `dead_code` warning appears for `config/xiuxian.rs` after removing
  the `wendao.link_graph` suppression.
- Marker scan command: empty output (zero marker occurrences in `xiuxian-daochang/tests`).

Additional check:

- Direct `--test memory_stream_consumer` clippy invocation is not valid for this
  package target naming. Verification was done via `--all-targets`, and the
  previous `too_many_lines` warning for
  `memory_promoted_events_are_queued_once_for_knowledge_ingest` no longer appears.

## Outcome

This wave tightened API contract docs and test-structure quality without adding
suppression attributes, while preserving zero marker debt in `xiuxian-daochang/tests`
and advancing pedantic/doc-markdown convergence.
