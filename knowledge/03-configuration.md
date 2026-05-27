# Codex CLI Configuration

## Overview

Codex CLI reads configuration from `~/.codex/config.toml` (TOML format) and environment variables. Configuration lets you set default values for the model, approval policy, and provider, so you do not need to pass CLI flags on every invocation.

## Configuration File Location

- Path: `~/.codex/config.toml`
- Override with `CODEX_HOME` env var

Create the config directory if needed:

```bash
mkdir -p ~/.codex
```

## Resolution Order (highest priority first)

1. CLI flags (`--model`, `-a`/`--ask-for-approval`, etc.)
2. Profile values (selected via `--profile <name>`)
3. Root config.toml values
4. Built-in defaults

## Key Settings

| Setting | Description | Example |
|---------|-------------|---------|
| `model` | Default model | `"o4-mini"` |
| `approval_policy` | What requires user approval | `"on-request"` |
| `sandbox_mode` | Sandbox restriction level | `"workspace-write"` |
| `model_reasoning_effort` | Reasoning depth | `"low"` / `"medium"` / `"high"` |

```toml
# ~/.codex/config.toml
model = "glm-4-flash"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
model_reasoning_effort = "medium"
```

Common model choices:
- `glm-4-flash` — Default for this course. Fast and cost-effective.
- `o4-mini` — OpenAI default. Fast and cost-effective for most tasks.
- `o3` — More capable reasoning. Better for complex tasks.

## Profiles

Named config presets selected with `--profile`:

```toml
[profiles.dev]
model = "glm-4-flash"
approval_policy = "on-request"

[profiles.strict]
approval_policy = "untrusted"
sandbox_mode = "read-only"
```

Select with:

```bash
codex --profile dev "run tests"
```

## Features

```toml
[features]
beta = ["multi_agent"]         # Beta features
experimental = ["web_search"]  # Experimental features
stable = ["compact"]           # Explicitly enable stable features
```

## Shell Environment Policy

Control env var passing:

```toml
[shell_environment_policy]
default = "allow"       # "allow" | "deny" | "preserve"
set = { FOO = "bar" }
allow = ["MY_VAR_*"]
deny = ["SECRET_*"]
```

## Custom Providers

Codex supports non-OpenAI models via custom providers:

```toml
[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
api_key_env_var = "DEEPSEEK_API_KEY"

[[model_providers.deepseek.models]]
id = "deepseek-chat"
name = "DeepSeek V3"
```

Reference in the model field:

```toml
model = "deepseek/deepseek-chat"
```

## Additional Instructions

Codex 没有独立的 `--instructions` 标志。可以通过管道或命令替换把指令文件内容注入 prompt：

```bash
# 通过命令替换
codex exec "$(cat ./my-instructions.md) review the code"

# 通过管道
cat ./my-instructions.md | codex exec "follow these rules and review the code"
```

也可以在 config.toml 中设置 `instructions` 字段，Codex 会在每次会话时加载：

```toml
instructions = "~/.codex/my-instructions.md"
```

These supplement (but do not replace) AGENTS.md content.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | Your OpenAI API key. Codex cannot function without this. |
| `CODEX_HOME` | No | Override config directory (default: `~/.codex/`) |
| `OPENAI_BASE_URL` | No | Override API base URL for proxies or self-hosted endpoints |
| `OPENAI_ORG_ID` | No | OpenAI organization ID for billing |

```bash
# ~/.zshrc or ~/.bashrc
export OPENAI_API_KEY="sk-..."
```

## Full Configuration Example

```toml
# ~/.codex/config.toml

model = "glm-4-flash"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
model_reasoning_effort = "medium"

[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
api_key_env_var = "DEEPSEEK_API_KEY"

[[model_providers.deepseek.models]]
id = "deepseek-chat"
name = "DeepSeek V3"

[profiles.dev]
model = "glm-4-flash"
approval_policy = "on-request"

[profiles.strict]
approval_policy = "untrusted"
sandbox_mode = "read-only"
```

## JSON Schema

Available at: `https://developers.openai.com/codex/config-schema.json`

## Troubleshooting

**Codex ignores my config file:**
- Verify the file is at `~/.codex/config.toml` (not `.yaml` or `.json`).
- Check TOML syntax. Invalid TOML silently falls back to defaults.
- Ensure no CLI flags are overriding the config.

**Model not found error:**
- Verify the model name is correct and available for your API key.
- For custom providers, check that the provider and model IDs match your config.

**API key not recognized:**
- Ensure `OPENAI_API_KEY` is exported: `export OPENAI_API_KEY="sk-..."`.
- For custom providers, check the `api_key_env_var` points to an environment variable that is set.
