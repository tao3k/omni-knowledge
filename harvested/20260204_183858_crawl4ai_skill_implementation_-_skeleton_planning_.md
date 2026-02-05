---
title: Crawl4ai Skill Implementation - Skeleton Planning Pattern
category: patterns
tags: ['skill', 'crawl4ai', 'chunking', 'skeleton-planning', 'mcp']
created: 2026-02-04T18:38:58.513653+00:00
---

# Crawl4ai Skill Implementation - Skeleton Planning Pattern

# Crawl4ai Skill Implementation Pattern

## Overview

Crawl4ai skill demonstrates the **Skeleton Planning Pattern** for intelligent web crawling with LLM-based chunking.

## Architecture

```
Main MCP Env                          Isolated .venv
┌─────────────────┐                  ┌─────────────────┐
│ crawl_url.py   │─── LLM Plan ───▶│ engine.py       │
│ (MCP entry)    │                  │ (crawl4ai deps) │
└─────────────────┘                  └─────────────────┘
```

## Key Patterns

### 1. Skeleton Planning Pattern

LLM sees TOC (~500 tokens) instead of full content (~10k+):
1. Extract lightweight skeleton (headers only)
2. LLM generates chunk plan based on user intent
3. Execute targeted extraction based on plan

### 2. Isolated Execution Pattern

Heavy dependencies (crawl4ai) isolated in `.venv`:
- Main env has LLM access
- Isolated env has crawl4ai
- Communication via `uv run engine.py --action smart --chunk_plan <JSON>`

### 3. Progressive Disclosure in SKILL.md

LLM-optimized format with 4-field frontmatter:
```yaml
---
name: crawl4ai
description: Use when crawling web pages, extracting markdown...
metadata:
  author: Omni Team
  version: "0.2.1"
  source: "https://github.com/unclecode/crawl4ai"
---
```

### 4. Reference Directory Pattern

```
scripts/references/
├── smart-chunking.md   # Architecture diagrams
├── chunking.md         # API reference
├── deep-crawl.md       # Usage patterns
└── README.md           # Index
```

## Action Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `smart` | LLM chunk planning + extraction | Large docs, specific queries |
| `skeleton` | TOC only | Quick overview |
| `crawl` | Full content | Complete content needed |

## Best Practices

- Use `skeleton` mode first for large documents
- Use `chunk_indices` for targeted extraction
- Set `max_depth` carefully with `max_pages` limit
- Combine with knowledge tools for RAG pipelines

## Files

- `SKILL.md` - LLM-optimized skill definition
- `README.md` - Human developer documentation
- `scripts/crawl_url.py` - MCP interface
- `scripts/engine.py` - Execution engine
- `scripts/graph.py` - Skeleton utilities
- `scripts/references/` - Detailed documentation

