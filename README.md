# Claude Knowledge System

A lightweight, native knowledge management plugin for [Claude Code](https://claude.com/claude-code). No external tools, no cloud services, no API keys — just files that Claude Code understands natively.

## What It Does

Gives Claude Code persistent project knowledge across sessions through three layers:

| Layer | Location | Loaded | Purpose |
|-------|----------|--------|---------|
| **Rules** | `.claude/rules/` | Always (every session) | Short directives: coding style, patterns, dos/don'ts |
| **Knowledge** | `.claude/knowledge/` | On demand (via `/query`) | Detailed context: architecture, features, deployment |
| **Memory** | `~/.claude/projects/*/memory/` | Always (built-in) | User preferences, feedback |

## Installation

Add the marketplace and install the plugin:

```bash
/plugin marketplace add gering/claude-knowledge-system
/plugin install knowledge-system
```

Or for local development/testing:

```bash
claude --plugin-dir /path/to/claude-knowledge-system/plugins/knowledge-system
```

### Updating

```bash
/plugin marketplace update claude-knowledge-system
/reload-plugins
```

Auto-updates are disabled by default for third-party marketplaces. You can enable them in `/plugin` → Marketplaces → Enable auto-update.

## Quick Start

### 1. Initialize your project

```
/knowledge-init
```

This creates the directory structure and starter files:

```
.claude/knowledge/
  _index.md           # Knowledge index
  architecture/       # Starter categories
  features/
  deployment/
```

It also adds a knowledge system section to your `CLAUDE.md`.

### 2. Add knowledge

Create knowledge files manually or use `/curate` after implementing features:

```
/curate "The notification system uses a queue-based approach with retry logic"
```

Claude decides where it belongs — as a rule (short directive) or knowledge file (detailed context).

### 3. Query knowledge

```
/query "How does the notification system work?"
```

This launches a fast Knowledge Agent (Haiku) that searches your indexed knowledge. If nothing is found, an Explore agent searches your source code as fallback.

## Skills

| Skill | Description |
|-------|-------------|
| `/query` | Search project knowledge, with source code fallback |
| `/curate` | Store new learnings as rules or knowledge files |
| `/knowledge-init` | Scaffold the knowledge system in a new project |
| `/knowledge-migrate` | Migrate from ByteRover to native knowledge system |

## How It Works

### Rules — Always Active

Files in `.claude/rules/` are loaded into every session. Use them for short, always-applicable directives:

```markdown
---
description: Service patterns for dependency injection
globs: "lib/services/**"
---
# Service Patterns
- Use lazy getters for service access
- NEVER use inline dependency lookups
```

### Knowledge — Query on Demand

Files in `.claude/knowledge/` are organized by topic and indexed in `_index.md`. They are never loaded automatically — Claude queries them via `/query` when needed.

This keeps your context window clean. Only relevant knowledge is loaded, only when you need it.

### Auto-Curate

A rule watches for pattern changes during your work. When established patterns change or new best practices emerge, Claude updates the relevant knowledge automatically.

### Knowledge Boundaries

A decision table helps Claude put information in the right place:

| Content type | Where |
|---|---|
| Short code directives (do/don't) | `.claude/rules/` |
| Workflow checklists (PR, deploy) | `CLAUDE.md` |
| Detailed architecture/features | `.claude/knowledge/` |
| User preferences/feedback | Memory (built-in) |

## Migrating from ByteRover

If you're using [ByteRover](https://byterover.dev), you can migrate to the native knowledge system:

```
/knowledge-migrate           # Run the migration
/knowledge-migrate --dry-run # Preview what would change
```

The migration:
- Reads all ByteRover knowledge files from `.brv/context-tree/`
- Deduplicates near-identical files
- Categorizes each file as a rule or knowledge entry
- Builds the knowledge index
- Cleans up ByteRover configuration
- Updates `CLAUDE.md`

## Design Decisions

**Why `/query` uses a named agent:** The Knowledge Agent has `memory: project` — it remembers which files are frequently needed and gets faster over time. An anonymous subagent would start fresh each time.

**Why `/curate` runs in-context:** Curate needs the conversation context to know what just happened — which files changed, which patterns were applied. A subagent would lose all of that.

**Why Haiku for the Knowledge Agent:** The agent does simple index lookup + file retrieval — no complex reasoning needed. Haiku is fast, cheap, and sufficient.

## License

MIT
