# CLI Coding Agent 对比

本文对比 Blade Code 与主流 CLI Coding Agent，帮助你了解各工具的特点和适用场景。

## 产品概览

| 产品 | 开发者 | 开源协议 | 默认模型 | 定位 |
|------|--------|----------|----------|------|
| **Blade Code** | 社区 | MIT | 多模型支持 | 多模型、可扩展的 AI 编码助手 |
| **Claude Code** | Anthropic | 闭源 | Claude | Anthropic 官方 CLI 工具 |
| **Gemini CLI** | Google | Apache 2.0 | Gemini 2.5 Pro | Google 官方终端 AI Agent |
| **Codex CLI** | OpenAI | Apache 2.0 | GPT-4o | OpenAI 官方编码 Agent |
| **OpenCode** | Anomaly | MIT | 多模型支持 | 开源 AI 编码 Agent |

## 核心特性对比

### 模型支持

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **多模型支持** | ✅ 任意 OpenAI 兼容 API | ❌ 仅 Claude | ❌ 仅 Gemini | ❌ 仅 OpenAI | ✅ 多模型 |
| **国产模型** | ✅ Qwen/DeepSeek 等 | ❌ | ❌ | ❌ | ✅ |
| **本地模型** | ✅ Ollama 等 | ❌ | ❌ | ❌ | ✅ |
| **免费额度** | 取决于模型提供商 | 有限 | 60 次/分钟，1000 次/天 | 取决于套餐 | 取决于模型提供商 |
| **上下文窗口** | 取决于模型 | 200K | 1M | 取决于模型 | 取决于模型 |

**Blade Code 优势**：不绑定任何模型提供商，支持国产模型（Qwen、DeepSeek）和本地模型（Ollama），用户可根据成本和性能自由选择。

### 工作模式

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **Plan 模式** | ✅ 只读调研 | ✅ | ❌ | ❌ | ✅ |
| **Spec 模式** | ✅ 六阶段工作流 | ❌ | ❌ | ❌ | ❌ |
| **自动编辑模式** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **非交互模式** | ✅ `--print` | ✅ `-p` | ✅ `-p` | ✅ | ✅ |
| **会话恢复** | ✅ `--resume` | ✅ | ✅ | ✅ | ✅ |

**Blade Code 独有**：
- **Spec 模式**：结构化的六阶段开发工作流（Init → Requirements → Design → Tasks → Implementation → Done），适合复杂功能开发
- **Plan 模式**：双重保护机制（系统提示 + 工具级限制），确保调研阶段不会意外修改代码

### 权限控制

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **权限模式** | 4 种 | 3 种 | 有限 | 有限 | 2 种 |
| **细粒度规则** | ✅ allow/ask/deny | ✅ | ❌ | ❌ | ❌ |
| **工具级限制** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **轮次上限** | ✅ 可配置 | ✅ | ❌ | ❌ | ❌ |
| **危险命令拦截** | ✅ 内置黑名单 | ✅ | ✅ | ✅ | ✅ |

**Blade Code 权限模式**：
- `default`：写操作需确认
- `autoEdit`：文件编辑自动通过，命令执行需确认
- `plan`：仅允许只读工具
- `yolo`：全自动（需自担风险）

### 扩展能力

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **MCP 支持** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Skills 系统** | ✅ 兼容 Claude | ✅ | ✅ 独立 Skills | ✅ 兼容 Claude | ❌ |
| **Subagents** | ✅ 可定制 | ✅ | ❌ | ❌ | ✅ |
| **Hooks 系统** | ✅ Pre/Post 钩子 | ✅ | ❌ | ❌ | ❌ |
| **自定义 Slash 命令** | ✅ | ✅ | ❌ | ❌ | ❌ |

**Blade Code 扩展特性**：
- **Skills**：封装常用工作流为可复用模块，兼容 Claude Code Skills 格式
- **Subagents**：可定制的子代理，支持工具权限限制
- **Hooks**：工具执行前后的钩子，支持自动格式化、lint 等

#### Skills 系统

| 工具 | 支持 | 兼容性 |
|------|------|--------|
| Blade Code | ✅ | 兼容 Claude Code Skills |
| Claude Code | ✅ | 原生支持 |
| Gemini CLI | ✅ | 独立 Skills 系统 (`SKILL.md`) |
| Codex CLI | ✅ | 兼容 Claude Code Skills (`~/.codex/skills`) |
| OpenCode | ❌ | 无 |

Blade Code 和 Codex CLI 的 Skills 系统都兼容 Claude Code，可以直接使用 [Anthropic 官方 Skills 仓库](https://github.com/anthropics/skills)。Gemini CLI 也有类似的 Skills 机制，通过 `SKILL.md` 定义，但格式与 Claude Code 略有不同。

### IDE 集成

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **ACP 协议** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **VS Code** | ✅ 通过 ACP | ✅ 官方插件 | ❌ | ✅ 官方插件 | ❌ |
| **JetBrains** | ✅ 通过 ACP | ❌ | ❌ | ❌ | ❌ |
| **Neovim** | ✅ 通过 ACP | ❌ | ❌ | ❌ | ❌ |
| **Zed** | ✅ 通过 ACP | ❌ | ❌ | ❌ | ❌ |

**Blade Code 优势**：通过 ACP（Agent Client Protocol）支持多种 IDE，一次配置多处使用。

### 内置工具

| 工具类型 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|----------|------------|-------------|------------|-----------|----------|
| **文件读写** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **代码搜索** | ✅ Glob/Grep | ✅ | ✅ | ✅ | ✅ |
| **Shell 执行** | ✅ 后台支持 | ✅ | ✅ | ✅ | ✅ |
| **Web 搜索** | ✅ 多引擎聚合 | ✅ | ✅ Google | ❌ | ✅ |
| **Web 抓取** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **用户交互** | ✅ AskUserQuestion | ✅ | ❌ | ❌ | ❌ |
| **TODO 管理** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Notebook 编辑** | ✅ | ✅ | ❌ | ❌ | ❌ |

### 配置管理

| 特性 | Blade Code | Claude Code | Gemini CLI | Codex CLI | OpenCode |
|------|------------|-------------|------------|-----------|----------|
| **双文件配置** | ✅ config + settings | ✅ | ❌ | ❌ | ✅ |
| **项目级配置** | ✅ `.blade/` | ✅ `.claude/` | ✅ `GEMINI.md` | ❌ | ✅ |
| **环境变量插值** | ✅ `${VAR}` | ✅ | ✅ | ✅ | ✅ |
| **配置向导** | ✅ 首次启动 | ❌ | ✅ | ✅ | ❌ |
| **settings.local.json** | ✅ 自动 gitignore | ✅ | ❌ | ❌ | ❌ |

## 安装方式对比

### Blade Code

```bash
npm install -g blade-code
# 或零安装体验
npx blade-code
```

### Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### Gemini CLI

```bash
npm install -g @google/gemini-cli
# 或 Homebrew
brew install gemini-cli
# 或零安装
npx https://github.com/google-gemini/gemini-cli
```

### Codex CLI

```bash
npm install -g @openai/codex
# 或 Homebrew
brew install --cask codex
```

### OpenCode

```bash
npm install -g opencode-ai
# 或 Homebrew
brew install opencode
# 或一键安装
curl -fsSL https://opencode.ai/install | bash
```

## 认证方式对比

| 产品 | 认证方式 |
|------|----------|
| **Blade Code** | API Key（支持环境变量插值）、OAuth（Copilot/Antigravity） |
| **Claude Code** | Anthropic API Key |
| **Gemini CLI** | Google OAuth、API Key、Vertex AI |
| **Codex CLI** | ChatGPT 登录、OpenAI API Key |
| **OpenCode** | 各模型提供商 API Key |

## 适用场景推荐

### 选择 Blade Code 如果你：
- 需要使用国产模型（Qwen、DeepSeek）或本地模型
- 希望在不同模型间灵活切换
- 需要结构化的开发工作流（Spec 模式）
- 使用 Zed、JetBrains、Neovim 等 IDE
- 需要与 Claude Code 生态兼容（Skills、配置格式）

### 选择 Claude Code 如果你：
- 主要使用 Claude 模型
- 需要官方支持和稳定性
- 已有 Claude Code Skills 资产

### 选择 Gemini CLI 如果你：
- 需要大上下文窗口（1M tokens）
- 希望使用 Google 免费额度
- 需要 Google Search 实时信息

### 选择 Codex CLI 如果你：
- 已有 ChatGPT Plus/Pro 订阅
- 主要使用 OpenAI 模型
- 需要官方 VS Code 集成

### 选择 OpenCode 如果你：
- 偏好 TUI 界面
- 需要桌面应用
- 希望使用完全开源方案

## 参考资源

- [Blade Code GitHub](https://github.com/echoVic/blade-code)
- [Claude Code 文档](https://docs.anthropic.com/claude-code)
- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Codex CLI GitHub](https://github.com/openai/codex)
- [OpenCode GitHub](https://github.com/anomalyco/opencode)
