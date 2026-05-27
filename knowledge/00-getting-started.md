# Getting Started with Codex CLI

## What is Codex CLI?

OpenAI Codex CLI is a **terminal-based AI coding assistant** that runs locally on your machine. You describe what you want in natural language, and Codex reads your code, suggests edits, writes new features, runs commands — all by directly understanding your project on disk. Unlike web-based chat tools, Codex runs in your local environment with direct file system access and command execution capability.

## Prerequisites

- **Node.js 22** or higher
- **macOS**, **Linux**, or **Windows** (via WSL)
- Active network connection
- An **OpenAI API key** (from platform.openai.com) or a **ChatGPT Plus/Pro/Team/Enterprise** account

API usage incurs costs based on the model and volume selected.

## Installation

Install globally via npm (recommended):

```bash
npm install -g @openai/codex
```

On macOS, Homebrew is also available:

```bash
brew install openai-codex
```

Alternatively, download pre-built binaries from the GitHub releases page.

Verify the installation:

```bash
codex --version
```

## Authentication

Set the OpenAI API key as an environment variable:

```bash
export OPENAI_API_KEY="sk-..."
codex
```

If you have a ChatGPT account, use interactive login:

```bash
codex --login
```

## Your First Session

Navigate to any project directory and start Codex:

```bash
cd my-project
codex
```

You will see a welcome message and a prompt. Type a natural language request:

```
Please explain the directory structure of this project.
```

Codex reads your files, analyzes the structure, and returns a summary. You can also start with a prompt directly:

```bash
codex "explain this project"
```

When Codex needs to edit files or run commands, its behavior depends on the **approval policy** — in the default Suggest mode, it proposes changes but does not execute them automatically.

## Key Terminology

| Term | Definition |
|------|-----------|
| **Terminal** | A text-based window for entering commands (Terminal.app on macOS, Windows Terminal on Windows) |
| **CLI** | Command Line Interface — a program controlled by typing commands |
| **API key** | A secret token used to authenticate with the OpenAI API |
| **Sandbox** | An isolated execution environment that prevents unintended system changes |
| **IDE** | Integrated Development Environment — a code editor with built-in tools (VS Code, IntelliJ, PyCharm) |
| **Approval policy** | Controls what Codex can do without asking for confirmation |
| **AGENTS.md** | A project-level instruction file that Codex reads at session start |
