# 入门指南

## 什么是 Codex CLI？

Codex CLI 是 OpenAI 推出的**终端 AI 编程助手**。你不需要在应用里点按钮，而是在终端里用自然语言描述你想做的事，Codex 会读取你的代码、建议修改方案、编写新功能、运行命令——这一切都是通过直接理解你机器上的项目来完成的。它和网页聊天工具最大的区别是：Codex 运行在你的本地环境，能直接读写文件、执行命令，是一个真正能动手的编程伙伴。

**终端新手？**——关键术语解释：
* **Terminal**（也叫 console 或 command line）：一个用文字输入命令而不是点击图标的窗口。macOS 上叫 Terminal.app，Windows 上叫 Windows Terminal 或 PowerShell。
* **CLI**（Command Line Interface，命令行界面）：通过输入命令而非图形界面来控制的程序。Codex CLI 就是一个 CLI 工具。
* **API key**（API 密钥）：让程序与服务进行身份验证的密钥。Codex CLI 使用 OpenAI 的 API key 来连接模型。
* **Sandbox**（沙箱）：一种安全隔离的执行环境。Codex CLI 在 Full Auto 模式下会在沙箱中运行命令，防止误操作破坏系统。
* **IDE**（Integrated Development Environment，集成开发环境）：带有内置工具的代码编辑器——VS Code、IntelliJ IDEA 和 PyCharm 都是常见的例子。

在深入了解 Codex 的各种参数和配置之前，你需要先在机器上安装并运行它。本模块将带你完成安装、身份验证，以及运行你的第一次会话。

## 前置条件

Codex CLI 需要 Node.js 22 或更高版本。支持 macOS、Linux 和 Windows（通过 WSL）。始终需要活跃的网络连接。

你需要一个 OpenAI API key（来自 platform.openai.com）或 ChatGPT Plus/Pro/Team/Enterprise 账户。API 调用会产生费用，具体取决于你使用的模型和用量。

## 安装 CLI

推荐通过 npm 全局安装，这也是最通用的方式：

```bash
npm install -g @openai/codex
```

macOS 用户也可以通过 Homebrew 安装：

```bash
brew install openai-codex
```

或者直接下载预编译的二进制文件。安装完成后，验证是否成功：

```bash
codex --version
```

## 身份验证

最简单的方式是设置 OpenAI API key 环境变量：

```bash
export OPENAI_API_KEY=sk-...
codex
```

如果你有 ChatGPT 账户，也可以通过交互式登录：

```bash
codex --login
```

本课程学习环境已预装 Codex CLI 并配置好认证，可直接进行下方操作。

## 你的第一次会话

导航到任何项目目录并启动 Codex：

```bash
cd my-project
codex
```

你会看到欢迎消息和提示符。直接用自然语言输入你想让 Codex 做的事：

```
请解释这个项目的目录结构
```

Codex 会读取你的文件、分析结构，并给你一个概要。你也可以直接带提示启动：

```bash
codex "explain this project"
```

当 Codex 需要执行编辑文件或运行命令等操作时，它的行为取决于审批策略（approval policy）——默认的 `untrusted` 模式下只自动执行受信命令（如 `ls`、`cat`），非受信命令需要用户确认。

## 写好 Prompt 的四个要素

和 Codex 对话时，好的 prompt 能大幅提升输出质量。官方推荐的有效 prompt 包含四个要素：

1. **Goal（目标）** — 明确你想要什么结果。例如"添加一个用户注册功能"
2. **Context（上下文）** — 告诉 Codex 相关背景。例如"项目使用 Express + Prisma + PostgreSQL"
3. **Constraints（约束）** — 限定范围和规则。例如"只用内置模块，不引入新依赖"
4. **Done when（完成标准）** — 定义什么时候算完成。例如"所有测试通过且 API 能正确创建用户"

一个完整的 prompt 示例：

```
帮我给 todo.js 添加优先级功能（Goal）。项目使用 Node.js，数据存在 JSON 文件中（Context）。
不引入外部依赖，保持现有功能不变（Constraints）。
完成标准：add 支持 --priority 参数，list 显示优先级（Done when）
```

初学者最常见的错误是只写 Goal 而省略其他三个要素。Codex 会尽力猜测，但补充完整能减少反复追问的次数。

就是这样——你已经准备好了。前往下一个模块学习让 Codex CLI 真正强大的斜杠命令。
# 斜杠命令

Codex CLI 提供了丰富的斜杠命令（slash commands），在交互会话中以 `/` 开头输入即可触发。这些命令是你控制 Codex 行为、管理上下文、审查代码的核心工具。掌握它们能让你从"输入提示词等结果"升级为"精准驱动 Codex 完成复杂任务"。本模块逐一介绍所有 16 个 CLI 斜杠命令。

## 会话管理

这些命令控制会话的创建、切换和恢复：

**`/new`** — 在当前会话中开始一段新的对话。不清除 AGENTS.md 和配置，但上下文历史会被重置。适合在同一个终端窗口中切换到不同任务：

```
/new
```

**`/resume`** — 恢复之前保存的会话。Codex 会自动保存会话历史，你可以从列表中选择继续：

```
/resume
```

也可以从命令行直接恢复：

```bash
codex resume --last    # 恢复最近一次会话
codex resume --all     # 列出所有保存的会话
```

**`/fork`** — 将一个保存的会话分支为新的对话线程。适合在已有对话基础上尝试不同的方案，而不影响原始会话：

```
/fork
```

**`/exit` 或 `/quit`** — 退出当前 Codex 会话。两者功能完全相同，也都可以用 `Ctrl+D` 替代。

## 模型与配置

**`/model`** — 切换当前会话使用的模型和推理深度（reasoning effort）。在需要更强推理能力或更快速响应时使用：

```
/model
```

Codex 会展示可用的模型列表让你选择。常见选择包括 `o4-mini`（快速且经济）和 `o3`（更强的推理能力）。

**`/approvals`** — 在会话中切换审批策略。三个选项：

- **Suggest**（默认）— 每个操作都需要你确认
- **Auto** — 自动批准文件编辑和命令执行
- **Never** — 全自动，不询问（适合 CI 环境）

```
/approvals
```

**`/status`** — 显示当前会话的配置信息和 token 使用量。用来确认当前模型、审批策略和上下文剩余容量：

```
/status
```

## 代码审查与上下文

**`/review`** — 审查当前工作目录的代码变更。Codex 会分析 git diff 并提供改进建议，相当于一个 AI 代码审查员：

```
/review
```

**`/diff`** — 显示当前 git diff，包括未跟踪的文件。在 Codex 修改了文件后用来快速查看所有变更：

```
/diff
```

**`/compact`** — 将对话历史压缩为摘要，释放上下文窗口中的 token 空间。长会话中当你感觉 Codex 响应变慢或开始遗忘早期上下文时使用：

```
/compact
```

**`/mention`** — 将特定文件或目录附加到当前对话中。当你想让 Codex 关注某个特定文件时使用：

```
/mention src/auth/login.ts
```

## 项目记忆

**`/init`** — 在当前目录生成 AGENTS.md 脚手架文件。Codex 会分析项目结构并生成一个包含基本信息的模板，你可以在此基础上补充项目规则：

```
/init
```

## 工具与外部服务

**`/mcp`** — 列出已配置的 MCP（Model Context Protocol）工具及其状态。在添加了 MCP 服务器后用来确认连接是否正常：

```
/mcp
```

## 账户与反馈

**`/logout`** — 登出当前账户。在共享机器上使用 Codex 后清除凭证：

```
/logout
```

**`/feedback`** — 将当前会话日志发送给 Codex 维护团队。遇到 bug 或想分享使用体验时使用：

```
/feedback
```

## 实验性与界面命令

**`/experimental`** — 开启或关闭实验性功能。Codex 团队会不定期推出新功能供用户试用，通过此命令管理：

```
/experimental
```

**`/theme`** — 切换终端主题（亮色/暗色）。根据你的终端背景色选择合适的显示主题：

```
/theme
```

**`/apps`** — 查看 Codex 集成的应用列表。了解当前可用的 IDE 集成和外部应用连接：

```
/apps
```

## 命令速查表

| 命令 | 用途 | 使用场景 |
|------|------|---------|
| `/new` | 新建对话 | 切换到不同任务 |
| `/resume` | 恢复会话 | 继续之前的工作 |
| `/fork` | 分支对话 | 在已有基础上尝试新方案 |
| `/model` | 切换模型 | 需要更强/更快的模型 |
| `/approvals` | 切换审批策略 | 中途调整自主程度 |
| `/status` | 查看配置 | 确认模型和 token 用量 |
| `/review` | 代码审查 | 审查本地变更 |
| `/diff` | 查看 diff | 检查 Codex 的修改 |
| `/compact` | 压缩上下文 | 长会话释放 token |
| `/mention` | 附加文件 | 让 Codex 关注特定文件 |
| `/init` | 生成 AGENTS.md | 初始化项目记忆 |
| `/mcp` | 列出 MCP 工具 | 确认外部工具状态 |
| `/experimental` | 实验功能 | 开关新特性 |
| `/theme` | 切换主题 | 适配终端配色 |
| `/apps` | 查看集成应用 | 了解可用连接 |
| `/logout` | 登出 | 共享机器清除凭证 |
| `/feedback` | 发送反馈 | 报告 bug |
| `/exit` | 退出 | 结束会话 |
| `/quit` | 退出 | 结束会话（同 /exit） |
# AGENTS.md 记忆系统

Codex CLI 中的记忆是指跨会话持久化的项目上下文。与每次重置的对话窗口不同，`AGENTS.md` 文件在每次 Codex 启动时自动加载，让你的 AI 助手从一开始就理解你的项目。你不需要每次都重新解释技术栈和规范——写一次，永远生效。本模块解释 AGENTS.md 的层级结构、如何创建和更新它们。

## 记忆层级

Codex CLI 的记忆系统基于 `AGENTS.md` 文件，采用分层加载机制。更具体的层级会覆盖更通用的：

**全局级**（`~/.codex/AGENTS.md`）——适用于你所有项目的通用规则。把你的个人编码风格、首选工具链、通用偏好放在这里。每个项目启动时都会加载这个文件，是你作为开发者的"身份卡"。

**项目级**（`./AGENTS.md`）——存放在项目根目录，是该项目的专属规则。把技术栈、命名规范、架构约定、常用命令放在这里。这个文件应该提交到 git，与团队共享，保证所有人使用 Codex 时遵循同样的规范。

**目录级**（`./sub/AGENTS.md`）——为项目中的子目录提供特定的规则。适合 monorepo 结构，让不同模块有各自的行为约定。Codex 在处理对应目录下的文件时会自动加载并优先使用目录级规则。

```
~/.codex/AGENTS.md          # 全局 — 所有项目共享
my-project/AGENTS.md        # 项目 — 团队共享
my-project/src/api/AGENTS.md # 目录 — 模块专属
```

优先级从低到高：全局 < 项目 < 目录。当规则冲突时，更具体的层级会覆盖更通用的。比如全局级说"使用 Jest 测试"，但某个子目录的 AGENTS.md 说"这里使用 Vitest"，那么 Codex 在处理该目录下的文件时会使用 Vitest。

Codex 还支持 `AGENTS.override.md` 文件（全局和项目级），优先级高于普通 AGENTS.md，适合临时覆盖规则而不修改主文件。

## 快速创建

最简单的方式是使用 `/init` 命令让 Codex 为你生成脚手架：

```
/init
```

Codex 会分析项目结构并生成一个包含基本信息（技术栈、命令、约定）的 AGENTS.md 模板。你可以在此基础上补充项目特有的规则。

## 编写有效的 AGENTS.md

```markdown
# 项目规范

## 技术栈
- Node.js 20 + TypeScript 5 + PostgreSQL 15
- Express for API, Prisma for ORM

## 常用命令
- `npm run dev` — 启动开发服务器
- `npm test` — 运行测试
- `npm run migrate` — 执行数据库迁移

## 编码规范
- 使用 PascalCase 命名组件
- 优先使用 const，避免 let
- 所有 API 必须有错误处理

## 禁止事项
- 不要使用 any 类型
- 禁止 prisma db push，必须用 migrate dev
```

几个关键原则：
- **具体胜于抽象** — "所有函数必须有返回类型注解"比"写好类型"有效得多
- **保持简洁** — 控制在 100 行以内，只写最有价值的信息
- **及时更新** — 项目规范变更时同步更新 AGENTS.md

## 高级技巧

### 拆分大文件为子文件

当项目规则过多导致 AGENTS.md 超过 100 行时，可以将详细规则拆分为子文件，主文件只保留概述和引用：

```markdown
# 项目规范

## 技术栈
- Node.js 20 + TypeScript 5 + PostgreSQL 15

## 详细规范
- 编码规范见 [coding-standards.md](./docs/coding-standards.md)
- API 设计规范见 [api-conventions.md](./docs/api-conventions.md)
- 测试规范见 [testing-guide.md](./docs/testing-guide.md)
```

这样做的好处是：AGENTS.md 保持简洁易读，Codex 在需要时可以通过 `/mention` 加载详细的子文件。

### 用 AGENTS.md 纠正反复出现的错误

如果你发现 Codex 在多个会话中反复犯同样的错误（比如总是用错误的导入路径、忽略某个约定），把这些纠正写进 AGENTS.md。这相当于给 Codex 一个"避免错误"的持久化清单：

```markdown
## 已知问题与纠正
- ❌ 不要从 `@/lib/db` 导入，正确路径是 `@/lib/database`
- ❌ API 路由不要以 `/api/` 开头，baseURL 已包含
- ❌ 测试文件放在 `__tests__/` 而不是 `tests/`
```

每当你发现新的重复错误，就追加一条。这个列表会随项目演进不断积累，让 Codex 越用越聪明。

## 配置文件中的相关设置

在 `~/.codex/config.toml` 中可以调整 AGENTS.md 的行为：

```toml
[agents]
# 如果找不到 AGENTS.md，查找这些替代文件名
project_doc_fallback_filenames = ["TEAM_GUIDE.md", "CONTRIBUTING.md"]

# AGENTS.md 文件最大加载大小（默认 32 KiB）
project_doc_max_bytes = 65536
```

`CODEX_HOME` 环境变量可以重定向配置和 AGENTS.md 的查找目录，默认是 `~/.codex/`。
