# 为项目配置 Codex CLI

让 Codex CLI 在项目上高效工作大约需要十分钟的设置时间。回报是：每次启动 Codex 都自动使用你偏好的模型、正确的审批策略和合适的沙箱级别，不需要重复传参数。本模块介绍配置文件、项目记忆和环境变量三个核心设置。

## 初始化项目记忆

在项目根目录手动创建一个 `AGENTS.md` 文件，或者用 `/init` 命令让 Codex 生成脚手架。Codex 每次启动时会自动读取这个文件，将其作为系统级指令注入上下文。把技术栈、关键命令和约定写进去，Codex 从第一条消息就能理解你的项目。

`AGENTS.md` 的写法要简洁、具体、每条都和多数会话相关。控制在 100 行以内，只写最有价值的信息。更详细的说明见上一模块的「AGENTS.md 记忆系统」。

## 配置文件

Codex CLI 的全局配置文件位于 `~/.codex/config.toml`（TOML 格式）。这是你设置模型、审批策略、沙箱和自定义提供商的地方。

`model` 控制 Codex 使用的默认模型。`approval_policy` 控制自动化程度——`untrusted` 只自动执行受信命令（如 `ls`、`cat`）、`on-request` 模型自行判断何时需要确认、`never` 全部自动执行。`sandbox_mode` 控制系统访问级别——`read-only` 只读、`workspace-write` 可写工作区、`danger-full-access` 完全访问。

```toml
# ~/.codex/config.toml
model = "glm-4-flash"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
model_reasoning_effort = "medium"
```

Codex 支持 **Profiles**（配置预设），通过 `--profile` 选择不同的配置组合：

```toml
[profiles.dev]
model = "glm-4-flash"
approval_policy = "on-request"

[profiles.strict]
approval_policy = "untrusted"
sandbox_mode = "read-only"
```

```bash
codex --profile dev "run tests"
```

如果你使用第三方兼容 API（如 DeepSeek 或自托管端点），通过 `model_providers` 配置自定义 base URL：

```toml
[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
api_key_env_var = "DEEPSEEK_API_KEY"
```

配置的优先级为：命令行参数 > Profile > `~/.codex/config.toml` > 内置默认。也就是说 `codex --model o3` 会覆盖配置文件中的 `model` 设置。

## 配置层级：个人 vs 项目

Codex 有两种配置位置，各有用途：

**个人配置**（`~/.codex/config.toml`）——你个人的全局偏好。适合放模型选择、审批策略等与具体项目无关的设置。这些配置不会提交到 git，不会影响团队其他成员。

**项目配置**（`.codex/config.toml`，注意在项目根目录下）——项目级别的配置。适合放团队共享的设置：模型提供商、MCP 服务器、特定的审批策略。这个文件应该提交到 git，让所有团队成员使用一致的 Codex 行为。

```bash
~/.codex/config.toml      # 个人配置——你的偏好
my-project/.codex/config.toml  # 项目配置——团队共享
```

当两者有冲突时，项目配置覆盖个人配置。这样你可以有个人默认设置，同时不覆盖团队约定。

## 环境变量

Codex CLI 依赖几个环境变量。`OPENAI_API_KEY` 是唯一的必需项——没有它 Codex 无法启动。将 API key 写入 shell 配置文件（`~/.zshrc` 或 `~/.bashrc`），这样每次打开终端都自动可用。

`CODEX_HOME` 可选，用于指定 Codex 的配置目录。默认是 `~/.codex`，如果你想在多个项目间切换不同配置，可以设置不同的 `CODEX_HOME`。`OPENAI_BASE_URL` 也是可选的，当你使用代理或自托管端点时覆盖默认的 API 地址。

```bash
# ~/.zshrc 或 ~/.bashrc
export OPENAI_API_KEY="sk-..."
export CODEX_HOME="$HOME/.codex"
```

修改 shell 配置文件后运行 `source ~/.zshrc`（或重启终端）使其生效。验证配置是否正确：

```bash
echo $OPENAI_API_KEY
# 应该输出你的 key（以 sk- 开头）
codex --version
# 确认 Codex CLI 已安装
```

三个环境变量中，只有 `OPENAI_API_KEY` 是硬性要求。`CODEX_HOME` 和 `OPENAI_BASE_URL` 按需配置，大多数情况下保持默认即可。
# 命令深入

你已经知道用 `codex` 启动交互会话。本模块介绍日常开发中真正高频的用法——非交互执行、模型切换、管道操作，以及让 Codex 融入 shell 工作流的技巧。

## 交互与单次模式

直接输入 `codex` 进入交互模式——一个持续的多轮对话，Codex 保持上下文，你可以连续追问。适合需要多步操作的任务：重构代码、修复 bug、逐步构建功能。

单次提问用引号包裹 prompt：`codex "explain this function"`。Codex 执行完毕后直接退出，不进入交互。适合一次性问题：查看解释、生成代码片段、快速分析。

```
# 交互模式——持续对话
codex

# 单次提问——执行后退出
codex "what does the handlePayment function do?"
```

交互模式中可以随时用 `Ctrl+C` 中断当前生成，输入新指令。用 `Ctrl+D` 或输入 `/quit` 退出。

## codex exec：非交互执行

`codex exec` 是 Codex 的非交互模式，专为脚本集成和自动化设计。它接受一个 prompt，处理后输出结果并退出，不会进入交互界面：

```bash
# 基本用法
codex exec "list all TODO comments in src/"

# 配合管道
git diff | codex exec "review these changes"

# 输出到文件
codex exec "generate API documentation" > api-docs.md
```

`codex exec` 是 CI/CD 集成的基础。结合 `-a never` 实现完全自动化运行：

```bash
codex exec -a never "fix all lint errors in src/"
```

## 模型与指令

`--model` 在命令行覆盖配置文件中的默认模型。`o4-mini` 是性价比最高的日常选择。`o3` 推理能力更强，适合复杂的架构决策和多步调试。在不确定时用 `o4-mini`，遇到它处理不了的问题再升级。

## 推理深度（Reasoning Effort）

除了选择模型，Codex 还允许你控制模型的推理深度。推理深度越高，模型花更多时间"思考"，适合复杂任务；深度越低，响应更快，适合简单任务：

| 推理深度 | 适用场景 | 示例 |
|----------|---------|------|
| **Low** | 简单查询、格式化、小修改 | "把这个函数改名为 handleAuth" |
| **Medium** | 日常开发任务（默认） | "添加一个带验证的注册 API" |
| **High** | 复杂调试、架构决策 | "找出这个内存泄漏的根本原因" |
| **Extra High** | 极端复杂问题、多系统集成 | "重新设计认证系统以支持 SSO" |

通过命令行设置：

```bash
codex --model_reasoning_effort high "debug the intermittent timeout issue"
```

或在 config.toml 中设置默认值：

```toml
model_reasoning_effort = "medium"  # 大多数情况下 medium 是最佳选择
```

最佳实践是日常使用 Medium，只在遇到 Codex 反复无法解决的问题时升级到 High 或 Extra High。Low 适合需要快速响应的简单操作。

通过管道或命令替换把指令文件内容注入 prompt，相当于一个临时的 `AGENTS.md`——适合按任务加载不同规则，而不是把所有规则塞进一个文件。

```
# 切换模型
codex --model o3 "debug the payment timeout issue"

# 通过命令替换加载指令文件
codex exec "$(cat ./code-review-rules.md) review PR #42"

# 通过管道加载指令文件
cat ./code-review-rules.md | codex exec "follow these rules and review PR #42"
```

## Shell 集成

`codex completion bash|zsh|fish` 生成 shell 补全脚本。把它加入 shell 配置文件后，输入 `codex --` 按 Tab 就能看到所有可用选项。小投入大回报——省去反复查文档。

管道是 Codex 的隐藏杀手锏。任何命令的输出都可以管道给 `codex exec` 分析：

```bash
# 生成补全并加载
codex completion zsh > ~/.zfunc/_codex

# 管道用法
git diff | codex exec "summarize the changes"
docker logs app 2>&1 | codex exec "find the root cause of the crash"
```

别名让高频操作更短：

```bash
# 加入 ~/.zshrc 或 ~/.bashrc
alias cx='codex exec'

# 使用
cx "what does this regex match?"
```

## 会话恢复

`codex resume` 恢复之前保存的会话。三个选项：

```bash
codex resume              # 从列表选择
codex resume --last       # 恢复最近一次
codex resume --all        # 列出所有保存的会话
```
