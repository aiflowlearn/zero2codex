# Skills System

## What is a Skill?

A skill is a directory containing reusable capabilities that Codex discovers and uses based on context. Skills are more powerful than simple commands — they support progressive loading, dynamic shell context injection, and invocation control.

## Skill Directory Structure

A skill directory contains:

- `SKILL.md` (required) — frontmatter with `name` + `description`, body with instructions
- `scripts/` (optional) — executable scripts the skill can invoke
- `references/` (optional) — reference docs injected into context
- `assets/` (optional) — static assets
- `agents/openai.yaml` (optional) — UI metadata, invocation policy, tool dependencies

```
.agents/skills/code-review/
├── SKILL.md
├── scripts/
│   └── analyze.py
└── references/
    └── checklist.md
```

## Skill Discovery (searched in order)

1. **REPO scope**: `.agents/skills` from CWD up to repo root
2. **USER scope**: `$HOME/.agents/skills`
3. **ADMIN scope**: `/etc/codex/skills`
4. **SYSTEM scope**: Built-in bundled skills

## SKILL.md Format

```markdown
---
name: my-skill
description: Does something useful
---

# Instructions
When invoked, do X, Y, Z.
```

The `description` field is critical — it controls when Codex automatically invokes the skill. Vague descriptions like "helps with code" never trigger. Include specific trigger words and use cases.

## Enabling/Disabling Skills

```toml
# ~/.codex/config.toml
[[skills.config]]
name = "my-skill"
enabled = false
```

## agents/openai.yaml

```yaml
name: my-skill
description: Does something useful
invocation_policy: manual  # manual | automatic
tools:
  - bash
  - read
  - write
```

## Built-in Tools

- `$skill-creator` — Creates new skill scaffolds
- `$skill-installer` — Installs skills from URLs

## Best Practices

- Write descriptions with specific trigger phrases and task types
- Keep SKILL.md under 500 lines; put detailed references in separate files
- Use `invocation_policy: manual` for skills with side effects (deploy, push)
- Reference support files with relative paths from SKILL.md
