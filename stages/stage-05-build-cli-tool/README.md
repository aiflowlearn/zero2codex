# Workflows 和自动化

用 Codex CLI 构建 CI/CD 集成、定时任务和多步骤自动化 Workflows。

当你把 Codex CLI 接入自动化基础设施时，它就从交互式助手变成了团队中的主动成员。本模块涵盖 CI/CD 集成、GitHub Actions 集成，以及构建可靠多步骤 Workflows 的模式。

## CI/CD 集成

`codex exec` 是 CI/CD 集成的基础。它以非交互方式运行 Codex，发送一个 prompt，并将结果返回到 stdout。结合 `-a never` 实现完全自动化运行。

一个常见的模式是将 Codex 作为 PR review 流程的一部分运行：

```yaml
# .github/workflows/codex-review.yml
name: Codex Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Codex Review
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          DIFF=$(git diff origin/main...HEAD)
          echo "$DIFF" | codex exec "Review these changes for bugs and security issues"
```

Codex 还提供了官方 GitHub Action：

```yaml
uses: openai/codex-action@v1
with:
  prompt: "Review the changes in this PR"
  model: "o4-mini"
```

## 多步骤 Workflow 模式

最可靠的 Workflows 将 skills、hooks 和 subagents 组合成一个管道，每个步骤都有明确的输入、输出和错误处理。

**"开发并验证"模式**将一个 Stop prompt hook（用于检查完成标准）与一个实现 skill 配对：

```toml
[[hooks.Stop]]
prompt = "Check: 1) Were all files in the spec modified? 2) Do tests pass? 3) Is the implementation complete per the requirements? If anything is incomplete, explain what remains."
timeout = 30
```

当 Codex 停止时，hook 评估是否满足所有需求。如果没有，它会告诉 Codex 缺少什么，Codex 继续工作。

**"并行 review"模式**使用多个 subagent 同时 review。一个 agent 检查安全性，另一个检查性能，第三个检查测试覆盖率。最后综合他们的发现。

## Worktree 隔离

对于跨代码库修改大量文件的任务，git worktree 为每个任务提供独立的代码检出。Codex 在隔离分支中进行更改，完成时返回 worktree 路径供你 review 或丢弃。

## Automations：定时自动化

Skills 定义的是"做什么"，Automations 定义的是"什么时候做"。Codex 支持通过 Automations 实现定时或事件触发的自动化任务：

- **定时任务** — 每天自动运行代码审查、检查依赖更新
- **事件触发** — 当特定文件变更时自动运行测试或格式化
- **条件执行** — 当 CI 失败时自动分析日志并生成修复建议

Automations 和 Skills 的关系：Skill 是可复用的能力定义，Automation 是调度这些能力的触发器。一个 Skill 可以被多个 Automation 调用，一个 Automation 也可以组合多个 Skills。

典型的自动化场景：每天早上 6 点用 security-review skill 扫描代码、每次 push 后用 test-runner skill 执行测试、每周一生成项目进度报告。
# 插件系统

Codex CLI 支持插件系统来扩展其能力。插件提供额外的 skills、MCP 服务器和自定义 Agent，让你无需手动配置就能接入新功能。

## 浏览和安装插件

在 Codex 交互会话中使用 `/plugins` 命令浏览可用的插件目录。你可以查看插件描述、安装到项目中，或卸载不再需要的插件。

```
/plugins
```

## 插件包含什么

一个插件可以包含：

- **Skills** — 新的可复用能力（自动发现并加载）
- **MCP Servers** — 外部工具和数据源的连接
- **Custom Agents** — 预定义的专业 subagent
- **Configuration** — 插件特定的 config.toml 设置

## 社区生态

Codex 的插件生态正在快速发展。常见的插件类别包括：

- **代码质量** — 自动 lint、格式化、类型检查
- **CI/CD 集成** — GitHub Actions、GitLab CI 自动化
- **数据库工具** — Schema 管理、查询优化
- **文档生成** — API 文档、README 维护
- **测试辅助** — 自动生成和维护测试

## 创建自己的插件

如果你想将团队的常用工具打包为插件供整个组织使用，可以创建一个包含 skill、MCP server 和配置的目录结构，然后通过 `$skill-installer` 分享给其他人。

插件的核心是 skill 目录 + 必要的配置文件。保持插件的 scope 清晰——一个插件做好一件事，而不是把所有功能打包在一起。
