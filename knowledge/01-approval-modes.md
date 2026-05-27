# Approval Policies and Sandbox Modes

## Overview

Codex CLI uses an **approval policy** system to control what actions require user confirmation, and a **sandbox mode** system to isolate command execution. These two settings together determine how autonomously and how safely Codex operates.

## Approval Policies

Codex supports four approval policies, configurable in `~/.codex/config.toml` or switchable mid-session with `/approvals`:

| Policy | Config Value | Behavior |
|--------|-------------|----------|
| **Untrusted** | `"untrusted"` | Only auto-executes trusted commands (e.g. `ls`, `cat`). Escalates all others. Most secure. |
| **On-request** | `"on-request"` | The model decides when to ask for user approval. Good balance of autonomy and safety. |
| **On-failure** | `"on-failure"` | (Deprecated) Runs commands without approval, only asks if a command fails. Prefer `on-request` or `never`. |
| **Never** | `"never"` | Executes everything without prompting. Use only in fully automated or CI environments. |

```toml
# ~/.codex/config.toml
approval_policy = "on-request"
```

Override per invocation with CLI flags:

```bash
codex -a never "refactor the auth module"
```

Switch mid-session using the `/approvals` slash command.

## Sandbox Modes

Sandbox modes control the level of system access Codex has when executing commands:

| Mode | Config Value | Behavior |
|------|-------------|----------|
| **Read-only** | `"read-only"` | Codex can read files but cannot write anything. Safest for review and analysis. |
| **Workspace-write** | `"workspace-write"` | Codex can write only within the project workspace. Good balance of safety and capability. |
| **Danger: Full Access** | `"danger-full-access"` | No restrictions. Codex can read and write anywhere. Use with caution. |

```toml
# ~/.codex/config.toml
sandbox_mode = "workspace-write"
```

### Platform Support

| Platform | Sandbox Technology |
|----------|-------------------|
| macOS | Seatbelt (Apple's built-in sandboxing) |
| Linux | bubblewrap (bwrap) |
| Windows | Native sandboxing |

### Extending Write Boundaries

In workspace-write mode, explicitly allow additional directories:

```toml
[sandbox_workspace_write]
writable_roots = ["/tmp/build", "/opt/output"]
```

## Combining Approval Policy and Sandbox

The two settings are independent and compose together:

| Scenario | approval_policy | sandbox_mode |
|----------|----------------|-------------|
| Learning / exploring | `untrusted` | `read-only` |
| Normal development | `on-request` | `workspace-write` |
| CI / automation | `never` | `workspace-write` or `danger-full-access` |

## Requirements.toml

Administrators can enforce constraints via `~/.codex/requirements.toml`. This file overrides user config and cannot be changed by the user. It controls allowed approval policies, sandbox modes, MCP server allowlists, and mandatory rules.
