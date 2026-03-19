# 312. xiuxian-qianji compiler manifest and topology guard extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/manifest_parser.rs` (new)
  - `src/engine/compiler/topology_validation.rs` (new)
- Goal: extract manifest parsing and static topology validation from compiler
  core into dedicated modules, keeping compiler as a pure orchestration shell.

## Implementation

1. Added manifest parsing module:
   - `src/engine/compiler/manifest_parser.rs`
   - introduced:
     - `parse(manifest_toml) -> Result<QianjiManifest, QianjiError>`
   - behavior preserved:
     - TOML parsing still maps parse failures to
       `QianjiError::Topology("Failed to parse TOML: ...")`.
2. Added topology guard module:
   - `src/engine/compiler/topology_validation.rs`
   - introduced:
     - `ensure_static_acyclic(&QianjiEngine) -> Result<(), QianjiError>`
   - behavior preserved:
     - still rejects static cycles with
       `QianjiError::Topology("Manifest contains a static cycle")`.
3. Updated compiler orchestration:
   - removed in-impl `parse_manifest(...)`.
   - `compile(...)` now delegates to:
     - `manifest_parser::parse(...)`
     - `topology_validation::ensure_static_acyclic(...)`
   - compiler core line count reduced from `183` to `175`.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/manifest_parser.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/topology_validation.rs`
  - result:
    - `compiler.rs`: `175`
    - `compiler/manifest_parser.rs`: `7`
    - `compiler/topology_validation.rs`: `11`
- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler core continues converging toward a stable, low-density orchestration
  layer with domain responsibilities explicitly separated.
- Parsing and topology guards are now independently evolvable and easier to
  test/replace without re-inflating compiler complexity.
