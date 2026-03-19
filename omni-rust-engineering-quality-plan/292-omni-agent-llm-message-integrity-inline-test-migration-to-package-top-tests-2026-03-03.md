# 292. xiuxian-daochang llm message-integrity inline-test migration to package-top tests (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target area: `src/llm/client` and package-top `tests/llm`
- Goal: remove inline `#[cfg(test)]` usage from source modules and converge
  message-integrity tests to package-top integration boundaries.

## Implementation

1. Removed inline test module wiring in source:
   - `src/llm/client/mod.rs`
   - deleted `#[cfg(test)] mod tests_message_integrity;`
2. Deleted source-inline test file:
   - `src/llm/client/tests_message_integrity.rs`
3. Added package-top integration test lane:
   - `tests/llm/message_integrity.rs`
   - wired via `tests/llm.rs`
4. Added stable test API boundary for message integrity:
   - `src/llm/test_api.rs`
   - introduced `ToolMessageIntegrityReport`
   - exposed `enforce_tool_message_integrity(...)`
5. Routed integration access through test-support exports:
   - `src/test_support/llm.rs`
   - `src/test_support/mod.rs`
6. Added crate-internal client bridge:
   - `src/llm/client/mod.rs`
   - `enforce_tool_message_integrity_for_tests(...)`
7. Visibility alignment for bridge payload:
   - `src/llm/client/message_integrity.rs`
   - `ToolMessageIntegrityReport` + integrity function moved to `pub(crate)`.

## Verification

- Inline test marker audit:
  - `rg -n "#\[cfg\(test\)\]" packages/rust/crates/xiuxian-daochang/src -g "*.rs" | wc -l`
  - result: `0`
- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test llm`
  - result: `28 passed`, `0 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass

## Outcome

- Message-integrity tests now follow package-top test layout rules.
- Source module `src/llm/client` no longer carries inline test module coupling.
