# 274. xiuxian-wendao zhenfa XML-Lite Top-Level Test Boundary Convergence (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-wendao`
- Objective:
  - remove remaining source-include style test remap in zhenfa XML-Lite lane,
  - keep tests in package-top `tests/` with stable public boundary access,
  - preserve behavior and pass required validation gates.

## Changes

### 1) Added a thin public adapter for XML-Lite rendering

Updated:

- `packages/rust/crates/xiuxian-wendao/src/zhenfa_router/native.rs`
- `packages/rust/crates/xiuxian-wendao/src/zhenfa_router/mod.rs`

Actions:

- added `render_xml_lite_hits(payload: &LinkGraphPlannedSearchPayload) -> String`,
- re-exported the adapter from `zhenfa_router` module entry,
- kept production behavior unchanged by delegating to existing internal
  `xml_lite::render_xml_lite`.

### 2) Replaced source include harness with direct integration tests

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit.rs`

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/unit/zhenfa_router/native/xml_lite_tests.rs`

Actions:

- removed `include!("../src/zhenfa_router/native/xml_lite.rs")`,
- removed nested `mod tests { ... }` include indirection,
- rebuilt test cases against `xiuxian_wendao::zhenfa_router::render_xml_lite_hits`
  and public `link_graph` models.

## Validation Evidence

### 1) Targeted nextest lane (`zhenfa-router` feature)

```bash
cargo nextest run -p xiuxian-wendao --features zhenfa-router --test zhenfa_router_xml_lite_unit
```

Result:

- `5 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced by this slice.

### 3) Structural proof

```bash
rg -n "include!\\(\\\"\\.\\./src/|mod tests\\s*\\{" \
  packages/rust/crates/xiuxian-wendao/tests --glob "*.rs"
```

Result:

- no matches.

## Outcome

- XML-Lite test lane now follows package-top integration-test boundary rules,
- no source remap/include indirection remains for this lane,
- behavior remains stable and quality gates are green.
