---
title: Claude Code的安装和使用~
published: 2026-06-12
description: 让我们学习Claude Code的安装与使用吧！
image:<img src="https://raw.githubusercontent.com/JuMoXia/JuMoXia-new-blog-images/master/img/d9b0f20e25018b4a8eb3a4800bb08d8d.jpg"/>
tags: [开发]
category: 开发
draft: false
comment: true
---

# Claude Code 安装与使用指南

## 1. 什么是 Claude Code

**Claude Code** 是 Anthropic 推出的一款命令行 AI 编程助手（CLI Agent），能够直接在终端中理解你的代码库、执行操作、编写代码，并与 Git、CI/CD 等工具无缝集成。

简单理解：Claude Code 就像一个驻扎在你终端里的资深程序员，能阅读整个项目、回答代码问题、直接帮你写代码、改 bug、跑测试。

### 1.1 核心优势

| 优势 | 说明 |
| :--- | :--- |
| **上下文强大** | 支持 200K Token 上下文窗口，可一次性理解大型项目 |
| **Agent 模式** | 不仅能回答问题，还能自主执行多步骤任务 |
| **工具集成** | 内置文件读写、Git、Shell 命令执行等工具 |
| **IDE 支持** | 支持 VS Code、JetBrains 等主流 IDE |
| **多模型** | 可灵活切换 Opus / Sonnet / Haiku 模型 |

---

## 2. 系统要求

| 条件 | 说明 |
| :--- | :--- |
| **操作系统** | macOS 10.15+ / Ubuntu 20.04+ / Debian 10+ / Windows 10/11（需 WSL 或 Git Bash） |
| **Node.js** | v18.0 或更高版本 |
| **Git** | 2.23+（Windows 下必须先安装 Git for Windows） |
| **内存** | 4GB 以上，推荐 8GB+ |

---

## 3. 安装方法

### 3.1 方法一：官方原生一键安装（⭐推荐）

最简单的方式，无需预装 Node.js，脚本自动处理所有依赖。

**macOS / Linux / WSL：**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell：**

```powershell
irm https://claude.ai/install.ps1 | iex
```

### 3.2 方法二：Homebrew 安装（macOS）

```bash
brew install --cask claude-code
```

### 3.3 方法三：WinGet 安装（Windows）

```powershell
winget install Anthropic.ClaudeCode
```

### 3.4 方法四：NPM 安装（传统通用方式）

```bash
# 全局安装
npm install -g @anthropic-ai/claude-code

# 验证安装
claude --version
```

### 3.5 方法五：Docker 容器化安装

```bash
curl -O https://raw.githubusercontent.com/RchGrav/claudebox/main/claudebox
chmod +x claudebox
```

> **验证安装**：安装完成后运行 `claude --version`，输出版本号即表示安装成功。

---

## 4. 基础配置

### 4.1 配置 API Key

**方式 A：OAuth 登录（有 Pro / Max 订阅）**

```bash
claude   # 按提示完成浏览器登录
```

**方式 B：API Key（控制台获取）**

```bash
# Linux / macOS
export ANTHROPIC_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Windows PowerShell
$env:ANTHROPIC_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

**持久化配置（推荐写入 Shell 配置文件）：**

```bash
# Linux / macOS 写入 ~/.zshrc 或 ~/.bashrc
echo 'export ANTHROPIC_API_KEY="sk-xxxxxxxx"' >> ~/.zshrc
source ~/.zshrc
```

### 4.2 首次启动

```bash
cd your-project    # 进入你的项目目录
claude             # 启动交互模式
```

首次运行会提示：选择主题、确认安全须知、信任当前工作目录等，按照提示操作即可。

---

## 5. 接入 DeepSeek V4

DeepSeek 官方提供了与 Anthropic 兼容的 API 端点，可以无缝替换后端模型，费用仅为 Anthropic 官方的约 **5%**。

### 5.1 获取 DeepSeek API Key

前往 [platform.deepseek.com](https://platform.deepseek.com) 注册账号并创建 API Key。

### 5.2 方案一：环境变量配置（推荐，最简单）

**Linux / macOS：**

```bash
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=你的DeepSeek_API_Key
export ANTHROPIC_MODEL=deepseek-v4-pro[1m]
export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro[1m]
export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro[1m]
export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash
export CLAUDE_CODE_EFFORT_LEVEL=max
```

**Windows PowerShell：**

```powershell
$env:ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
$env:ANTHROPIC_AUTH_TOKEN="你的DeepSeek_API_Key"
$env:ANTHROPIC_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-v4-pro[1m]"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL="deepseek-v4-flash"
$env:CLAUDE_CODE_SUBAGENT_MODEL="deepseek-v4-flash"
$env:CLAUDE_CODE_EFFORT_LEVEL="max"
```

### 5.3 方案二：settings.json 配置（持久化）

编辑 `~/.claude/settings.json`（Windows 为 `C:\Users\<用户名>\.claude\settings.json`）：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的DeepSeek_API_Key",
    "ANTHROPIC_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max"
  }
}
```

### 5.4 方案三：claudeep 一键配置工具

[npm 包 claudeep](https://www.npmjs.com/package/claudeep) 提供自动化配置脚本。

```bash
# 安装
npm install -g claudeep@latest

# 配置
claudeep setup 你的API_KEY

# 健康检查
claudeep doctor

# 自动修复
claudeep doctor --fix
```

### 5.5 模型对照表

| 用途 | DeepSeek 模型 | 说明 |
| :--- | :--- | :--- |
| **主力推理** | `deepseek-v4-pro[1m]` | 对应 Opus/Sonnet，强大推理能力 |
| **快速任务** | `deepseek-v4-flash` | 对应 Haiku，轻量快速 |
| **子代理** | `deepseek-v4-flash` | 子任务执行，节省成本 |

> **注意**：Claude Code 使用模型白名单机制，不能直接用 `--model` 指定 DeepSeek 模型名，必须通过环境变量映射。配置完成后可在 Claude Code 中问"你是什么模型"验证是否接入成功。

---

## 6. 核心使用方式

### 6.1 四种使用模式

| 模式 | 命令 | 说明 |
| :--- | :--- | :--- |
| **交互模式** | `claude` | 启动对话式 REPL，每次修改需确认 |
| **一次性查询** | `claude -p "问题"` | 快速查询并输出结果，不进入交互 |
| **管道模式** | `cat logs.txt \| claude -p "分析这些日志"` | 将管道内容作为上下文处理 |
| **继续会话** | `claude -c` | 恢复上次的对话历史 |

### 6.2 高级启动参数

```bash
# 自动执行模式（跳过确认，适合长时间任务）
claude --dangerously-skip-permissions

# 指定配置文件
claude --settings ~/.claude/settings.custom.json

# 查看帮助
claude --help
```

---

## 7. 交互模式常用命令

| 命令 | 功能 |
| :--- | :--- |
| `/init` | 扫描项目，生成 CLAUDE.md 配置文件 |
| `/clear` | 清除对话历史 |
| `/compact` | 压缩上下文，节省 Token |
| `/config` | 配置主题、模型等参数 |
| `/cost` | 查看 Token 消耗和预估费用 |
| `/model` | 切换模型（Opus / Sonnet / Haiku） |
| `/doctor` | 运行系统健康检查 |
| `/mcp` | 管理 MCP 服务器 |
| `/memory` | 编辑记忆和自定义规则 |
| `/plan` | 进入规划模式（先设计方案再编写代码） |
| `@文件名` | 引用文件到上下文中 |
| `!命令` | 执行终端命令（如 `!npm test`） |
| `Esc 双击` | 回退对话 |
| `Ctrl + C` | 取消当前操作 |
| `Shift + Tab` | 切换自动接受编辑模式 |

---

## 8. IDE 集成

### 8.1 VS Code

在 VS Code 扩展市场搜索 **"Claude Code"** 安装官方扩展。安装后在左侧活动栏会出现 Claude Code 图标，点击即可在侧边栏中使用。

功能亮点：
- 内联 diff 查看，编辑前后对比一目了然
- 支持 `@文件` 引用项目文件
- 与 VS Code 文件浏览器联动

### 8.2 JetBrains

在 JetBrains 插件市场（Settings → Plugins）搜索 `Claude Code` 安装（当前为 Beta 版本）。

支持 IntelliJ IDEA、WebStorm、PyCharm、GoLand 等主流 JetBrains IDE。

### 8.3 快捷键

| 平台 | 快捷键 | 功能 |
| :--- | :--- | :--- |
| Mac | `Cmd + Esc` | 快速启动 Claude Code |
| Windows | `Ctrl + Esc` | 快速启动 Claude Code |

---

## 9. MCP 服务器配置

MCP（Model Context Protocol）允许 Claude Code 连接外部工具和数据源。

### 9.1 配置示例

编辑 `~/.claude/.mcp.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-github"]
    }
  }
}
```

### 9.2 管理 MCP 服务器

在交互模式中使用 `/mcp` 命令即可查看、添加、删除 MCP 服务器。

---

## 10. 实战技巧

### 10.1 项目初始化

```bash
cd your-project
claude            # 启动 Claude Code
/init             # 让 Claude 扫描项目生成 CLAUDE.md
```

CLAUDE.md 是项目的"说明书"，Claude 每次对话都会读取它来理解项目结构、编码规范和约定。

### 10.2 高效提问技巧

| 技巧 | 示例 |
| :--- | :--- |
| **具体描述需求** | "在 user.py 中为 User 模型添加邮箱验证方法" |
| **引用文件** | `@src/utils/auth.ts` 帮我重构这个文件的认证逻辑 |
| **分步执行** | 先"分析当前代码结构"，再"给出优化方案"，最后"实施修改" |
| **使用 Plan 模式** | `/plan` 先让 Claude 设计方案，确认后再执行 |

### 10.3 成本控制

```bash
# 随时查看消耗
/cost

# 切换更便宜的模型处理简单任务
/model haiku

# 压缩上下文避免过长对话
/compact
```

| 模型 | 费用对比 | 适用场景 |
| :--- | :--- | :--- |
| DeepSeek V4 Pro | ~$7/月 | 主力编程，高性价比 |
| DeepSeek V4 Flash | ~$2/月 | 简单任务、子代理 |
| Anthropic Opus 官方 | ~$200/月 | 最强性能，企业级 |

### 10.4 常见问题排查

| 问题 | 解决方案 |
| :--- | :--- |
| **无法连接 API** | 检查 `ANTHROPIC_BASE_URL` 设置是否正确，确认网络能访问 DeepSeek API |
| **模型不识别** | 确认使用环境变量映射而非 `--model` 参数 |
| **权限不足** | 首次使用需信任工作目录，按提示确认 |
| **命令未找到** | 确认 Node.js ≥ 18，npm 全局安装路径在 PATH 中 |
| **中文乱码** | Windows 终端请使用 Windows Terminal 而非传统 cmd |

### 10.5 安全配置建议

```bash
# 限定文件访问范围（在 CLAUDE.md 中声明）
# 不要用 root 权限运行
# 定期检查 ~/.claude/ 下的日志和配置
# 生产环境慎用 --dangerously-skip-permissions
```

---

## 11. 总结

Claude Code 的学习路线可概括为：

**安装环境准备 → 选择安装方式 → 配置 API Key →（可选）接入 DeepSeek V4 → 项目初始化 /init → 交互模式入门 → 掌握常用命令 → IDE 集成 → MCP 扩展 → 实战优化**

接入 DeepSeek V4 后，你可以用不到官方 1/20 的成本获得强大的 AI 编程助手，强烈推荐尝试！
