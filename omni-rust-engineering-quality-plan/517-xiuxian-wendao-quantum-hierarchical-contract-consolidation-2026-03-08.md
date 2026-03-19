# 517. Xiuxian Wendao Quantum Hierarchical Contract Consolidation

Date: 2026-03-08

## Scope

This shard records the internal contract consolidation that threads
`HierarchicalHit` through Wendao's quantum-fusion pipeline.

## Why This Change Was Needed

After hierarchical uplink activation, the quantum-fusion path still unpacked the
resolved hierarchy into ad hoc fields and then rebuilt the final `QuantumContext`
from those loose values.

That left duplicated structure in three places:

- `ResolvedQuantumAnchor`,
- `QuantumContextCandidate`,
- `QuantumContext` construction.

This was not a correctness bug, but it was weak engineering: the new
hierarchical contract existed, yet the pipeline still carried duplicated
`anchor_id` / `semantic_path` state outside that contract.

## What Changed

### Resolved Anchor Contract

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`
so `ResolvedQuantumAnchor` now carries:

- the external trimmed `anchor_id` preserved for response compatibility,
- one `HierarchicalHit` value for canonical lineage and owning document data.

This removes the previous split between `seed_doc_id` and `semantic_path`.

### Topology Expansion

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/topology_expansion.rs`
so candidate expansion reads the seed document from
`anchor.hierarchical_hit.doc_id` instead of carrying a separate `seed_doc_id`
field.

### Candidate-to-Context Construction

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scored_context.rs`
so `QuantumContextCandidate` now stores a `HierarchicalHit` instead of a raw
`semantic_path` field.

Updated
`packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
with a crate-private helper:

- `QuantumContext::from_hierarchical_hit(...)`

This makes the final context assembly consume the stabilized hierarchical
contract instead of reconstructing semantic data manually.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast`
  passed (`87 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Once a domain contract exists, internal pipelines should carry that contract
  directly instead of copying its fields into parallel ad hoc structs.
- Compatibility-sensitive fields can stay separate when they serve a different
  purpose than the canonical domain object; here, the trimmed request anchor is
  distinct from the canonical hierarchical hit.
- Small crate-private constructor helpers are preferable to repeated manual
  struct assembly at multiple call sites.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_anchor.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/topology_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/scored_context.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
