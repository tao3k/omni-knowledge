# Agent Skills Best Practices - Progressive Disclosure

> **Category**: ARCHITECTURE | **Date**: 2026-02-04

# Agent Skills Best Practices

Source: https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills

## Core Principle: Progressive Disclosure

### Level 1: Minimal Metadata
```yaml
---
name: skill-name
description: Brief description
---
```

### Level 2: SKILL.md Body
Full content loaded when Claude deems relevant.

### Level 3+: Additional Files
Referenced by name for specific scenarios.

## Standard SKILL.md Structure

```yaml
---
name: skill-name
version: 0.1.0
description: Brief description
routing_keywords:
  - keyword1
  - keyword2
execution_mode: subprocess
intents:
  - Use case 1
  - Use case 2
---
```

## Development Best Practices

1. **Start with evaluation**: Identify gaps by observing agent behavior
2. **Structure for scale**: Split SKILL.md when unwieldy
3. **Think from Claude's perspective**: name/description determine triggering
4. **Iterate with Claude**: Let Claude capture successful patterns

## Code Execution

Skills can include pre-written scripts executed directly without loading into context.
