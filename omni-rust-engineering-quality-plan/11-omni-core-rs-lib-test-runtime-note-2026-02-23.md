# Omni-Core-RS Lib Test Runtime Note (2026-02-23)

## Symptom

On macOS/Nix, running:

`CARGO_TARGET_DIR=/tmp/workspace-strict-proof cargo test -p omni-core-rs --lib`

may fail at process startup with:

`dyld: symbol not found in flat namespace '_PyBool_Type'`

This is a runtime linking issue for the PyO3 extension-module test binary, not
a Rust compile error.

## Working Execution Pattern

Resolve `libpython` dynamically from the active interpreter and preload it:

```bash
PYLIB="$(python3 - <<'PY'
import os
import sysconfig
print(os.path.join(sysconfig.get_config_var('LIBDIR'), sysconfig.get_config_var('LDLIBRARY')))
PY
)"

DYLD_INSERT_LIBRARIES="$PYLIB" \
CARGO_TARGET_DIR=/tmp/workspace-strict-proof \
cargo test -p omni-core-rs --no-fail-fast
```

## Canonical Wrapper

Use the repository wrapper script for day-to-day execution:

```bash
scripts/rust/test_omni_core_rs.sh
scripts/rust/test_omni_core_rs.sh --lib --no-fail-fast
```

The script resolves `libpython` and applies the preload path automatically on
macOS.

## Validation

Using the preloaded `libpython` approach:

- `cargo test -p omni-core-rs --lib --no-fail-fast` → `EXIT:0` (`5` tests passed)
- `cargo test -p omni-core-rs --no-fail-fast` → `EXIT:0` (`5` unit + `10`
  integration tests passed)

## Recommendation

For local/CI runs on macOS with this runtime layout, execute `omni-core-rs`
tests through the preload pattern above to avoid false-negative failures.
