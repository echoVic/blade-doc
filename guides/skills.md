# Skills 系统

Skills 是 Blade 的一种动态能力扩展机制，允许用户将常用的工作流、专业知识或复杂指令封装成可复用的模块。

Blade 的 Skills 系统与 [Claude Code Skills](https://code.claude.com/docs/en/skills) 高度兼容，同时提供了一些增强特性。

## 什么是 Skills？

Skills 本质上是结构化的 Prompt 和指令集合。它们被存储为文件（`SKILL.md`），Blade 会在启动时自动发现这些文件。

- **AI 自动调用**：Blade 的 AI 模型会根据 Skill 的 `description` 自动判断何时调用某个 Skill。
- **用户手动调用**：你可以通过 Slash 命令（如 `/skill-name`）直接触发特定的 Skill。
- **权限控制**：你可以为每个 Skill 指定允许使用的工具（如只读权限），提高安全性。

## 快速开始

### 查看可用 Skills

使用 `/skills` 命令可以打开 Skills 管理面板，查看当前环境下的所有可用 Skills。

```bash
/skills
```

### 创建新 Skill

Blade 内置了一个 `skill-creator` Skill，可以引导你交互式地创建新 Skill。

```bash
/skill-creator
```

或者你可以手动创建一个 Skill。

## Skill 结构

一个 Skill 通常是一个包含 `SKILL.md` 文件的目录。

```
my-skill/
├── SKILL.md          # 核心定义文件 (必须)
├── scripts/          # 辅助脚本 (可选)
│   └── helper.py
└── templates/        # 模板文件 (可选)
    └── template.txt
```

### SKILL.md 格式

`SKILL.md` 由 YAML Frontmatter（元数据）和 Markdown 内容（指令）组成。

```yaml
---
name: code-review                # Skill 名称 (kebab-case)
description: Review code for best practices. Use when reviewing PRs. # AI 触发依据
allowed-tools: [Read, Grep]      # 允许使用的工具
user-invocable: true             # 是否允许用户通过 /code-review 调用
argument-hint: <file_path>       # 参数提示
---

# Code Review Guide

## Instructions
1. Read the provided file.
2. Check for:
   - Logic errors
   - Security vulnerabilities
   - Style guide violations
3. Output a summary report.
```

### 元数据字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | Skill 的唯一标识符，只能包含小写字母、数字和连字符（如 `my-skill`）。 |
| `description` | 是 | **关键字段**。描述 Skill 的功能以及**何时使用**。AI 依靠此字段决定是否调用该 Skill。 |
| `allowed-tools` | 否 | 允许该 Skill 使用的工具列表。如果不指定，则遵循默认权限。推荐限制为最小可用集。 |
| `user-invocable` | 否 | 如果设为 `true`，用户可以通过 `/<name>` 直接调用该 Skill。 |
| `disable-model-invocation` | 否 | 如果设为 `true`，AI 将不会自动调用此 Skill（仅限用户手动调用）。 |
| `argument-hint` | 否 | 手动调用时的参数提示，如 `<file_path>`。 |
| `version` | 否 | 版本号。 |

## 存放位置与优先级

Blade 会自动扫描以下目录中的 Skills。如果存在同名 Skill，优先级高的会覆盖优先级低的。

| 优先级 | 范围 | 路径 | 说明 |
|--------|------|------|------|
| 1 (最高) | 项目级 | `.blade/skills/` | 当前项目的专用 Skills，可随 Git 共享给团队。 |
| 2 | 项目级 (兼容) | `.claude/skills/` | 兼容 Claude Code 的项目级 Skills。 |
| 3 | 用户级 | `~/.blade/skills/` | 你的个人全局 Skills，在任何项目中均可用。 |
| 4 | 用户级 (兼容) | `~/.claude/skills/` | 兼容 Claude Code 的用户级 Skills。 |
| 5 (最低) | 内置 | (Built-in) | Blade 内置的基础 Skills（如 `skill-creator`）。 |

## 最佳实践

1. **描述要清晰**：`description` 应该包含"做什么"（What）和"何时用"（When）。
   - *Bad*: "Code helper."
   - *Good*: "Generate commit messages following conventional commits format. Use when the user wants to commit changes."
2. **最小权限原则**：尽可能通过 `allowed-tools` 限制 Skill 的权限。例如，一个只读的分析 Skill 不需要 `Write` 或 `Bash` 权限。
3. **提供示例**：在 Markdown 内容中提供 Examples，帮助 AI 理解如何更好地执行任务。
4. **共享给团队**：将通用的团队规范（如代码审查标准、部署流程）封装为项目级 Skills，提交到 Git 仓库，确保团队成员拥有一致的 AI 辅助能力。

## 兼容性

Blade 的 Skills 系统设计为兼容 Claude Code。你可以直接复用现有的 Claude Code Skills，只需将它们放在标准的 `.claude/skills` 目录下，或者迁移到 `.blade/skills` 目录。
