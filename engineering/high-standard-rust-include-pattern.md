# The Artisan's Guard: High-Standard Rust Engineering - The Include Pattern

## 1. Overview: The "Physical Split, Logical Unified" Paradigm

In high-performance Rust engineering—especially within the **Sovereign Kernel** and **Qianji (千机) Orchestrator**—we often face a conflict between Rust's module system and the need for extreme modularity. Large files (exceeding 300 lines) degrade cognitive focus and complicate concurrent development.

The **Include Pattern**, as pioneered in the `Ralph-Workflow` project, provides a sophisticated solution: physically splitting a module into logical sub-files while using the `include!` macro to merge them back into a single compilation unit.

## 2. Technical Mechanism

Unlike the standard `mod sub_module;` declaration which creates a new namespace, `include!("path.rs")` performs a literal source-level injection.

### Standard Structure (The Orchestrator File)

`src/json_parser/claude.rs`:

```rust
// 1. Module-level imports and shared types
use crate::common::truncate_text;
use std::io::{self, BufRead};

// 2. Component injection (The Core Logic)
include!("claude/parser.rs");

// 3. Display and formatting logic
include!("claude/formatting.rs");

// 4. State management
mod delta_handling;

// 5. Stream processing
include!("claude/stream_parsing.rs");

// 6. Isolated Test Suite
#[cfg(test)]
include!("claude/tests.rs");
```

## 3. Core Advantages

### A. Hyper-Modularity (Exceeding the 300-Line Rule)

By splitting `parser.rs`, `formatting.rs`, and `stream_parsing.rs`, each file remains focused on a single concern. This satisfies the **Auditor's Codex** requirement for fine-grained modules without polluting the module tree with excessive sub-namespaces.

### B. Logical Cohesion

Because `include!` injects code into the _current_ scope, all sub-files share the same `use` statements and private symbols defined in the orchestrator. This eliminates the need for repetitive `super::*` imports or complex visibility (`pub(crate)`) management between sub-components.

### C. Clean Testing

Test files can be physically separated (`tests.rs`), keeping the main logic "clean" while still allowing tests to access private functions and state—a key requirement for rigorous internal verification.

## 4. Implementation Standards for CyberXiuXian Workshop

When employing this pattern in `packages/rust/crates/*`, the following rules apply:

1.  **Orchestrator Dominance**: The main file (e.g., `mod.rs` or `lib.rs`) MUST act as the "Source of Truth" for imports and module-level documentation.
2.  **Naming Convention**: Sub-files MUST be organized in a directory named after the orchestrator file (e.g., `claude.rs` -> `claude/*.rs`).
3.  **No Nested Includes**: To avoid "Inclusion Hell," only the top-level orchestrator may use `include!`. Sub-components should remain pure logic files.
4.  **Documentation Anchors**: Each sub-file MUST start with a brief comment indicating its parent orchestrator for better traceability in IDEs.

## 5. Case Study: Ralph-Workflow Integration

In `Ralph-Workflow`, the `json_parser` handles multi-agent streaming output. By using this pattern, it manages:

- **NDJSON parsing** (internal logic)
- **Ansi-formatting** (presentation)
- **Streaming state** (state management)

This ensures that adding support for a new agent (e.g., `gemini-v2.rs`) is as simple as creating a new orchestrator and implementing the corresponding sub-files, maintaining a zero-debt architecture.

---

_Status: MANDATORY for Kernel-Level Development (V1.0)_
_Referenced: Auditor's Codex (High-Standard Era)_
