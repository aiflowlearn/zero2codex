# 实战任务 4.1: 创建 subagent

## 目标
定义一个专门的 code review subagent。

## 操作
创建文件 `.codex/agents/reviewer.toml`：
```bash
mkdir -p .codex/agents
cat > .codex/agents/reviewer.toml << 'EOF'
name = "security-reviewer"
description = "Security-focused code reviewer. Use after writing auth or data handling code."
developer_instructions = """You are a senior security engineer specializing in application security.

Review priorities:
1. Authentication and authorization flaws
2. Injection vulnerabilities (SQL, XSS, command)
3. Data exposure and sensitive information handling

For each finding, provide: severity (Critical/High/Medium/Low), location (file:line), description, and a concrete fix."""
EOF
```

验证：
```bash
cat .codex/agents/reviewer.toml
```

## 完成标准
`.codex/agents/reviewer.toml` 文件存在且格式正确（TOML 格式，包含 name、description、developer_instructions）。理解 subagent 使用 TOML 格式定义（不是 Markdown），有自己的上下文窗口。
# 实战任务 4.2: 用 subagent 做 code review

## 目标
调用自定义 subagent 审查代码。

## 操作
在一个有代码的项目目录中启动 Codex：
```bash
codex
```

在会话中请求使用 reviewer agent：
```
用 security-reviewer agent 审查当前项目中的所有代码
```

Codex 会将审查任务委托给你定义的 security-reviewer subagent。观察它是否按照 developer_instructions 中定义的优先级列表进行审查。

也可以用 `/agent` 命令查看和切换 agent 线程。

## 完成标准
Codex 调用了自定义 subagent，输出包含了安全审查结果（按 Critical/High/Medium/Low 分级）。理解 subagent 的 developer_instructions 控制了它的审查行为和输出格式。
# 实战任务 4.3: 用 /review 审查

## 目标
用内置的 /review 命令审查代码变更。

## 操作
先在项目中做一个小修改（如果之前有 Codex 生成的代码，可以用它）：
```bash
codex "在 hello.ts 中添加一个 greet 函数"
```

然后用 /review 审查变更：
```
/review
```

观察 /review 的输出。它会分析 git diff 并提供改进建议。

再试审查未提交的变更：
```
/review uncommitted
```

## 完成标准
/review 输出了对当前变更的审查结果，包含改进建议。理解 /review 是一个快速获得 AI 代码审查的方式，适合在提交前做最后检查。
# 实战任务 4.4: 用 /fork 分支对话

## 目标
使用 /fork 在已有对话基础上创建分支，尝试不同方案。

## 操作
启动 Codex 并完成一个任务：
```bash
codex
```

在会话中输入一个修改请求：
```
帮我写一个冒泡排序函数
```

Codex 生成排序函数后，用 /fork 创建对话分支：
```
/fork
```

在新分支中尝试不同的方案：
```
把冒泡排序改成快速排序，对比两种实现的性能差异
```

观察原始对话不受影响——/fork 创建了独立的对话线程。

## 完成标准
成功创建了对分支。在新分支中尝试了不同方案，原始对话保持不变。理解 /fork 是安全探索不同方案的方式——不影响已有工作。
# 实战任务 4.5: 用 codex exec 非交互运行

## 目标
用 `codex exec` 实现非交互式代码分析，体验自动化场景的用法。

## 操作
用 codex exec 执行一次性任务：
```bash
codex exec "列出当前项目中所有没有测试的函数"
```

用管道将 git diff 传给 codex exec 做代码审查：
```bash
git diff | codex exec "review these changes and list potential issues"
```

将输出保存到文件：
```bash
codex exec "generate a summary of the project architecture" > architecture.md
```

## 完成标准
`codex exec` 成功执行并返回结果，不进入交互模式。管道用法正常工作。理解 `codex exec` 是 CI/CD、脚本和自动化的核心命令。
