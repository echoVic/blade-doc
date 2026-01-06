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
| `TaskOutput` | ReadOnly | 获取后台任务输出，支持后台 Shell（`bash_xxx`）和后台 Agent（`agent_xxx`），可阻塞等待完成或立即返回当前状态。 |
| `TodoWrite` | ReadOnly | 管理会话内 TODO（持久化到 `<configDir>/todos/<session>-agent-<session>.json`，默认 `~/.blade/todos`），返回完整列表。 |
| `EnterPlanMode` / `ExitPlanMode` | ReadOnly | 进入/退出 Plan 模式并触发用户确认。 |

## 交互

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `AskUserQuestion` | ReadOnly | `questions`（1-4 个问题，每个包含 `question`、`header`、`options`、`multiSelect`） | 在执行过程中向用户提问，收集偏好或澄清需求。支持单选/多选，用户可选择预设选项或输入自定义答案。 |

## Spec 模式工具

Spec 模式（规格驱动开发）提供一套完整的工具集，用于结构化的需求分析和任务管理：

| 名称 | 类型 | 参数 | 说明 |
| --- | --- | --- | --- |
| `EnterSpecMode` | ReadOnly | `name`（Spec 名称）、`description`（描述） | 创建新的 Spec 并进入 Spec 模式。 |
| `UpdateSpec` | Write | `file_type`（proposal/requirements/design/tasks）、`content` | 更新指定的 Spec 文档。 |
| `GetSpecContext` | ReadOnly | 无 | 获取当前 Spec 的上下文信息和进度。 |
| `TransitionSpecPhase` | Write | `phase`（目标阶段） | 转换工作流阶段（init → requirements → design → tasks → implementation → done）。 |
| `AddTask` | Write | `title`、`description`、`complexity`、`dependencies`、`files` | 添加新任务到任务列表。 |
| `UpdateTaskStatus` | Write | `task_id`、`status`（pending/in_progress/completed/skipped） | 更新任务状态。 |
| `ValidateSpec` | ReadOnly | 无 | 验证 Spec 的完整性和一致性。 |
| `ExitSpecMode` | ReadOnly | `archive`（是否归档） | 退出 Spec 模式，可选择归档已完成的 Spec。 |

## MCP 工具

通过 `blade mcp add ...` 注册的服务器会在运行时加载，其工具会按原始名称加入工具列表。连接状态取决于 MCP 注册表的健康检查。

## 权限与模式提示

- ReadOnly 工具在 `default` / `autoEdit` / `plan` / `spec` 模式下自动允许（Plan 模式拒绝所有非只读工具）。
- Write 工具仅在 `autoEdit` / `yolo` 自动通过，其余模式需要确认。
- Execute 工具需确认，`yolo` 模式直接放行。
- Spec 模式下，Spec 相关的 Write 工具（如 `UpdateSpec`、`AddTask`）会自动允许。

详见「权限系统」章节。
