# 实战任务 3.1: 创建自定义 skill

## 目标
创建一个可复用的 code review skill。

## 操作
在项目中创建目录和文件：
```bash
mkdir -p .agents/skills/review
```

创建 `.agents/skills/review/SKILL.md`：
```markdown
---
name: review
description: Review code changes for quality, security, and best practices. Use when the user mentions review, code quality, or audit.
---

Review the following code changes. Focus on:
1. Security vulnerabilities
2. Code quality and readability
3. Best practices for the project's tech stack

For each finding, provide: severity, location, description, and a concrete fix.
```

## 完成标准
在 Codex 交互会话中输入 `/`，命令列表中出现 review skill。理解 skill 通过 `.agents/skills/` 目录发现，SKILL.md 中的 description 决定了 Codex 何时自动调用它。
# 实战任务 3.2: 用 $skill-creator 生成 skill

## 目标
使用内置的 $skill-creator 工具交互式创建 skill。

## 操作
在 Codex 交互会话中请求创建 skill：

```
用 $skill-creator 帮我创建一个 TypeScript 类型检查的 skill，自动检查新增代码中的类型错误
```

或者手动创建更复杂的 skill，加入脚本支持：

```bash
mkdir -p .agents/skills/type-check/scripts
```

创建 `SKILL.md`：
```markdown
---
name: type-check
description: Check TypeScript types in modified files. Use when the user writes new TypeScript code or modifies existing types.
---

Run the TypeScript compiler in noEmit mode on the project. Report any type errors found.
```

## 完成标准
成功创建了一个带 description 的 skill。理解 $skill-creator 可以交互式生成脚手架，也可以手动创建目录结构。
# 实战任务 3.3: 创建 hook 自动格式化

## 目标
配置 PostToolUse hook，让 Codex 每次编辑文件后自动运行格式化。

## 操作
编辑 `~/.codex/config.toml`，添加 hook 配置：

```toml
[[hooks.PostToolUse]]
command = "npx prettier --write $FILE"
```

如果项目不使用 prettier，可以用其他格式化工具：

```toml
[[hooks.PostToolUse]]
command = "npx eslint --fix $FILE"
```

在 Codex 中测试效果：
```bash
codex "在 src/ 目录下创建一个 hello.ts 文件，内容为一个简单的 hello 函数"
```

## 完成标准
Codex 创建文件后，格式化 hook 自动触发。观察 Codex 的输出日志中是否显示了格式化操作。理解 PostToolUse hook 在 Codex 使用工具后自动执行 shell 命令。
# 实战任务 3.4: 创建 hook 安全扫描

## 目标
配置 PostToolUse hook，在文件编辑后自动扫描敏感信息。

## 操作
编辑 `~/.codex/config.toml`，添加安全扫描 hook：

```toml
[[hooks.PostToolUse]]
command = "grep -c 'API_KEY\\|SECRET\\|PASSWORD\\|TOKEN' $FILE 2>/dev/null && echo '⚠️ WARNING: Potential secret detected in' $FILE >&2 || true"
```

测试 hook 是否生效：
```bash
codex "创建一个 config.ts 文件，里面包含一个示例 API 配置"
```

观察 Codex 的输出中是否显示了安全警告。如果 Codex 生成了包含 `API_KEY` 等关键词的代码，hook 会触发警告。

## 完成标准
当 Codex 创建的文件中包含敏感关键词时，hook 触发并显示警告。理解安全 hook 是防御敏感信息泄露的自动化手段，可以防止意外将密钥提交到代码中。
# 实战任务 3.5: 添加 MCP server

## 目标
用 `codex mcp add` 命令添加一个 MCP 服务器。

## 操作
添加一个文件系统 MCP 服务器（允许 Codex 访问指定目录）：

```bash
codex mcp add filesystem --env ALLOWED_DIRS=/tmp -- npx -y @modelcontextprotocol/server-filesystem /tmp
```

或者手动在 config.toml 中添加：

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
```

验证 MCP 服务器已添加：
```bash
codex
```

在会话中输入 `/mcp` 查看已配置的工具列表。

## 完成标准
`/mcp` 输出中出现了新添加的 MCP 服务器和它的工具。理解 `codex mcp add` 是添加 MCP 服务器的快捷方式，也可以直接编辑 config.toml。
# 实战任务 3.6: 用 MCP 查询外部数据

## 目标
通过 MCP 服务器让 Codex 访问外部数据源。

## 操作
确认上一步添加的 MCP 服务器已连接：
```bash
codex
```

在会话中输入 `/mcp` 确认工具可用。

然后请求 Codex 使用 MCP 工具：
```
列出 /tmp 目录下的文件
```

Codex 会通过文件系统 MCP 服务器读取 /tmp 目录内容并返回结果。

也可以尝试其他 MCP 操作：
```
读取 /tmp 下某个文件的内容
```

## 完成标准
Codex 成功通过 MCP 服务器访问了外部数据。理解 MCP 让 Codex 的能力超越了本地文件系统——可以连接数据库、API、GitHub 等任何实现了 MCP 协议的服务。
