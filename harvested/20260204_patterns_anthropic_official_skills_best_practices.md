# Anthropic Official Skills Best Practices

> **Category**: PATTERNS | **Date**: 2026-02-04

# Anthropic Official Skills Best Practices

## Overview

Patterns derived from analyzing [github.com/anthropics/skills](https://github.com/anthropics/skills) repository.

---

## 1. SKILL.md Format

### 4-Field Frontmatter

```yaml
---
name: <skill-identifier>
description: <when-to-use-this-skill>
metadata:
  author: <maintainer>
  version: "<semantic-version>"
  source: <upstream-repo-url>  # Optional but recommended
---
```

### Description Best Practices

- Be specific about activation triggers
- Include use cases
- Mention related skills

**Example:**
```yaml
description: "Use when writing Vue SFCs, defineProps/defineEmits, watchers, 
or using Transition/Teleport/Suspense/KeepAlive"
```

---

## 2. Knowledge Hierarchy

Organize content in tiers:

| Tier | Purpose | Content Type |
|------|---------|--------------|
| **Core** | Essential concepts, CLI commands, fundamental APIs | Tables, examples |
| **Features** | Advanced capabilities, optional patterns | Code samples |
| **Best Practices** | Recommendations, patterns to follow | Guidelines |
| **Advanced** | Edge cases, deep customization | Edge cases |

---

## 3. Reference Directory Pattern

```
skills/<skill-name>/
├── SKILL.md              # Top-level index (required)
├── README.md             # Human docs (optional)
├── references/          # Detailed docs (optional)
│   ├── topic1.md
│   └── topic2.md
└── scripts/             # Skill commands
```

Each reference file covers one topic in depth with:
- Architecture diagrams (ASCII)
- Code examples
- API reference tables

---

## 4. Command Documentation Pattern

### Parameters Table

```markdown
**Parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | str | - | Target URL (required) |
| `action` | str | "smart" | Action mode |
```

### Action Modes Table

```markdown
**Action Modes:**
| Mode | Description | Use Case |
|------|-------------|----------|
| `smart` | LLM generates chunk plan | Large docs |
| `skeleton` | Extract lightweight TOC | Quick overview |
| `crawl` | Return full content | Small pages |
```

### Examples Pattern

```markdown
**Examples:**
```python
# Smart crawl (default)
@omni("crawl4ai.CrawlUrl", {"url": "https://example.com"})

# Skeleton only
@omni("crawl4ai.CrawlUrl", {"url": "https://example.com", "action": "skeleton"})
```
```

---

## 5. Alias Pattern

Support command aliases for usability:

```python
### `crawl_url` (alias: `webCrawl`)
```

---

## 6. Cross-Reference Pattern

Reference other skills explicitly:

```markdown
| Topic | Description | Reference |
|-------|-------------|-----------|
| Skeleton Planning | LLM sees TOC (~500 tokens) | [smart-chunking.md](references/smart-chunking.md) |
```

---

## 7. Best Practices Checklist

- [ ] 4-field frontmatter with source URL
- [ ] Clear description with activation triggers
- [ ] Parameters table for each command
- [ ] Examples for all modes/actions
- [ ] Reference directory for deep dives
- [ ] Cross-references between related skills
- [ ] Semantic versioning in metadata
- [ ] Core/Features/Best Practices/Advanced organization

---

## 8. Anti-Patterns to Avoid

- **Verbose SKILL.md**: Remove boilerplate, keep actionable content
- **Inconsistent metadata**: Use same schema across all skills
- **Missing examples**: Always provide copy-paste examples
- **Deep nesting**: Limit reference depth to 1 level
- **No activation signals**: Description should clearly trigger skill routing

