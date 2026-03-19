# 385. xiuxian-types rename and cross-crate type-sink audit (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/omni-types` -> `packages/rust/crates/xiuxian-types`
  - direct dependents updated:
    - `xiuxian-vector`
    - `xiuxian-wendao`
    - `xiuxian-zhixing`
    - `xiuxian-tui`
    - `omni-memory`
    - `xiuxian-daochang`
    - `omni-core-rs` (Python binding crate)
- Goal:
  - complete package rename from `omni-types` to `xiuxian-types`,
  - perform cross-crate audit for additional shared contracts that should sink
    into `xiuxian-types`,
  - execute one real sink migration to prove the pattern.

## Implementation

1. Package rename completed:
   - crate folder renamed:
     - `packages/rust/crates/omni-types` -> `packages/rust/crates/xiuxian-types`
   - crate name renamed in manifest:
     - `name = "xiuxian-types"` in
       `packages/rust/crates/xiuxian-types/Cargo.toml`
   - workspace member path updated in root `Cargo.toml`,
   - all rust dependency entries switched from `omni-types` to
     `xiuxian-types`,
   - all rust imports switched from `omni_types::...` to
     `xiuxian_types::...`.

2. One concrete sink migration completed (shared contract dedup):
   - moved `MemoryGateVerdict` and `MemoryGateDecision` to
     `xiuxian-types`:
     - `packages/rust/crates/xiuxian-types/src/lib.rs`
   - removed duplicate definitions in:
     - `packages/rust/crates/omni-memory/src/gate.rs`
     - `packages/rust/crates/xiuxian-daochang/src/contracts/memory_gate.rs`
   - both crates now consume the same canonical contract type from
     `xiuxian-types`.

3. Schema registry expansion:
   - added `MemoryGateVerdict` and `MemoryGateDecision` to
     `get_schema_json` and `get_registered_types` in
     `packages/rust/crates/xiuxian-types/src/lib.rs`,
   - keeps Python/LLM-facing schema discovery aligned with shared type source.

4. Non-code references aligned:
   - `packages/rust/README.md`
   - `nix/modules/lefthook.nix`
   - `packages/python/core/src/omni/core/context/tools.py`
   - `packages/rust/crates/xiuxian-types/README.md`

## Cross-Crate Type Sink Audit

### Already sunk in this wave

- `MemoryGateVerdict`, `MemoryGateDecision`
  - old duplicates:
    - `packages/rust/crates/omni-memory/src/gate.rs`
    - `packages/rust/crates/xiuxian-daochang/src/contracts/memory_gate.rs`
  - canonical now:
    - `packages/rust/crates/xiuxian-types/src/lib.rs`

### Recommended next sink targets (high value)

1. `ToolSearchResult` / `HybridSearchResult` (vector contract family)
   - duplicate locations:
     - `packages/rust/crates/xiuxian-types/src/lib.rs`
     - `packages/rust/crates/xiuxian-vector/src/skill/mod.rs`
     - `packages/rust/crates/xiuxian-vector/src/keyword/fusion/types.rs`
   - recommendation:
     - define one canonical external contract in `xiuxian-types` (`vector` module),
     - keep `xiuxian-vector` internal ranking structs private when needed,
     - add explicit conversion at API boundary.

2. `KnowledgeCategory` / `KnowledgeEntry` (knowledge metadata family)
   - duplicate locations:
     - `packages/rust/crates/xiuxian-skills/src/knowledge/types/category.rs`
     - `packages/rust/crates/xiuxian-skills/src/knowledge/types/entry.rs`
     - `packages/rust/crates/xiuxian-wendao/src/types/entry.rs`
   - recommendation:
     - extract shared category enum + common metadata header into
       `xiuxian-types::knowledge`,
     - keep crate-specific storage/runtime fields local.

3. `SymbolKind` (code symbol taxonomy)
   - duplicate locations:
     - `packages/rust/crates/omni-tags/src/types.rs`
     - `packages/rust/crates/xiuxian-wendao/src/dependency_indexer/symbols/model.rs`
   - recommendation:
     - introduce a superset `SymbolKind` in `xiuxian-types::code`,
     - map crate-local parser variants to the shared taxonomy.

### Explicitly not recommended to sink (for now)

- `QueryIntent`:
  - `xiuxian-vector` uses ranking-mode enum, `xiuxian-wendao` uses NLP intent struct.
  - Same name, different semantics.
- `ExecutionState`:
  - Runtime orchestration internals in qianji/tui; not a shared contract.
- `ChatMessage`:
  - `xiuxian-daochang` session transport payload differs from `xiuxian-llm` multimodal protocol model.
- `ZhixingConfig`:
  - config domain contract should converge in `xiuxian-config-core`, not `xiuxian-types`.

## Verification

- Rename + dependent compile validation:
  - `cargo check -p xiuxian-types -p xiuxian-vector -p xiuxian-wendao -p xiuxian-zhixing -p omni-memory -p xiuxian-tui -p omni-core-rs`
  - result: pass
- Mandatory touched-crate lint gates:
  - `cargo clippy -p xiuxian-types -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-zhixing -- -W clippy::too_many_lines`
  - `cargo clippy -p omni-memory -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-tui -- -W clippy::too_many_lines`
  - `cargo clippy -p omni-core-rs -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p omni-memory --test test_gate`
  - result: `3 passed`
  - `cargo nextest run -p xiuxian-vector --test test_search --test search_impl_unit`
  - result: `22 passed`
  - `cargo nextest run -p xiuxian-daochang --test contracts --test agent_memory_gate_flow`
  - result: `11 passed`, `1 skipped`

## Outcome

- The rename to `xiuxian-types` is complete at workspace/dependency/import level.
- One real shared-contract sink (`MemoryGate*`) is complete and validated.
- A clear, prioritized next-wave sink backlog is now established.
