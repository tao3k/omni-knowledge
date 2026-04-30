# Third-Pass Codex To Omni Adoption Plan (2026-02-23)

## Objective

Translate third-pass Codex Rust engineering patterns into project-specific
execution slices for `omni-dev-fusion`.

This file is Omni-specific and complements:

- `07-second-pass-plan-refinement-2026-02-22.md`
- `12-rust-quality-gates-checklist-2026-02-23.md`

## Current Integration Focus

Primary Rust/Python boundary evidence:

- `packages/rust/bindings/python/src/vector/mod.rs`
- `packages/python/foundation/src/omni/foundation/bridge/rust_vector.py`

Hotspot and baseline evidence:

- `assets/knowledge/omni-rust-engineering-quality-plan/05-evidence-metrics-snapshot-2026-02-18.md`
- `assets/knowledge/omni-rust-engineering-quality-plan/07-second-pass-plan-refinement-2026-02-22.md`

## Priority Slices

## Slice 1: Keep Workspace Strict-Clippy Convergence Stable

Mapped Codex pattern:

- Workspace-wide mandatory lint policy and central governance.

Execution:

1. Keep `cargo clippy --workspace -- -D warnings` as non-negotiable gate.
2. Keep crate-level exceptions narrow and reasoned (especially PyO3 boundary
   crate `omni-core-rs`).

Validation:

```bash
cargo clippy --workspace -- -D warnings
```

Feature alignment:

- Feature A, Feature B, Feature G.

## Slice 2: Operationalize Omni-Core-RS Runtime-Test Reliability

Mapped Codex pattern:

- Runtime boundary hardening and deterministic test entrypoints.

Execution:

1. Use `scripts/rust/test_omni_core_rs.sh` as canonical test launcher for
   macOS/Nix runtime-linking compatibility.
2. Wire this launcher into local/CI Rust quality lanes where `omni-core-rs`
   tests are required.

Validation:

```bash
scripts/rust/test_omni_core_rs.sh
scripts/rust/test_omni_core_rs.sh --lib --no-fail-fast
```

Feature alignment:

- Feature B, Feature E, Feature G.

## Slice 3: Hotspot Modularization Wave (Rust Core + Bridge Impact)

Mapped Codex pattern:

- Domain-focused crate/module boundaries and maintainable implementation units.

Execution:

1. Continue decomposing top hotspot files in `xiuxian-vector`, `xiuxian-daochang`, and
   `xiuxian-skills`.
2. Prioritize files that directly affect Rust-Python API behavior first.
3. Keep interface-only `mod.rs` exports and push tests out of production modules
   where complexity is high.

Validation:

```bash
cargo test -p xiuxian-vector --no-fail-fast
cargo test -p xiuxian-daochang --no-fail-fast
```

Feature alignment:

- Feature D, Feature G.

## Slice 4: Bridge Contract Tightening (Rust <-> Python)

Mapped Codex pattern:

- Typed boundaries and explicit contract behavior around transport/runtime edges.

Execution:

1. Strengthen regression tests for `RustVectorStore` cache/default/fallback
   behavior in Python side contracts.
2. Keep schema/shape checks synchronized with Rust-side bindings.
3. Apply same contract pattern to additional bridge surfaces as they grow.

Validation:

```bash
uv run pytest packages/python/foundation/tests/unit/services/test_vector_memory_guard.py -q
uv run pytest packages/python/foundation/tests/unit/services/test_rust_vector_bridge_schema.py -q
```

Feature alignment:

- Feature D, Feature G.

## Slice 5: Dependency Security Lanes To Mandatory

Mapped Codex pattern:

- Automated supply-chain policy (`cargo-audit`, `cargo-deny`) in CI.

Execution:

1. Add or complete repository-level `cargo audit` and `cargo deny check` lanes.
2. Record every temporary exception with owner + removal condition.

Validation:

```bash
cargo audit
cargo deny check
```

Feature alignment:

- Feature F, Feature G.

## Slice 6: CI/Release Hardening

Mapped Codex pattern:

- Required-result aggregation, path-aware CI control, and release preflight.

Execution:

1. Add required-result CI aggregator for Rust quality jobs.
2. Add path-aware triggers for expensive Rust jobs.
3. Add release preflight checks for version/tag/build coherence.

Validation:

```bash
just rust-quality-gate
cargo nextest run --workspace --no-fail-fast
```

Feature alignment:

- Feature E, Feature G.

## Exit Signal

This adoption slice is considered effective when:

1. Strict workspace clippy remains continuously green.
2. `omni-core-rs` runtime-linking false negatives are removed from daily test
   flow.
3. Hotspot decomposition and bridge contract checks produce measurable scorecard
   improvements in `06-high-quality-rust-engineering-scorecard.md`.
