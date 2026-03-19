# 521. Xiuxian Wendao Coactivation Touch Queue Depth

Date: 2026-03-11

## Scope

Introduce a runtime-configurable queue depth for the link-graph saliency
coactivation touch worker and cover the default resolution behavior.

## What Changed

- Added `touch_queue_depth` to `LinkGraphCoactivationRuntimeConfig` with a
  default of `DEFAULT_LINK_GRAPH_COACTIVATION_TOUCH_QUEUE_DEPTH`.
- Wired `link_graph.coactivation.touch_queue_depth` (and
  `link_graph.saliency.coactivation.touch_queue_depth`) to the runtime
  resolver alongside the environment override.
- Switched the saliency touch worker to read the resolved queue depth when
  building the `sync_channel`.
- Added a unit test that asserts the default queue depth is resolved when no
  overrides exist.

## Validation Evidence

Executed and failed:

```bash
cargo nextest run -p xiuxian-wendao
```

Outcome: failed during compilation with pre-existing unresolved imports in
`xiuxian-wendao` tests and bin targets (for example missing `unified_symbol`
exports, dependency indexer root exports, and missing `From` impls for search
args).

```bash
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Outcome: failed with existing clippy errors (use of `unwrap()` on `Result`, a
min/max comparison flagged as always true/false) and emitted warnings across
workspace crates; `xiuxian-wendao` did not complete clippy validation.
