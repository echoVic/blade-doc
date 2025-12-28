# 🧰 工具列表（当前实现）

内置工具由 `src/tools/builtin/index.ts` 注册，并在运行时附加已连接的 MCP 工具。工具类型划分依赖 `ToolKind`（ReadOnly / Write / Execute），影响权限模式的判定。

## 文件与编辑

| 名称 | 类型 | 主要参数 | 说明 |
| --- | --- | --- | --- |
| `Read` | ReadOnly | `file_path`（绝对路径，可选 `offset/limit/encoding`） | 读取文本/图片/PDF/ipynb，默认最多 2000 行，返回带行号内容。支持 `encoding: base64` 读取二进制文件。 |
| `Write` | Write | `file_path`、`content`、`encoding`、`create_directories` | 写入或创建文件；若文件已存在，必须先使用 Read 读取。支持二进制写入（`encoding: base64`）。 |
| `Edit` | Write | `file_path`、`old_string`、`new_string`、`replace_all` | 基于字符串的精确替换，`replace_all` 控制是否全量替换。 |
| `NotebookEdit` | Write | `notebook_path`、`cell_id`、`new_source`、`cell_type`、`edit_mode` | 按 cell ID 替换/插入/删除单元格（`edit_mode`: replace/insert/delete）。 |

> **Base64 编解码**：Read 和 Write 工具通过 `encoding: 'base64'` 参数支持二进制文件的读写。Read 会自动检测二进制文件并以 Base64 编码返回内容。

## 搜索

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `Glob` | ReadOnly | `pattern`、`path`、`max_results`、`include_directories`、`case_sensitive` | 使用 fast-glob 查找文件，遵循 `.gitignore`。 |
| `Grep` | ReadOnly | `pattern`、`path`、`glob`、`type`、`output_mode`、`-i/-n/-A/-B/-C`、`head_limit`、`offset`、`multiline` | 基于 ripgrep，支持正则、上下文与多输出模式。 |

## Shell 与执行

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `Bash` | Execute | `command`、`cwd`、`env`、`timeout`、`run_in_background` | 执行命令；后台运行会返回 `bash_id`（后台会话 ID）。 |
| `BashOutput` | ReadOnly | `bash_id`、`filter`(正则，可选) | 读取后台命令的最新输出。 |
| `KillShell` | Execute | `shell_id` | 终止后台命令（使用 Bash 返回的后台 ID）。 |
| `Skill` | Execute | `skill`、`args` | 动态调用自定义 Skill，根据 `SKILL.md` 定义执行。 |
| `SlashCommand` | Execute | `command`、`arguments` | 执行自定义 Slash 命令（仅限自定义命令）。 |

## 网络

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `WebFetch` | ReadOnly | `url`、`method`、`headers`、`body`、`timeout`、`follow_redirects`、`max_redirects`、`return_headers` | 获取网页/接口内容并返回响应数据。 |
| `WebSearch` | ReadOnly | `query`、`allowed_domains`、`blocked_domains` | 多提供商聚合搜索，自动回退并返回摘要结果。 |

## 任务与计划

| 名称 | 类型 | 说明 |
| --- | --- | --- |
| `Task` | ReadOnly | 启动子 Agent（使用 `.blade/agents` / `~/.blade/agents` 中的配置），参数 `subagent_type`、`description`、`prompt`。 |
| `TodoWrite` | ReadOnly | 管理会话内 TODO（参数 `todos`，包含 `content` / `activeForm` / `status` / `priority`）。 |
| `EnterPlanMode` / `ExitPlanMode` | ReadOnly | 进入/退出 Plan 模式并触发用户确认；`ExitPlanMode` 需要 `plan` 文本。 |

## MCP 工具

通过 `blade mcp add ...` 注册的服务器会在运行时加载，其工具会按原始名称加入工具列表。连接状态取决于 MCP 注册表的健康检查。

## 权限与模式提示

- ReadOnly 工具在 `default` / `autoEdit` / `plan` 模式下自动允许（Plan 模式拒绝所有非只读工具）。
- Write 工具仅在 `autoEdit` / `yolo` 自动通过，其余模式需要确认。
- Execute 工具需确认，`yolo` 模式直接放行。

详见「权限系统」章节。
