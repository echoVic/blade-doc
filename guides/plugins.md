# 插件系统

Blade Code 提供了一个强大的插件系统，允许你通过插件扩展功能，包括自定义命令、代理、技能、Hooks 和 MCP 服务器。

## 概述

插件是包含特定目录结构的文件夹，可以提供以下扩展：

- **命令** - 自定义 Slash 命令
- **代理** - 自定义 Subagents
- **技能** - 自定义 Skills
- **Hooks** - 生命周期钩子
- **MCP 服务器** - Model Context Protocol 服务器配置

## 插件目录结构

一个标准的插件目录结构如下：

```
my-plugin/
├── .blade-plugin/          # 或 .claude-plugin/
│   └── plugin.json         # 插件清单文件（必需）
├── commands/               # Slash 命令
│   └── my-command.md
├── agents/                 # 自定义代理
│   └── my-agent.md
├── skills/                 # 技能
│   └── my-skill/
│       └── SKILL.md
├── hooks/                  # Hooks 配置
│   └── hooks.json
└── .mcp.json              # MCP 服务器配置
```

## 插件清单 (plugin.json)

每个插件必须包含一个清单文件，位于 `.blade-plugin/plugin.json` 或 `.claude-plugin/plugin.json`：

```json
{
  "name": "my-plugin",
  "description": "我的自定义插件",
  "version": "1.0.0",
  "author": {
    "name": "Your Name",
    "email": "your@email.com",
    "url": "https://your-website.com"
  },
  "license": "MIT",
  "repository": "https://github.com/user/my-plugin",
  "keywords": ["blade", "plugin"],
  "bladeVersion": ">=0.0.47"
}
```

### 清单字段说明

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 插件唯一标识符，kebab-case 格式，2-64 字符 |
| `description` | string | ✅ | 插件简短描述 |
| `version` | string | ✅ | 语义化版本号 (如 "1.0.0") |
| `author` | object | ❌ | 作者信息 |
| `license` | string | ❌ | 许可证标识 (如 "MIT") |
| `repository` | string | ❌ | 仓库 URL |
| `homepage` | string | ❌ | 主页 URL |
| `keywords` | string[] | ❌ | 搜索关键词 |
| `dependencies` | object | ❌ | 依赖的其他插件 |
| `bladeVersion` | string | ❌ | 最低 Blade 版本要求 |

## 插件位置

Blade Code 会从以下位置自动加载插件（按优先级从低到高）：

1. **用户级** - `~/.blade/plugins/` 或 `~/.claude/plugins/`
2. **项目级** - `.blade/plugins/` 或 `.claude/plugins/`
3. **CLI 参数** - 通过 `--plugin-dir` 指定（最高优先级）

```bash
# 使用 CLI 参数加载插件
blade --plugin-dir /path/to/my-plugin
```

## 命名空间

为避免命名冲突，插件资源会自动添加命名空间前缀：

- **命令**: `/plugin-name:command-name`
- **技能**: `plugin-name:skill-name`
- **代理**: `plugin-name:agent-name`
- **MCP 服务器**: `plugin-name__server-name`

例如，名为 `my-plugin` 的插件中的 `commit` 命令，使用时为 `/my-plugin:commit`。

## 管理插件

使用 `/plugins` 命令管理插件：

```bash
# 打开插件管理界面
/plugins

# 列出所有已加载的插件
/plugins list

# 显示插件详细信息
/plugins info <plugin-name>

# 从 Git 仓库安装插件
/plugins install user/repo
/plugins install https://github.com/user/repo

# 卸载插件
/plugins uninstall <plugin-name>

# 更新插件
/plugins update <plugin-name>

# 启用/禁用插件
/plugins enable <plugin-name>
/plugins disable <plugin-name>

# 刷新插件列表
/plugins refresh

# 显示插件统计
/plugins stats
```

## 创建自定义命令

在 `commands/` 目录下创建 `.md` 文件：

```markdown
---
description: 我的自定义命令
argument-hint: <参数>
allowed-tools:
  - Read
  - Write
---

这是命令的提示词内容。

当用户执行此命令时，这段文本会作为系统提示发送给 AI。
```

### 命令配置选项

| 字段 | 说明 |
|------|------|
| `description` | 命令描述 |
| `argument-hint` | 参数提示 |
| `allowed-tools` | 允许使用的工具列表 |
| `model` | 指定使用的模型 |
| `disable-model-invocation` | 禁用模型调用 |

## 创建自定义代理

在 `agents/` 目录下创建 `.md` 文件：

```markdown
---
name: my-agent
description: 我的自定义代理
tools:
  - Read
  - Write
  - Bash
color: cyan
permissionMode: default
---

你是一个专业的代码助手。

请按照以下原则工作：
1. 保持代码简洁
2. 遵循最佳实践
```

### 代理配置选项

| 字段 | 说明 |
|------|------|
| `name` | 代理名称 |
| `description` | 代理描述 |
| `tools` | 可用工具列表 |
| `color` | 显示颜色 |
| `model` | 使用的模型 |
| `permissionMode` | 权限模式 |
| `skills` | 启用的技能列表 |

## 创建自定义技能

在 `skills/` 目录下创建技能文件夹，包含 `SKILL.md`：

```markdown
---
name: my-skill
description: 我的自定义技能
tools:
  - Read
  - Write
---

# 技能说明

这是技能的详细说明和使用指南。
```

## 配置 Hooks

在 `hooks/hooks.json` 中配置生命周期钩子：

```json
{
  "hooks": [
    {
      "matcher": "*.ts",
      "hooks": [
        {
          "type": "PreToolUse",
          "command": "npm run lint -- $FILE"
        }
      ]
    }
  ]
}
```

## 配置 MCP 服务器

在 `.mcp.json` 中配置 MCP 服务器：

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./server.js"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

## 最佳实践

1. **使用有意义的名称** - 插件名称应清晰描述其功能
2. **提供完整文档** - 在 README 中说明插件用途和使用方法
3. **版本管理** - 使用语义化版本号管理更新
4. **最小权限原则** - 只请求必要的工具权限
5. **测试插件** - 在发布前充分测试各项功能

## 示例插件

以下是一个完整的示例插件结构：

```
example-plugin/
├── .blade-plugin/
│   └── plugin.json
├── commands/
│   ├── format.md
│   └── lint.md
├── agents/
│   └── code-reviewer.md
├── skills/
│   └── typescript-expert/
│       └── SKILL.md
└── README.md
```

`plugin.json`:
```json
{
  "name": "example-plugin",
  "description": "示例插件，包含代码格式化和审查功能",
  "version": "1.0.0",
  "author": {
    "name": "Blade Team"
  },
  "license": "MIT"
}
```
