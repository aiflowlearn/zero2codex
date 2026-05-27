# 实战任务 1.1: 验证 Codex CLI 环境

## 目标
确认 Codex CLI 已安装并可正常使用。

## 前置条件
如果你的环境还没有 Codex CLI，需要先安装并配置认证。以下提供两种方式：

**安装方式（任选其一）：**

npm 全局安装（推荐）：
```bash
npm install -g @openai/codex
```

Homebrew 安装（macOS）：
```bash
brew install openai-codex
```

**认证方式（任选其一）：**

方式一：API Key 环境变量：
```bash
export OPENAI_API_KEY=sk-你的密钥
```

方式二：ChatGPT 账户登录：
```bash
codex --login
```

> 本课程学习环境已预装 Codex CLI 并配置好认证，可直接进行下方操作。

## 操作
验证安装：
```bash
codex --version
```

验证模型响应：
```bash
codex exec "hello world"
```

## 完成标准
`codex --version` 输出版本号，`codex exec "hello world"` 正常返回文本回复。
# 实战任务 1.2: 第一次对话

## 目标
在项目目录中启动 Codex CLI 并完成第一次对话。

## 操作
进入一个代码项目目录（没有的话先创建空文件夹）：
```bash
mkdir -p ~/my-first-codex && cd ~/my-first-codex
```

启动 Codex CLI：
```bash
codex
```

输入你的第一个请求：
```
请解释这个项目的目录结构，不要修改任何文件。
```

## 完成标准
Codex 读取项目文件并给出分析。你观察到它是在项目内部理解代码，而不是从零开始让你描述。
# 实战任务 1.3: 发现斜杠命令

## 目标
了解 Codex CLI 的全部斜杠命令。

## 操作
在交互会话中输入 `/`，观察弹出的命令列表。

然后依次尝试以下命令：

```
/status
```
查看当前会话配置和 token 使用量。

```
/model
```
浏览可用的模型列表（不切换直接退出即可）。

```
/compact
```
压缩当前对话上下文。

也可以在终端中查看 CLI 参数：
```bash
codex --help
```

## 完成标准
看到完整的斜杠命令列表。理解斜杠命令在交互会话中使用（以 `/` 开头），而 CLI 参数在启动时指定。
# 实战任务 1.4: 体验不同审批模式

## 目标
体验 Codex CLI 不同审批策略（`untrusted`、`on-request`、`never`）的行为差异。

## 操作
先体验默认的 `untrusted` 模式（只自动执行受信命令，其余需确认）：
```bash
codex
```
在会话中输入：
```
在当前目录创建一个 hello.txt 文件
```
观察：Codex 对非受信命令会请求确认。

然后切换到 `on-request` 模式（模型自行决定何时需要确认）：
```
/approvals
```
选择 `on-request`，再次请求：
```
在 hello.txt 中写入 Hello World
```
观察：模型根据操作风险自行决定是否请求确认。

退出后从命令行体验 `never` 模式（完全自动，不询问，确保在安全目录中）：
```bash
codex -a never "给 hello.txt 添加一行时间戳"
```

## 完成标准
成功体验了不同审批模式的行为差异。理解 `untrusted` 最安全（只自动执行受信命令）、`on-request` 模型自行判断（平衡效率和安全）、`never` 完全无人值守（适合 CI/自动化场景）。掌握 `-a` / `--ask-for-approval` 标志的用法。
# 实战任务 1.5: 创建 AGENTS.md

## 目标
用 `/init` 命令为项目创建 AGENTS.md 文件。

## 操作
在 Codex 交互会话中输入：
```
/init
```

Codex 会分析项目结构并生成 AGENTS.md 脚手架。打开查看生成的内容：

```bash
cat AGENTS.md
```

手动补充你的项目规则：
```bash
cat >> AGENTS.md << 'EOF'

## 编码规范
- 使用 PascalCase 命名组件
- 优先使用 const

## 禁止事项
- 不要使用 any 类型
EOF
```

## 完成标准
项目根目录生成了 AGENTS.md 文件。打开查看内容，它应该包含项目约定和编码规范。
# 实战任务 1.6: 查看记忆文件

## 目标
了解 Codex CLI 的记忆文件层级结构。

## 操作
查看全局记忆目录：
```bash
ls ~/.codex/
cat ~/.codex/AGENTS.md
```

如果全局目录不存在，可以先创建一个简单的全局规则：
```bash
mkdir -p ~/.codex
echo '- 优先使用 TypeScript 而不是 JavaScript' > ~/.codex/AGENTS.md
```

查看项目记忆文件：
```bash
cat ./AGENTS.md
```

## 完成标准
看到全局（`~/.codex/AGENTS.md`）和项目（`./AGENTS.md`）两级记忆文件。理解项目级是最常用的，全局级是跨项目通用的偏好。
# 实战任务 1.7: 让 Codex 记住规则

## 目标
体验 AGENTS.md 的规则持久化功能。

## 操作
在 AGENTS.md 中添加一条规则：
```bash
echo '- 所有新函数必须添加 @since 日期注释' >> AGENTS.md
```

确认规则已写入：
```bash
cat AGENTS.md
```

然后开启新会话测试：
```bash
codex "帮我写一个 formatDate 工具函数"
```

## 完成标准
Codex 生成的代码遵循了 AGENTS.md 中定义的规则（函数带有 @since 日期注释）。理解你可以通过编辑 AGENTS.md 让 Codex 记住项目规则，不需要每次重新提示。
