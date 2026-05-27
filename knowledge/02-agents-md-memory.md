# AGENTS.md Memory System

## Overview

AGENTS.md provides persistent, layered instructions that Codex reads before every session. It acts as project-level memory — capturing coding conventions, architecture decisions, and workflow preferences so Codex understands your project from the first message.

## Resolution Order

Codex loads AGENTS.md files in a layered chain, with more specific layers overriding more general ones:

1. **Global override**: `~/.codex/AGENTS.override.md` (if exists)
2. **Global**: `~/.codex/AGENTS.md` (if exists)
3. **Project override**: Walk from repo root to CWD, checking `AGENTS.override.md`, then `AGENTS.md`, then fallback filenames
4. **Built-in defaults**

## Directory-Scoped Overrides

AGENTS.md files in subdirectories override parent instructions for tasks within that directory:

```
project-root/
├── AGENTS.md              # Project-wide rules
├── src/
│   ├── legacy/
│   │   └── AGENTS.md      # Special rules for legacy code
│   └── api/
│       └── AGENTS.md      # API-specific conventions
```

When Codex works in `src/legacy/`, it loads both the root and legacy AGENTS.md, with the legacy version taking precedence for conflicts.

## Quick Start with /init

Run `/init` in the Codex CLI to generate an AGENTS.md scaffold:

```
/init
```

This creates a template with sections for tech stack, commands, and conventions.

## Configuration in config.toml

```toml
[agents]
# Alternate filenames to check if AGENTS.md is not found
project_doc_fallback_filenames = ["TEAM_GUIDE.md", "CONTRIBUTING.md", ".cursorrules"]

# Maximum file size to load (default 32768 = 32 KiB)
project_doc_max_bytes = 65536
```

## Writing Effective AGENTS.md

- Keep under 100 lines — only include information relevant to most sessions
- Be specific: "Use vitest, not Jest" beats "Use a good test framework"
- Include tech stack, key commands, and project conventions
- Commit to git so the whole team shares the same Codex context

## Example

```markdown
# Project: Payment Service

## Stack
- Node.js 20, TypeScript 5, PostgreSQL 15
- Express for API, Prisma for ORM

## Commands
- `npm run dev` -- start with hot reload
- `npm test` -- run test suite
- `npm run migrate` -- apply pending migrations

## Conventions
- All monetary values stored as integers (cents)
- Database columns: snake_case; TypeScript: camelCase
- Never use prisma db push, always migrate dev
```

## Environment Variables

- `CODEX_HOME` — Redirect the home directory for config and AGENTS.md lookup (default: `~/.codex/`)
