# Anthropic SKILL.md Official Pattern

> **Category**: ARCHITECTURE | **Date**: 2026-02-04

# Anthropic SKILL.md Official Pattern

Source: https://github.com/anthropics/skills

## Root SKILL.md (4 fields)

```yaml
---
name: skill-name
description: One-line purpose. Use when <use-case-1>, <use-case-2>, or <use-case-3>.
metadata:
  author: Author Name
  version: "YYYY.MM.DD"
---
```

## Reference Files (2 fields)

```yaml
---
name: reference-name
description: Brief purpose description.
---
```

## Key Conventions

1. **Description Pattern**: "<Topic>. Use when <use-case-1>, <use-case-2>, or <use-case-3>."

2. **Progressive Disclosure**:
   - Level 1: name + description (system prompt)
   - Level 2: Full SKILL.md body (when relevant)
   - Level 3+: Additional files (when needed)

3. **Reference Files**: Minimal metadata for fast parsing

4. **Root SKILL.md**: Full metadata (author, version) for management
