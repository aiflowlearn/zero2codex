# Skills

Skills 是 Codex 根据上下文自动发现和使用的可复用能力。它们比简单的命令更强大：支持渐进式加载以保持轻量、动态上下文注入和调用控制。本模块将展示如何设计和构建有效的 skills。

## Skills 如何加载

Codex 保持 skill 加载的轻量化。Skill 描述会被加载，让 Codex 知道有哪些可用能力。完整的 `SKILL.md` 内容仅在 skill 被调用时才加载，支撑文件仅在需要时才被读取。

这意味着你可以安装很多 skills 而不会淹没上下文窗口。Codex 从描述中知道它们的存在，然后只为它决定使用的 skills 加载实际指令。

Skills 存放在 `.agents/skills/<name>/SKILL.md`（项目作用域，提交到 git）或 `$HOME/.agents/skills/<name>/SKILL.md`（个人作用域）。发现顺序为 REPO → USER → ADMIN → SYSTEM：

```
.agents/skills/code-review/
├── SKILL.md          # 指令（必需）
├── scripts/
│   └── analyze.py
└── references/
    └── checklist.md
```

## 编写有效的 Skill 描述

description 字段是 skill 中最重要的部分。它控制 Codex 何时自动调用该 skill，必须包含足够的信号让 Codex 将其与真实用户请求匹配。像 "helps with code" 这样模糊的描述永远不会触发。包含具体触发词的明确描述才有效：

```markdown
---
name: security-review
description: Scan code for security vulnerabilities including injection flaws, authentication issues, and data exposure. Use when reviewing code, preparing a PR, or when the user mentions security or audit.
---

Review the code for security vulnerabilities. Focus on:
1. SQL injection and XSS
2. Authentication and authorization flaws
3. Sensitive data exposure

For each finding, provide: severity, location, and a concrete fix.
```

包含任务类型（"scan"、"generate"、"analyze"）、主题领域（"security"、"API"、"database"）和显式触发短语。

## Skill 目录结构

一个 skill 目录可以包含：

| 文件 | 必需 | 用途 |
|------|------|------|
| `SKILL.md` | 是 | frontmatter（name + description）+ 指令正文 |
| `scripts/` | 否 | 可执行脚本 |
| `references/` | 否 | 参考文档 |
| `assets/` | 否 | 静态资源 |
| `agents/openai.yaml` | 否 | UI 元数据、调用策略、工具依赖 |

## 启用和禁用 Skills

在 config.toml 中控制 skill 的启用状态：

```toml
# ~/.codex/config.toml
[[skills.config]]
name = "my-skill"
enabled = false
```

## 内置工具

Codex 附带两个内置 skill 工具：
- `$skill-creator` — 交互式创建新 skill 脚手架
- `$skill-installer` — 从 URL 安装 skills

## 最佳实践

- 保持 SKILL.md 在 500 行以内；将详细参考资料放在单独的文件中
- 使用 `invocation_policy: manual`（在 `agents/openai.yaml` 中）为有副作用的 skill（部署、推送）限制为手动调用
- 用相对路径从 SKILL.md 引用支撑文件

### 编写高质量 Skill 的三条规则

**一个 Skill 只做一件事**。不要创建"全能" skill——把代码审查、测试生成和部署分开成独立的 skills。scope 清晰的 skill 更容易被 Codex 正确触发，也更容易维护。

**在描述中包含 2-3 个具体用例**。模糊的描述让 Codex 无法判断何时调用，具体的用例让它能精确匹配用户意图：

```markdown
# 好的描述——包含具体触发场景
description: "Generate unit tests for JavaScript/TypeScript functions. Use when the user asks to 'add tests', 'write test cases', or 'cover this function with tests'. Supports Jest and Vitest."
```

**定义触发短语**。在描述中显式列出用户可能说的关键词——"add tests"、"write test cases"、"cover with tests"。这些短语是 Codex 决定是否调用 skill 的直接信号。
# Hooks

Hooks 是生命周期事件处理器，在 Codex 会话的特定时刻自动触发。它们让你无需手动操作就能执行团队标准、运行自动化检查、集成外部工具。本模块介绍 Hooks 的配置和常见用法。

## Hook 事件

Codex 支持在关键生命周期节点触发 hooks：

| 事件 | 触发时机 |
|------|---------|
| **PreToolUse** | Codex 使用工具之前（读文件、写文件、执行命令） |
| **PostToolUse** | 工具使用之后 |
| **Stop** | Codex 即将停止响应时 |

## 配置 Hooks

Hooks 在 `~/.codex/config.toml` 中配置。每个 hook 指定事件类型和动作（command 或 prompt）：

```toml
# 文件写入后自动格式化
[[hooks.PostToolUse]]
command = "npx prettier --write $FILE"

# Codex 停止前检查完成度
[[hooks.Stop]]
prompt = "Check: 1) Were all files in the spec modified? 2) Do tests pass? If incomplete, continue."
timeout = 30
```

用 `/hooks` 命令在 CLI 中交互查看和管理已配置的 hooks。

## 命令型 Hook

命令型 hook 在指定事件触发时执行 shell 命令。最常见的用例是自动格式化：

```toml
# 每次文件写入后自动运行 prettier
[[hooks.PostToolUse]]
command = "npx prettier --write $FILE"
```

也用于安全扫描：

```toml
# 检查文件中是否有敏感信息泄露
[[hooks.PostToolUse]]
command = "grep -c 'API_KEY\\|SECRET\\|PASSWORD' $FILE && echo 'WARNING: Potential secret detected' >&2 || true"
```

## Prompt 型 Hook

Prompt 型 hook 在事件触发时向 Codex 注入一条提示。最常见的用例是完成度验证——在 Codex 停止前检查它是否真的完成了任务：

```toml
[[hooks.Stop]]
prompt = "Verify: 1) All requested files were created. 2) Tests pass. 3) No TODO comments left. If incomplete, continue working."
timeout = 30
```

这个模式特别有效：当 Codex 认为自己完成了但实际有遗漏时，prompt hook 会告诉它缺少什么，让它继续工作。

## 最佳实践

- 保持 hook 命令快速——慢速 hook 会拖慢交互体验
- 命令型 hook 用于自动化（格式化、扫描），prompt 型 hook 用于验证
- 在加入 config.toml 前先独立测试 hook 命令
- 用文件模式匹配避免在不相关的文件变更上触发
# MCP（Model Context Protocol）

MCP 让 Codex 连接外部工具和数据源，扩展它的能力边界。通过 MCP，Codex 可以访问数据库、调用 API、使用第三方服务——所有这些都在同一个对话中完成。本模块介绍 MCP 的两种服务器类型和配置方法。

## MCP 的两种服务器

### STDIO 服务器

运行本地进程，通过标准输入/输出通信。适合本地工具、自定义脚本：

```toml
# ~/.codex/config.toml
[mcp_servers.my-stdio]
command = "node"
args = ["server.js"]
env = { API_KEY = "secret" }
env_vars = ["EXISTING_VAR"]
cwd = "/path/to/workdir"
```

### Streamable HTTP 服务器

连接远程 MCP 端点。适合云服务、第三方 API：

```toml
[mcp_servers.my-http]
url = "https://mcp.example.com/sse"
bearer_token_env_var = "MCP_BEARER_TOKEN"
http_headers = { X-Custom = "value" }
```

## 通用配置选项

| 选项 | 说明 |
|------|------|
| `startup_timeout_sec` | 服务器启动超时（秒） |
| `tool_timeout_sec` | 单次工具调用超时（秒） |
| `enabled` | 启用/禁用（默认 true） |
| `required` | 服务器无法启动时是否报错（默认 false） |
| `enabled_tools` | 工具白名单 |
| `disabled_tools` | 工具黑名单 |

## 用 CLI 添加 MCP 服务器

最快捷的方式是用命令行添加：

```bash
# 添加 STDIO 服务器
codex mcp add my-server --env API_KEY=xxx -- node server.js

# 添加 HTTP 服务器
codex mcp add my-http --url https://mcp.example.com/sse
```

添加后 Codex 会自动将服务器配置写入 config.toml。

## 验证和使用

在交互会话中输入 `/mcp` 查看已配置的 MCP 工具和连接状态。确认服务器正常后，直接在对话中请求 Codex 使用对应的工具：

```
列出当前仓库的所有 open pull requests
```

Codex 会自动调用已连接的 MCP 服务器完成请求。

## 示例：添加 GitHub MCP 服务器

```toml
[mcp_servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_PERSONAL_ACCESS_TOKEN = "ghp_..." }
```

添加后你可以直接让 Codex 操作 GitHub——查看 PR、搜索 issue、审查代码，而不需要手动运行 `gh` 命令。

## 何时使用 MCP

MCP 不是万能的——它适合连接外部服务和数据源，但不适合 Codex 本身就能做的事。判断标准：

**适合用 MCP 的场景：**
- 连接外部 API（GitHub、Jira、Slack 等）
- 访问数据库或云服务
- 使用需要认证的第三方工具
- 调用本地开发服务器或自定义脚本

**不需要 MCP 的场景：**
- Codex 已经能做的事（读文件、写文件、运行命令、搜索代码）
- 简单的代码分析和重构
- 使用 `git`、`npm`、`docker` 等 CLI 工具（Codex 可以直接在终端运行）

原则是：**如果 Codex 本身就能完成，就不需要 MCP。MCP 扩展的是 Codex 原生能力之外的边界。**
