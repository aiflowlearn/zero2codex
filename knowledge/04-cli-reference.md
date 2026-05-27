# Codex CLI Reference

## Overview

This article covers all Codex CLI commands, slash commands, flags, and usage patterns. Codex is invoked from the terminal with the `codex` command followed by optional flags and a prompt.

## CLI Commands

### `codex` — Interactive Session

Starts an interactive coding session. Codex reads AGENTS.md, loads your configuration, and presents a prompt.

```bash
cd my-project
codex
```

### `codex "prompt"` — Session with Initial Prompt

Starts a session and immediately sends the given prompt:

```bash
codex "explain the authentication flow"
```

### `codex exec "prompt"` — Non-Interactive Mode

Runs Codex in non-interactive mode. Processes the prompt, outputs the result, and exits. Designed for scripting, piping, and CI/CD:

```bash
codex exec "list all TODO comments in the src directory"
```

Piping output:

```bash
git diff | codex exec "review these changes"
```

### `codex completion <shell>` — Shell Completions

Generates shell completion scripts:

```bash
codex completion zsh > ~/.zfunc/_codex
codex completion bash >> ~/.bashrc
codex completion fish > ~/.config/fish/completions/codex.fish
```

### `codex mcp add <name>` — Add MCP Server

Adds an MCP server to your configuration:

```bash
codex mcp add my-server --env API_KEY=xxx -- node server.js
codex mcp add my-http --url https://mcp.example.com/sse
```

### `codex resume` — Resume Previous Session

Resumes a previously saved conversation:

```bash
codex resume              # Pick from list
codex resume --last       # Resume most recent
codex resume --all        # Show all saved sessions
```

## Slash Commands

### CLI Slash Commands

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/approvals` | Set what Codex can do without asking | Switch between Auto and Read-only mid-session |
| `/compact` | Summarize conversation to free tokens | After long runs to prevent context overflow |
| `/diff` | Show Git diff including untracked files | Review Codex edits before committing |
| `/exit` | Exit CLI (same as `/quit`) | Alternative spelling |
| `/feedback` | Send logs to Codex maintainers | Report issues or share diagnostics |
| `/fork` | Fork a saved conversation into new thread | Branch an older session for different approach |
| `/init` | Generate AGENTS.md scaffold | Capture persistent instructions for repo |
| `/logout` | Sign out of Codex | Clear credentials on shared machine |
| `/mcp` | List configured MCP tools | Check available external tools |
| `/mention` | Attach file to conversation | Point Codex at specific files/folders |
| `/model` | Choose active model + reasoning effort | Switch models before a task |
| `/new` | Start new conversation in same session | Fresh prompt without leaving CLI |
| `/quit` | Exit CLI | Leave session |
| `/resume` | Resume saved conversation | Continue from previous session |
| `/review` | Review working tree | Second set of eyes on local changes |
| `/status` | Display session config and token usage | Verify model, approval policy, context capacity |

### IDE Extension Slash Commands

| Command | Purpose |
|---------|---------|
| `/auto-context` | Toggle Auto Context (recent files + IDE context) |
| `/cloud` | Switch to cloud mode (remote execution) |
| `/cloud-environment` | Choose cloud environment |
| `/feedback` | Open feedback dialog with optional logs |
| `/local` | Switch to local mode (workspace execution) |
| `/review` | Start code review mode |
| `/status` | Show thread ID, context usage, rate limits |

## CLI Flags

| Flag | Description |
|------|-------------|
| `--model <model>` | Override model for this session |
| `-a, --ask-for-approval <policy>` | Set approval policy (untrusted/on-request/never) |
| `--profile <name>` | Select a named config profile |
| `--login` | Browser-based ChatGPT authentication |
| `--version` | Print version and exit |
| `--help` | Display help text |

## Common Usage Patterns

```bash
# Quick question
codex exec "what does handleWebhook do?"

# Code exploration
codex "walk me through the request lifecycle"

# Focused refactor
codex "extract the validation logic into a shared utility"

# CI/CD integration
codex exec "check for security vulnerabilities in dependencies"

# With custom instructions via command substitution
codex exec "$(cat ./rules.md) migrate from MySQL to PostgreSQL"
```

## Keyboard Shortcuts (Interactive Mode)

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit the session |
| `Enter` | Send the current prompt |
| `Up/Down` | Navigate prompt history |
