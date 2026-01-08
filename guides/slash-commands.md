# Slash 命令

Slash 命令由 `src/slash-commands` 提供，输入 `/` 触发建议，`Tab` 补全，`Enter` 执行。无论命令成功与否都会显示结果或错误提示。

## 可用命令（当前实现）

| 命令 | 作用 | 备注 |
| --- | --- | --- |
| `/help` | 列出所有 Slash 命令 | 内置帮助。 |
| `/clear` | 清空消息区 | 同 `Ctrl+L` 清屏逻辑。 |
| `/exit` | 退出程序 | 同 `Ctrl+C`。 |
| `/version` | 显示版本信息 | 从 `package.json` 读取。 |
| `/status` | 显示当前项目/配置概览 | 检查 `BLADE.md`、项目类型等。 |
| `/context` / `/cost` | 显示上下文/成本示例信息 | 当前为静态示例，便于占位。 |
| `/init` | 分析项目生成或改进 `BLADE.md` | 调用 Agent 读取代码并写入 `BLADE.md`。 |
| `/model [add|remove]` | 管理模型配置 | 无参时弹出模型选择器；`add` 打开向导；`remove <name>` 删除匹配名称。 |
| `/theme` | 打开主题选择器 | 立即应用并写回配置。 |
| `/permissions` | 打开权限管理器 | 操作 `.blade/settings.local.json`。 |
| `/mcp` | 显示 MCP 状态 | 由内置命令输出当前配置。 |
| `/agents [list|create|help]` | 管理子代理配置 | 读取 `.blade/agents` / `~/.blade/agents`；`list` 文本列出，`create` 打开创建向导。 |
| `/resume` | 打开会话选择器 | 可恢复历史对话。 |
| `/compact` | 手动触发上下文压缩 | 调用 UI 压缩逻辑。 |
| `/tasks [clean]` | 列出所有后台任务（shells 和 agents） | 显示后台 shell 和 agent 的运行状态；`clean` 清理已完成的会话。 |
| `/skills` | 打开 Skills 管理面板 | 查看和管理所有可用 Skills。 |
| `/skill-creator` | 交互式创建新 Skill | 引导用户生成 SKILL.md。 |
| `/plugins [list|info|install|...]` | 管理插件系统 | 安装、卸载、启用、禁用插件。 |
| `/git [status|log|diff|review|commit]` | Git 操作集成。`commit` (或 pre-commit) 会分析暂存区变动，调用 AI 生成符合历史风格的 commit message。 | 需在 Git 仓库中使用。 |

## 自定义 Slash 命令

Blade 支持创建自定义 Slash 命令，将常用的 Prompt 模板或复杂任务封装为简单的命令。该系统与 Claude Code 的自定义命令兼容。

### 目录结构与优先级

Blade 会按以下顺序（优先级从低到高）扫描目录中的 Markdown 文件：

1. `~/.blade/commands/` (用户级)
2. `~/.claude/commands/` (用户级，兼容 Claude Code)
3. `.blade/commands/` (项目级)
4. `.claude/commands/` (项目级，兼容 Claude Code)

项目级命令会覆盖同名的用户级命令。

### 创建自定义命令

创建一个 Markdown 文件（例如 `my-command.md`），文件名即为命令名（不含 `/`）。

**文件格式：**

```markdown
---
description: 命令的简要描述 (用于 /help 和 AI 调用)
argument-hint: "[参数提示]"
---

这里是命令的内容。支持以下功能：

1. **参数插值**：使用 `$1`, `$2` 等占位符引用用户输入的参数。
2. **Bash 嵌入**：使用反引号执行 shell 命令。

示例：
请帮我重构 `$1` 文件。
下面是相关的测试结果：
\`npm test $1\`
```

### 配置选项 (Frontmatter)

| 字段 | 说明 |
|------|------|
| `description` | **必需**。用于在 `/help` 中显示，以及帮助 AI 理解何时使用该命令。 |
| `argument-hint` | 在输入命令时显示的参数提示文本。 |
| `allowed-tools` | 允许该命令使用的工具列表。 |
| `model` | 指定执行该命令的模型。 |
| `disable-model-invocation` | 设为 `true` 时，禁止 AI 自动调用此命令（仅限用户手动使用）。 |

## 补全与导航

- 输入 `/` 自动展示建议；继续输入可模糊匹配（Fuse.js）。
- `Tab` 选中当前高亮项；`↑/↓` 在建议列表中移动。
- 输入包含空格后不再展示命令建议，避免干扰子参数输入。
- **Skills 支持**：如果某个 Skill 配置了 `user-invocable: true`，它也会出现在命令列表中（如 `/code-review`）。

## 典型用法

```bash
/init                     # 生成或改进 BLADE.md
/git status               # 查看 Git 状态
/git review               # 让 Agent 生成变更点评
/model add                # 新增模型配置
/permissions              # 编辑本地权限规则
/theme                    # 切换主题
/resume                   # 选择历史会话
/tasks                    # 查看后台任务（shells 和 agents）
/tasks clean              # 清理已完成的后台任务
/plugins                  # 打开插件管理界面
/plugins list             # 列出所有已加载的插件
/plugins install user/repo # 从 GitHub 安装插件
```

## 注意事项

- 部分命令（如 `/config`）当前为占位信息，仅展示示例文本。
- Slash 命令执行前会确保配置 Store 已初始化；如配置缺失会在对话区提示错误。
- `/git` 系列依赖本地 Git 状态，遇到缺失或异常会返回错误消息而非中断程序。
