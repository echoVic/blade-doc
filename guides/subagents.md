# Subagents 系统

Subagent 是可定制的子代理，Task 工具会根据配置选择合适的子代理执行子任务。

## 内置 Subagents

Blade 内置了 4 个核心 Subagent，开箱即用：

| 名称 | 工具集 | 用途 |
| --- | --- | --- |
| `general-purpose` | 所有工具 | 通用代理，用于研究复杂问题、搜索代码和执行多步骤任务 |
| `Explore` | Glob, Grep, Read, WebFetch, WebSearch | 快速探索代码库，支持按模式查找文件、搜索关键词、回答代码库相关问题 |
| `Plan` | 所有工具 | 软件架构师代理，用于设计实现计划、分析需求、识别关键文件和依赖 |
| `statusline-setup` | Read, Edit | 配置 Claude Code 状态栏设置 |

### Explore 代理

Explore 代理支持三种彻底程度级别：
- **quick**：1-2 次 Glob 搜索，仅读取关键文件
- **medium**：多种 Glob 模式，Grep 搜索关键词，读取 5-10 个文件
- **very thorough**：穷尽搜索，所有模式，读取所有相关文件

调用示例：`"请使用 Explore 代理以 very thorough 级别搜索所有 API 端点"`

## 加载位置与优先级

`SubagentRegistry` 按以下顺序加载（后加载的会覆盖前面的）：

1. 内置配置（`builtinAgents.ts`）
2. `~/.blade/agents/*.md`（用户级，可覆盖内置）
3. `<project>/.blade/agents/*.md`（项目级，优先级最高）

每个文件需包含 YAML frontmatter 与正文，frontmatter 描述元数据，正文作为系统提示词。

```markdown
---
name: code-reviewer
description: Review code for bugs and risks. Use this when you need critical feedback.
tools:
  - Read
  - Grep
  - Glob
color: blue   # 可选：red/blue/green/yellow/purple/orange/pink/cyan
---

# 这里是系统提示词
你是一名代码审查专家...
```

字段说明：
- `name`：唯一标识（kebab-case）。
- `description`：简要用途，建议包含 “Use this when …” 场景。
- `tools`：允许的工具列表，留空则不做限制。
- `color`：UI 标记颜色（可选）。

## 使用方式

- **调用**：在对话中提出需求并指名 subagent，或让 LLM 自行选择；Task 工具会在 Plan/默认循环中按权限模式执行。
- **管理**：输入 `/agents` 打开管理器；`/agents list` 文本列出所有 subagents；`/agents create` 打开创建向导；`/agents help` 查看格式说明。
- **查看可用类型**：在系统提示中会列出已加载的 subagents 及其工具集（见 `getDescriptionsForPrompt`）。

## 注意事项

- Subagent 是无状态的，每次调用使用新的上下文。
- 工具权限仍受主会话的权限模式约束（Plan 模式下 Task 仍被视为只读）。
- 修改或新增配置后需重启 Blade 或再次进入 UI 以重新加载。
