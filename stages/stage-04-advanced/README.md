# Subagent 子代理

将复杂任务委托给专业的 AI Agent，每个 Agent 拥有独立的上下文窗口和自定义指令。

Subagent 让 Codex 将工作委托给专业的 AI 助手，每个助手有自己的上下文窗口、工具集和指令。它们防止长任务中的上下文污染，支持并行执行，并允许你将领域专业知识编码到可复用的 Agent 中。本模块涵盖创建、配置和有效使用 Subagent。

## 创建 Subagent

Subagent 是 TOML 格式的配置文件，存放在 `~/.codex/agents/`（个人范围）或 `.codex/agents/`（项目范围，提交到 git）。

TOML 文件定义 Agent 的身份。必需字段有三个：`name`、`description` 和 `developer_instructions`。像给专家写简报一样来写 developer_instructions：

```toml
# .codex/agents/reviewer.toml
name = "security-reviewer"
description = "Security-focused code reviewer. Use after writing auth, data handling, or payment code."
developer_instructions = """You are a senior security engineer.

Review priorities:
1. Authentication and authorization flaws
2. Injection vulnerabilities (SQL, XSS, command)
3. Data exposure and sensitive information handling

For each finding, provide: severity (Critical/High/Medium/Low), location (file:line), description, and a concrete fix."""
```

## 配置选项

除了基本字段，TOML 还支持强大的选项：

- `model` — Agent 使用的模型（如 `o4-mini` 用于快速任务，`o3` 用于深度分析）
- `model_reasoning_effort` — 推理深度（`low`/`medium`/`high`）
- `sandbox_mode` — 沙箱级别（`read-only`/`workspace-write`/`danger-full-access`）
- `nickname_candidates` — 简短别名（如 `["reviewer", "audit"]`）

Per-agent 的 MCP 服务器和 Skill 配置：

```toml
name = "db-expert"
description = "Database query optimizer and migration reviewer"
developer_instructions = "You are a database expert..."

[mcp_servers.db-tools]
command = "node"
args = ["db-mcp-server.js"]

[[skills.config]]
name = "migration-review"
enabled = true
```

## 内置 Agent

Codex 附带三个内置 Agent：

| Agent | 用途 |
|-------|------|
| `default` | 通用编程 Agent |
| `worker` | 执行委派的子任务 |
| `explorer` | 搜索和读取代码 |

## 全局 Agent 设置

在 config.toml 中控制 Agent 的并发和深度：

```toml
[agents]
max_threads = 6                # 最大并发 subagent 数（默认 6）
max_depth = 1                  # 嵌套深度（默认 1）
job_max_runtime_seconds = 300  # 每个 subagent 超时时间
```

## CSV Fan-Out

```toml
[agents]
spawn_agents_on_csv = true  # 解析 CSV 输入，每行生成一个 agent
```

## CLI 使用

在交互会话中输入 `/agent` 切换不同的 Agent 线程。
# 高级功能

掌握了 Skills、Hooks、MCP 和 Subagent 之后，本模块介绍 Codex 的几个高级工作流功能：/review 审查模式、/fork 对话分支、Worktree 隔离，以及非交互执行的最佳实践。

## /review 审查模式

`/review` 是 Codex 内置的代码审查命令。它会分析当前工作目录的变更并给出改进建议：

```
/review
```

/review 支持几种预设模式：

- **对比 base branch** — 审查当前分支相对主分支的变更
- **Uncommitted changes** — 审查未提交的修改
- **指定 commit** — 审查某个特定 commit 的变更
- **Custom instructions** — 按自定义规则审查

```
/review uncommitted
```

这相当于一个 AI 代码审查员，能快速发现安全漏洞、性能问题和代码风格问题。

## /plan 模式：先规划再动手

`/plan` 让 Codex 进入规划模式——分析任务并输出详细的实施计划，但不执行任何修改。这在复杂任务中非常有用：先让 Codex 展示它的思路，你审查计划后再让它动手。

两种方式进入 /plan 模式：

```
# 方式一：在会话中输入斜杠命令
/plan

# 方式二：启动时使用 Shift+Tab 切换
# 在输入 prompt 后，按 Shift+Tab 而不是 Enter，Codex 会以规划模式响应
```

/plan 的输出包含：任务分析、文件影响范围、实施步骤、潜在风险。你可以在规划基础上调整方向，确认后再让 Codex 执行。

## /fork 对话分支

`/fork` 将一个保存的会话分支为新的对话线程。适合在已有对话基础上尝试不同的方案，而不影响原始会话：

```
/fork
```

典型用法：你让 Codex 实现了方案 A，但想看看方案 B 是否更好。用 `/fork` 在原始对话的基础上分支出去尝试方案 B，如果方案 B 不好，原始对话不受影响。

## 线程管理：一个线程一个任务

官方推荐的工作流是**一个线程对应一个任务**。不要在同一个会话中堆砌多个不相关的需求——每个会话应该有清晰的目标。

这样做的原因：
- Codex 的上下文窗口有限，混合多个任务会稀释每个任务的上下文
- 单一任务的会话更容易用 `/resume` 恢复和继续
- `/compact` 在聚焦单一任务时效果更好

当你需要分支探索时用 `/fork`——但只在任务真正分叉时才用，不要把它当作日常操作。

## Worktree 隔离

Worktree 是 Codex App 中的 git worktree 功能，为每个任务提供独立的代码检出：

- Worktree 创建在 `$CODEX_HOME/worktrees`，使用 detached HEAD 状态
- 两种同步方式：**Overwrite**（全文件替换）和 **Apply**（补丁式）
- 自动清理：超过 4 天或总数超过 10 个的旧 worktree 会自动清理
- 清理前会自动创建快照用于恢复

适用场景：并行处理多个任务、隔离实验性修改、保持主工作区干净。

## codex exec 非交互执行

`codex exec` 是 CI/CD 和自动化的核心命令：

```bash
# 基本用法
codex exec "fix all lint errors"

# 配合管道
git diff origin/main | codex exec "review these changes"

# GitHub Action 集成
# uses: openai/codex-action@v1
```

最佳实践：
- 设置超时：避免长时间运行的任务卡住 CI
- 使用 `-a never` 实现完全自动化
- 解析退出码判断执行结果

## 常见错误与规避

官方总结了使用 Codex 时最常见的 8 个错误，了解它们能帮你少走弯路：

| # | 常见错误 | 正确做法 |
|---|---------|---------|
| 1 | 模糊的 prompt（"帮我改改代码"） | 用四要素写清晰 prompt（Goal + Context + Constraints + Done when） |
| 2 | 在一个会话中堆砌多个不相关任务 | 一个线程一个任务，用 `/new` 或 `/fork` 分开 |
| 3 | 不使用 `/plan` 就开始复杂修改 | 先 `/plan` 审查方案，再执行 |
| 4 | 忽略 `/compact` 导致上下文溢出 | 长会话中定期 `/compact` 释放 token |
| 5 | 把所有规则塞进 AGENTS.md | 超过 100 行时拆分为子文件引用 |
| 6 | 手动执行 Codex 本身能做的事 | 信任 Codex 的文件读写和命令执行能力 |
| 7 | 不使用 MCP 连接外部工具 | 需要外部 API/服务时主动配置 MCP |
| 8 | 在 CI 中不加超时和审批控制 | `codex exec` 必须设 `-a never` 和超时 |

这些错误的共同点是：没有充分利用 Codex 的能力，或者给了它不清晰的指令。解决方法回到基本原则——清晰的 prompt、合理的配置、充分利用内置功能。
