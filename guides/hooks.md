# Hooks 系统

Blade Hooks System 允许您在工具执行的关键时刻运行自定义命令或脚本，实现工作流自动化和验证。该系统与 Claude Code 的 Hooks 机制高度兼容。

## 快速开始

### 1. 启用 Hooks

在项目根目录下的 `.blade/settings.json` 中启用 hooks：

```json
{
  "hooks": {
    "enabled": true
  }
}
```

### 2. 配置你的第一个 Hook

在 `.blade/settings.json` 中添加 hook 配置。例如，在每次编辑 TypeScript 文件后自动运行 Prettier 格式化：

```json
{
  "hooks": {
    "enabled": true,
    "PostToolUse": [
      {
        "matcher": {
          "tools": "Edit",
          "paths": "**/*.ts"
        },
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write {{file_path}}",
            "statusMessage": "Formatting code..."
          }
        ]
      }
    ]
  }
}
```

## Hook 事件类型

Blade 支持 11 种 Hook 事件：

| 事件 | 触发时机 | 主要用途 |
|------|---------|---------|
| `PreToolUse` | 工具执行前 | 拦截、修改工具调用、请求用户确认 |
| `PostToolUse` | 工具执行后 | 注入上下文、运行验证、代码格式化 |
| `PostToolUseFailure` | 工具执行失败后 | 错误处理、重试逻辑 |
| `PermissionRequest` | 权限确认前 | 自动批准/拒绝 |
| `UserPromptSubmit` | 用户提交消息时 | 动态上下文注入 |
| `SessionStart` | 会话开始时 | 环境变量注入 |
| `SessionEnd` | 会话结束时 | 清理、记录 |
| `Stop` | Agent 完成响应时 | 阻止停止、强制继续 |
| `SubagentStop` | 子 Agent 完成时 | 阻止停止、请求继续 |
| `Notification` | 通知发送时 | 过滤、转发通知 |
| `Compaction` | 上下文压缩时 | 阻止压缩 |

## 配置详解

### 匹配器 (Matcher)

你可以通过 `matcher` 字段指定 Hook 的触发条件：

- `tools`: 匹配工具名称（支持 glob 模式，如 `Edit|Write`）。
- `paths`: 匹配文件路径（支持 glob 模式，如 `**/*.ts`）。
- `commands`: 匹配命令内容（仅限 Bash 工具）。

### Hook 动作

`hooks` 数组定义了触发时要执行的动作。目前主要支持 `command` 类型：

```json
{
  "type": "command",
  "command": "./scripts/check-security.sh",
  "timeout": 5
}
```

- `command`: 要执行的 shell 命令。支持变量插值（如 `{{file_path}}`）。
- `timeout`: 超时时间（秒）。
- `statusMessage`: 执行时在 UI 上显示的状态消息。

### 变量插值

在 `command` 中可以使用以下变量：

- `{{file_path}}`: 相关文件的路径。
- `{{tool_name}}`: 触发 Hook 的工具名称。
- `{{tool_input}}`: 工具的输入参数（JSON 字符串）。

## 示例

### 安全检查 (PreToolUse)

在编辑敏感文件前运行安全扫描脚本：

```json
{
  "PreToolUse": [
    {
      "matcher": {
        "tools": "Edit|Write|NotebookEdit",
        "paths": "**/.env*|**/credentials.json"
      },
      "hooks": [
        {
          "type": "command",
          "command": ".blade/hooks/security-check.sh",
          "timeout": 5
        }
      ]
    }
  ]
}
```

### 自动测试 (PostToolUse)

修改代码后自动运行相关测试：

```json
{
  "PostToolUse": [
    {
      "matcher": {
        "tools": "Edit",
        "paths": "src/**/*.ts"
      },
      "hooks": [
        {
          "type": "command",
          "command": "npm test -- {{file_path}}",
          "statusMessage": "Running tests..."
        }
      ]
    }
  ]
}
```
