# 实战任务 5.1: 用 Codex CLI 创建 CLI TODO 工具

## 目标
综合运用前 4 个模块学到的知识，用 Codex CLI 从零创建一个可运行的 CLI 工具。

## 操作

### 第一步：创建项目目录

```bash
mkdir -p ~/todo-cli && cd ~/todo-cli
```

### 第二步：启动 Codex CLI 并提出需求

```bash
codex
```

输入以下需求：

```
帮我用 Node.js 创建一个 CLI TODO 工具，保存为当前目录下的 todo.js。要求：
1. 支持 add "任务内容" 添加任务
2. 支持 list 列出所有任务
3. 支持 done <id> 标记完成
4. 支持 delete <id> 删除任务
5. 数据存储在本地 JSON 文件中
6. 不使用任何外部依赖，只用 Node.js 内置模块
```

Codex 会为你创建 `todo.js` 文件。过程中它可能会请求确认，审查它的计划后批准。

### 第三步：退出 Codex CLI

确认 Codex 已经创建好 `todo.js` 后，输入以下命令退出：

```
/quit
```

或者按 `Ctrl + D` 退出。

### 第四步：运行测试

退回到终端后，在 `~/todo-cli` 目录下运行以下命令验证工具是否正常工作：

```bash
node todo.js add "买菜"
node todo.js add "写代码"
node todo.js list
node todo.js done 1
node todo.js list
```

## 完成标准
所有命令正常执行，list 输出包含添加的任务。
# 实战任务 5.2: 用 /review 审查自己的项目

## 目标
用 /review 命令审查上一任务构建的 TODO 工具。

## 操作
进入 TODO 工具项目目录并启动 Codex：
```bash
cd ~/todo-cli
codex
```

用 /review 审查项目：
```
/review
```

观察审查结果。Codex 会分析代码并给出改进建议。

如果发现问题，可以在同一会话中让 Codex 修复：
```
根据审查结果修复代码中的问题
```

也可以用 `codex exec` 做快速审查：
```bash
codex exec "请审查这个项目中所有代码的安全性、代码质量和最佳实践"
```

## 完成标准
/review 输出了完整的审查报告，包含发现的问题和改进建议。这是课程的最后一个实践任务——你已经完整体验了从安装 Codex CLI 到用它构建并审查一个真实项目的全过程。
# 实战任务 5.3: 用 codex exec 添加新功能

## 目标
用 `codex exec` 非交互模式为已有的 TODO 工具添加新功能。

## 操作
在上一任务的 TODO 工具项目中：

```bash
cd ~/todo-cli
```

用 codex exec 添加优先级功能：
```bash
codex exec "给 todo.js 添加优先级功能：add 命令支持 --priority high/medium/low（默认 medium），list 命令显示优先级。保持现有功能不变。"
```

验证新功能：
```bash
node todo.js add "紧急会议" --priority high
node todo.js list
```

再用 codex exec 添加截止日期功能：
```bash
codex exec "给 todo.js 添加截止日期功能：add 命令支持 --due YYYY-MM-DD，list 命令显示截止日期。"
```

验证：
```bash
node todo.js add "提交报告" --priority high --due 2026-05-30
node todo.js list
```

## 完成标准
新功能正常工作，list 输出显示优先级和截止日期。理解 `codex exec` 也可以用于迭代开发——先让 Codex 创建基础功能，再用 exec 逐步添加新功能。
