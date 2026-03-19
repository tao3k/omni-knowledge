# 302. xiuxian-qianji compiler io mechanism config module extraction (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/engine/compiler.rs`
  - `src/engine/compiler/io_mechanisms.rs` (new)
- Goal: extract command/write_file/suspend parameter decoding from compiler core
  into a dedicated submodule, keeping `compiler.rs` orchestration-focused.

## Implementation

1. Added dedicated module:
   - `src/engine/compiler/io_mechanisms.rs`
   - introduced:
     - `CommandMechanismConfig`
     - `WriteFileMechanismConfig`
     - `SuspendMechanismConfig`
     - `command_mechanism_config(node_def)`
     - `write_file_mechanism_config(node_def)`
     - `suspend_mechanism_config(node_def)`
2. Updated compiler wiring:
   - `compiler.rs` now declares `mod io_mechanisms;`
   - `build_command_mechanism`, `build_write_file_mechanism`,
     `build_suspend_mechanism` now consume extracted config structs and only
     construct mechanism instances.
3. Preserved behavior defaults:
   - `command.output_key`: `stdout`
   - `write_file.output_key`: `write_file_result`
   - `suspend.reason`: `suspended`
   - `suspend.prompt`: `Waiting for input...`
   - `write_file.path` keeps `path` -> `target_path` fallback order.

## Verification

- Size audit:
  - `wc -l packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs packages/rust/crates/xiuxian-qianji/src/engine/compiler/io_mechanisms.rs`
  - result:
    - `compiler.rs`: `648`
    - `compiler/io_mechanisms.rs`: `77`
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

- `compiler.rs` complexity is further reduced while behavior stays unchanged.
- Mechanism parameter contracts are now isolated behind domain-named parser
  functions, making future evolution safer and easier to review.
