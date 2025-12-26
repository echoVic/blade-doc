# 🧰 工具列表（当前实现）

内置工具由 `src/tools/builtin/index.ts` 注册，并在运行时附加已连接的 MCP 工具。工具类型划分依赖 `ToolKind`（ReadOnly / Write / Execute），影响权限模式的判定。

## 文件与编辑

| 名称 | 类型 | 主要参数 | 说明 |
| --- | --- | --- | --- |
| `Read` | ReadOnly | `file_path`（绝对路径，可选 `offset/limit/encoding`） | 读取文本/图片/PDF/ipynb，默认最多 2000 行，返回带行号内容。 |
| `Write` | Write | `file_path`、`content`、`encoding`、`mode`、`mkdirs` | 写入或创建文件，支持备份/权限检查/目录自动创建。 |
| `Edit` | Write | `file_path`、`old_string`、`new_string`、`pattern`、`max_replacements` | 按字符串或正则替换，支持回滚、预览与并发文件锁。 |
| `NotebookEdit` | Write | `file_path`、`content` | 针对 `.ipynb` 的写入，保持 JSON 结构。 |

## 搜索

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `Glob` | ReadOnly | `pattern`、`cwd`、`ignore`、`limit` | 使用 fast-glob 查找文件，内置忽略常见目录。 |
| `Grep` | ReadOnly | `pattern`、`path`、`glob`、`context` 等 | 基于 ripgrep，支持多文件内容搜索、上下文行、大小写/正则开关。 |

## Shell 与执行

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `Bash` | Execute | `command`、`cwd`、`env`、`background`、`timeout` | 执行命令，可后台运行并返回 `bash_id`（后台会话 ID），包含风险提示。 |
| `BashOutput` | ReadOnly | `bash_id`、`filter`(正则，可选) | 读取后台命令的最新输出。 |
| `KillShell` | Execute | `shell_id`、`signal` | 终止后台命令（使用 Bash 返回的后台 ID）。 |
| `Skill` | Execute | `skill_name`、`args` | 动态调用自定义 Skill，根据 `SKILL.md` 定义自动执行。 |
| `SlashCommand` | Execute | `command` | 在主对话中执行 Slash 命令（供系统调用，用户通常无需直接使用）。 |

## 网络

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `WebFetch` | ReadOnly | `url`、`method`、`headers`、`body`、`trim` | 获取网页/接口内容，自动裁剪文本并提示编码。 |
| `WebSearch` | ReadOnly | `query`、`site`、`language`、`region` | 多引擎聚合搜索（Google, DuckDuckGo, Bing 等），智能回退，返回摘要。 |

## 任务与计划

| 名称 | 类型 | 说明 |
| --- | --- | --- |
| `Task` | ReadOnly | 启动子 Agent（使用 `.blade/agents` / `~/.blade/agents` 中的配置），`subagent_type` 决定工具集。 |
| `TodoWrite` | ReadOnly | 管理会话内 TODO（持久化到 `<configDir>/todos/<session>-agent-<session>.json`，默认 `~/.blade/todos`），返回完整列表。 |
| `EnterPlanMode` / `ExitPlanMode` | ReadOnly | 进入/退出 Plan 模式并触发用户确认。 |

## MCP 工具

通过 `blade mcp add ...` 注册的服务器会在运行时加载，其工具会按原始名称加入工具列表。连接状态取决于 MCP 注册表的健康检查。

## 权限与模式提示

- ReadOnly 工具在 `default` / `autoEdit` / `plan` 模式下自动允许（Plan 模式拒绝所有非只读工具）。
- Write 工具仅在 `autoEdit` / `yolo` 自动通过，其余模式需要确认。
- Execute 工具需确认，`yolo` 模式直接放行。

详见「权限系统」章节。
