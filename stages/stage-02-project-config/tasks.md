# 实战任务 2.1: 编辑 config.toml

## 目标
创建 Codex CLI 的全局配置文件，设置默认模型和审批策略。

## 操作
创建配置目录和文件：
```bash
mkdir -p ~/.codex
cat > ~/.codex/config.toml << 'EOF'
model = "glm-4-flash"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
model_reasoning_effort = "medium"
EOF
```

验证文件内容：
```bash
cat ~/.codex/config.toml
```

启动 Codex 确认配置已加载：
```bash
codex
```
在会话中输入 `/status` 查看当前配置。

## 完成标准
`~/.codex/config.toml` 文件存在且内容正确。`/status` 输出确认使用 glm-4-flash 模型和 on-request 审批策略。理解 config.toml 是 Codex 的核心配置，使用 TOML 格式（不是 YAML 或 JSON）。
# 实战任务 2.2: 配置审批模式

## 目标
将审批策略从 `on-request` 改为 `never`，体验自动化程度的差异。

## 操作
编辑 `~/.codex/config.toml`，修改 approval_policy：
```toml
model = "glm-4-flash"
approval_policy = "never"
sandbox_mode = "workspace-write"
```

对比两种模式的行为：
```bash
codex "在当前目录创建一个 hello.txt 文件，内容为 hello world"
```

观察 Codex 是否自动执行了文件写入操作，没有弹出确认提示。

再改回 `on-request` 模式对比：
```toml
approval_policy = "on-request"
```

## 完成标准
`never` 模式下 Codex 自动执行了操作，不需要手动确认。理解四种审批策略的区别：`untrusted` 最安全（只自动执行受信命令）、`on-failure`（已废弃，仅失败时询问）、`on-request` 模型自行判断何时需要确认、`never` 完全无人值守（适合 CI/自动化）。
# 实战任务 2.3: 用 /model 切换模型

## 目标
体验不同模型的能力差异，学会在会话中切换模型。

## 操作
在交互会话中输入 `/model`，浏览可用的模型列表。

然后用命令行参数对比不同模型回答同一个问题：
```bash
codex exec --model o4-mini "用一段话解释什么是 monad"
codex exec --model o3 "用一段话解释什么是 monad"
```

对比响应速度。o4-mini 通常最快，o3 推理更强但更慢。

再试一个代码任务：
```bash
codex exec --model o4-mini "写一个 TypeScript 深拷贝函数"
```

## 完成标准
成功用 `/model` 和 `--model` 参数切换了模型。理解日常简单任务用 o4-mini 足够，复杂推理和调试用 o3。
# 实战任务 2.4: 配置 shell 自动补全

## 目标
为 Codex CLI 配置 shell 自动补全，加速日常使用。

## 操作
根据你的 shell 类型生成补全脚本：

zsh 用户：
```bash
mkdir -p ~/.zfunc
codex completion zsh > ~/.zfunc/_codex
```

在 `~/.zshrc` 中确保有以下配置（如果没有就添加）：
```bash
fpath=(~/.zfunc $fpath)
autoload -U compinit
compinit
```

重新加载 shell 配置：
```bash
source ~/.zshrc
```

测试补全：
```bash
codex --<按 Tab 键>
```

## 完成标准
输入 `codex --` 后按 Tab 键，出现可用的参数列表。理解 shell 补全让你不需要记住所有参数，按 Tab 即可浏览选项。
# 实战任务 2.5: 管道和 codex exec

## 目标
将 Codex 融入 shell 工作流，用管道分析命令输出。

## 操作
用管道把文件内容发给 Codex 分析：
```bash
cat ~/.codex/config.toml | codex exec "explain what each config option does"
```

把 git diff 管道给 Codex 做代码审查：
```bash
git diff | codex exec "summarize these changes in 3 bullet points"
```

尝试将日志输出管道给 Codex 分析：
```bash
ls -la | codex exec "list the 3 largest files and their sizes"
```

## 完成标准
管道命令成功将外部输出传给 Codex 分析。理解 `codex exec` 是非交互执行的核心命令，管道让 Codex 能分析任何命令行工具的输出。
# 实战任务 2.6: 用指令文件定制 Codex 行为

## 目标
通过项目级规则文件给 Codex 加载任务专属规则，对比有无规则的输出差异。

## 操作
创建一个代码审查规则文件：
```bash
cat > /tmp/review-rules.md << 'EOF'
You are a strict code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Missing error handling
Rate each finding as Critical, Warning, or Info.
EOF
```

方式一：把规则文件内容作为 prompt 的一部分传入：
```bash
codex exec "$(cat /tmp/review-rules.md) review the code in the current directory"
```

方式二：通过管道传入规则文件：
```bash
cat /tmp/review-rules.md | codex exec "follow these rules and review the code in the current directory"
```

对比不加规则的输出：
```bash
codex exec "review the code in the current directory"
```

## 完成标准
带规则的输出明显更有结构和针对性，遵循了规则文件中定义的审查要点。理解 Codex 没有独立的 `--instructions` 标志，而是通过管道或命令替换把规则内容注入 prompt，适合按任务加载不同规则，而不是把所有规则写进 AGENTS.md。
# 实战任务 2.7: 配置环境变量

## 目标
将 OPENAI_API_KEY 写入 shell 配置文件，确保 Codex 每次启动都能自动使用。

## 操作
检查当前环境变量是否已设置：
```bash
echo $OPENAI_API_KEY
```

如果为空，将 API key 写入 shell 配置文件：
```bash
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

验证设置成功：
```bash
echo $OPENAI_API_KEY
# 应该输出你的 key
```

启动 Codex 确认能正常工作：
```bash
codex exec "say hello"
```

## 完成标准
`echo $OPENAI_API_KEY` 输出你的 API key。Codex 能正常启动并响应。理解 `OPENAI_API_KEY` 是 Codex 唯一的必需环境变量，`CODEX_HOME` 和 `OPENAI_BASE_URL` 是可选的。
