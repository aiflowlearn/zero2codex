# Subagents and Worktrees

## Subagents / Multi-Agent

Custom agents defined in TOML files extend Codex's capabilities by providing specialized AI assistants with their own context window, tool set, and instructions.

### Agent Definition Files

- Personal agents: `~/.codex/agents/*.toml`
- Project agents: `.codex/agents/*.toml` (committed to repo)

### Required Fields

```toml
name = "code-reviewer"
description = "Reviews code for quality and security"
developer_instructions = "You are a senior code reviewer. Check for security vulnerabilities, code quality issues, and best practice violations."
```

### Optional Fields

```toml
nickname_candidates = ["reviewer", "audit"]
model = "o4-mini"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

# Per-agent MCP servers
[mcp_servers.agent-specific]
command = "node"
args = ["tools.js"]

# Per-agent skill config
[[skills.config]]
name = "review-skill"
enabled = true
```

### Built-in Agents

| Agent | Purpose |
|-------|---------|
| `default` | General-purpose coding agent |
| `worker` | Executes delegated sub-tasks |
| `explorer` | Searches and reads code |

### Global Agent Settings

```toml
[agents]
max_threads = 6                # Max concurrent subagents (default 6)
max_depth = 1                  # Nesting depth for agent spawning (default 1)
job_max_runtime_seconds = 300  # Timeout per subagent task
```

### CSV Fan-Out

```toml
[agents]
spawn_agents_on_csv = true  # Parse CSV input, spawn one agent per row
```

### CLI Usage

Type `/agent` to switch between agent threads in the TUI.

---

## Worktrees

Git worktrees enable parallel task execution in the Codex App. Each Codex task gets its own isolated checkout.

### Location

Worktrees are created at `$CODEX_HOME/worktrees` in detached HEAD state.

### Sync Methods

| Method | Description |
|--------|-------------|
| **Overwrite** | Full file replacement — replaces local files with worktree state |
| **Apply** | Patch-based — applies only the diff from worktree changes |

### Auto-Cleanup

- Worktrees older than **4 days** are automatically cleaned up
- When **10+ worktrees** exist, oldest are removed first
- **Snapshots** are taken before deletion for recovery

### Use Cases

- Run multiple Codex tasks in parallel on the same repo
- Experiment with different approaches simultaneously
- Keep main checkout clean while Codex works in isolation

---

## GitHub Action Integration

```yaml
uses: openai/codex-action@v1
```

**Key inputs**: `prompt`/`prompt-file`, `model`, `effort`, `sandbox`, `output-file`, `safety-strategy`.

**Output**: `final-message`

**Safety strategies**: `drop-sudo` (removes sudo before running) or `unprivileged-user` (runs as non-privileged user).
