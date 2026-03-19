# 298. xiuxian-qianji compiler formal-audit module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area: `src/engine/compiler.rs`
- Goal: reduce compiler multi-responsibility density by extracting
  `formal_audit` parameter parsing/validation helpers into a dedicated module.

## Implementation

1. Added dedicated helper module:
   - `src/engine/compiler/formal_audit.rs`
   - moved helpers:
     - retry target extraction
     - LLM-controller mode detection
     - threshold/max-retries validation
     - output/retry-counter/score key resolution
2. Wired module into compiler:
   - `src/engine/compiler.rs` now declares `mod formal_audit;`
   - `build_formal_audit_mechanism` uses `formal_audit::*` helpers.
3. Continued suppression cleanup during extraction:
   - kept cfg-specific `formal_audit` and `llm` builder variants
     (no `unused_self` suppression).
4. Local lint hygiene:
   - gated `QianjiError` import in helper module under `feature = "llm"` to
     avoid non-feature unused import warnings.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/formal_audit.rs`
  - result:
    - `compiler.rs`: `905`
    - `compiler/formal_audit.rs`: `94`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`
- Global suppression audit:
  - `rg -n "#\\[allow\\(|#!\\[allow\\(|cfg_attr\\([^\\)]*allow\\(" packages/rust/crates -g "*.rs" | wc -l`
  - result: `0`

## Outcome

- Compiler flow remains behavior-compatible while formal-audit parsing/validation
  logic is now isolated and easier to evolve independently.
