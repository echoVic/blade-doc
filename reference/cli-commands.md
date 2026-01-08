# 📋 Blade CLI 命令参考

本文基于当前源码（`src/blade.tsx`、`src/commands/*`、`src/cli/config.ts`）梳理，确保与实际行为一致。

## 🎯 默认入口 `blade [message…]`

```bash
# 打开交互式界面
blade

# UI 就绪后自动发送首条消息
blade "帮我创建一个 README"
```

- 无子命令时启动 Ink 界面。若未配置模型，会自动进入「模型配置向导」（`ModelConfigWizard`）。
- 传入的 `message` 会在 UI 初始化后直接提交，无需手动回车。

**常用参数**

| 参数 | 作用 |
| --- | --- |
| `--debug [filters]` | 打开调试日志，支持类别过滤（如 `--debug agent,ui` 或 `--debug "!chat,!loop"`）。 |
| `--permission-mode <mode>` | 权限模式：`default`（默认）、`autoEdit`（自动编辑）、`yolo`（全自动）、`plan`（规划模式）、`spec`（规格驱动开发模式）。 |
| `--yolo` | `--permission-mode yolo` 的快捷方式，自动批准所有工具调用。 |
| `--resume [id]` / `-r` | 恢复历史会话：`--resume` 交互选择，`--resume <id>` 直接加载。 |
| `--continue` / `-c` | 继续最近一次对话。 |
| `--max-turns <n>` | 控制对话轮次（0 = 禁用对话，-1 = 无限但受安全上限 100 轮保护，N > 0 = 限制轮次）。 |
| `--system-prompt <text>` | 替换默认系统提示词。 |
| `--append-system-prompt <text>` | 追加到默认系统提示词。 |

**输出与输入选项**

| 参数 | 作用 |
| --- | --- |
| `--output-format <format>` | 输出格式：`text`（默认）、`json`、`stream-json`。仅在 `--print` 模式下生效。 |
| `--input-format <format>` | 输入格式：`text`（默认）、`stream-json`。 |
| `--include-partial-messages` | 流式输出时包含部分消息块。 |
| `--replay-user-messages` | 重放 stdin 中的用户消息（当前仅解析参数，尚未接入输出管线）。 |

**安全选项**

| 参数 | 作用 |
| --- | --- |
| `--allowed-tools <tools>` | 允许的工具白名单（空格或逗号分隔）。 |
| `--disallowed-tools <tools>` | 禁止的工具黑名单（空格或逗号分隔）。 |
| `--add-dir <paths>` | 额外允许工具访问的目录。 |

**会话选项**

| 参数 | 作用 |
| --- | --- |
| `--session-id <id>` | 使用指定的会话 ID。 |
| `--fork-session` | 恢复时创建新会话 ID（fork 模式）。 |

**MCP 选项**

| 参数 | 作用 |
| --- | --- |
| `--mcp-config <paths>` | 从 JSON 文件或字符串加载 MCP 服务器配置。 |
| `--strict-mcp-config` | 仅使用 `--mcp-config` 指定的 MCP 服务器。 |

**配置选项**

| 参数 | 作用 |
| --- | --- |
| `--settings <path|json>` | 指定 settings JSON 路径或字符串（当前仅解析参数，尚未接入）。 |
| `--agents <json>` | 自定义 Agent 配置（当前仅解析参数，尚未接入）。 |
| `--setting-sources <list>` | 指定配置来源列表（当前仅解析参数，尚未接入）。 |

**集成选项**

| 参数 | 作用 |
| --- | --- |
| `--acp` | 以 ACP（Agent Client Protocol）模式运行，用于 IDE 集成。 |
| `--ide` | 启动时自动连接 IDE（当前仅解析参数，未自动连接）。 |

**插件选项**

| 参数 | 作用 |
| --- | --- |
| `--plugin-dir <path>` | 从指定目录加载插件（优先级最高）。 |

## 🖨️ 打印模式 `-p / --print`

走 `src/commands/print.ts` 分支，不启动 UI：

```bash
blade --print "解释什么是 TypeScript"
echo "请总结这段文字" | blade -p --output-format json
```

- 支持 `--output-format text|json|stream-json`（默认 text）。
- 会创建最小化 Agent，读取命令行消息或 stdin，再直接输出结果。

## 🛠️ 子命令

| 子命令 | 说明 | 备注 |
| --- | --- | --- |
| `blade doctor` | 环境自检：配置加载、Node 版本、当前目录读写权限、依赖可用性。 | 成功返回 0，否则返回 1。 |
| `blade install [stable\|latest] [--force]` | 占位实现，模拟下载/安装输出。 | 目前仅打印流程，不做实际安装。 |
| `blade update` | 读取本地 `package.json` 版本并提示已是最新。 | 仅本地检查，无联网。 |
| `blade mcp ...` | 管理 MCP 服务器。见下表。 | |

`mcp` 子命令（`src/commands/mcp.ts`）：

| 命令 | 用法 | 说明 |
| --- | --- | --- |
| `mcp list` / `mcp ls` | `blade mcp list` | 列出已加载的 MCP 服务器（全局 + 项目）并尝试连接，完成后自动断开。 |
| `mcp add <name> <cmdOrUrl> [args…]` | `blade mcp add github -- npx -y @modelcontextprotocol/server-github` | 支持 `--transport stdio|http|sse`、`--env KEY=VAL`、`--header "K: V"`、`--timeout <ms>`、`--global`。 |
| `mcp remove <name>` / `rm` | `blade mcp remove github` | 从项目配置移除服务器，支持 `--global` 删除全局配置。 |
| `mcp get <name>` | `blade mcp get github` | 打印单个服务器配置 JSON。 |
| `mcp add-json <name> '<json>'` | `blade mcp add-json api '{"type":"http","url":"..."}'` | 直接传入 JSON 串，支持 `--global`。 |

## 🧭 交互界面要点

- `/` 开头触发 Slash 命令补全，`@` 开头触发文件路径补全（自动读取并注入上下文，见「@ 文件提及」章节）。
- 快捷键：`Ctrl+C / Ctrl+D` 终止任务或退出；`Ctrl+L` 清屏；`Ctrl+T` 展开/折叠思维链；`Esc` 关闭建议/中断执行；`Shift+Tab` 在 `default → autoEdit → plan → spec` 间循环。
- 没有模型配置时自动进入模型向导；若解析配置失败会在对话区显示错误。

## 📋 权限模式说明

Blade 支持五种权限模式，通过 `--permission-mode` 参数或 `Shift+Tab` 快捷键切换：

| 模式 | 说明 |
| --- | --- |
| `default` | 默认模式，只读工具自动允许，写入和执行工具需要确认。 |
| `autoEdit` | 自动编辑模式，写入工具自动允许，执行工具需要确认。 |
| `yolo` | 全自动模式，所有工具自动允许（危险）。 |
| `plan` | 规划模式，仅允许只读工具，用于调研和方案制定。 |
| `spec` | 规格驱动开发模式，支持结构化的需求分析和任务管理。 |

## 🔁 示例

```bash
# 打印模式（脚本集成）
git diff | blade --print --append-system-prompt "请给出代码审查建议"

# 启用 Plan 模式启动 UI
blade --permission-mode plan

# 启用 Spec 模式启动 UI
blade --permission-mode spec

# 恢复历史会话并继续对话
blade --resume 2024-12-foo-session
```

更多用法见其他章节（工具列表、Slash 命令、Plan 模式、Spec 模式等）。
