# Hooks

## Overview

Hooks are lifecycle event handlers that run automatically at specific points during a Codex session. They let you enforce team standards, run automated checks, and integrate external tools without manual intervention.

## Hook Events

Codex supports lifecycle hooks that trigger at key moments:

| Event | When It Fires |
|-------|---------------|
| **PreToolUse** | Before Codex uses a tool (file read, write, command execution) |
| **PostToolUse** | After a tool is used |
| **Stop** | When Codex is about to stop responding |

## Configuration

Hooks are configured in `~/.codex/config.toml`. Each hook specifies an event type and an action (command or prompt):

```toml
# Auto-format code after file writes
[[hooks.PreToolUse]]
command = "npx prettier --write $FILE"

# Prompt-based hook: check completion criteria before stopping
[[hooks.Stop]]
prompt = "Check: 1) Were all files in the spec modified? 2) Do tests pass? If anything is incomplete, explain what remains."
timeout = 30
```

## Managing Hooks

Use `/hooks` in the CLI to view and manage configured hooks interactively.

## Common Use Cases

### Auto-Formatting on Save

Run a formatter every time Codex writes a file:

```toml
[[hooks.PostToolUse]]
command = "npx prettier --write $FILE"
```

### Security Scan After Edits

Check for sensitive information after code changes:

```toml
[[hooks.PostToolUse]]
command = "grep -r 'API_KEY\\|SECRET\\|PASSWORD' $FILE && echo 'WARNING: Potential secret detected' >&2"
```

### Completion Verification

Ensure Codex fully completes a task before stopping:

```toml
[[hooks.Stop]]
prompt = "Verify: 1) All requested files were created or modified. 2) Tests pass. 3) No TODO comments left behind. If incomplete, continue working."
timeout = 30
```

## Best Practices

- Keep hook commands fast — slow hooks degrade the interactive experience
- Use prompt hooks for verification and command hooks for automation
- Test hooks independently before adding them to config.toml
- Be specific about file patterns to avoid triggering on irrelevant changes
