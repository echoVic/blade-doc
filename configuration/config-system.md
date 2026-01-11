# Blade 配置系统

Blade 使用双文件配置体系：`config.json`（基础配置）和 `settings.json` / `settings.local.json`（行为配置）。加载顺序和字段均来自当前实现的 `ConfigManager` / `defaults.ts`。

## 文件位置与优先级

```
~/.blade/            # 用户级
  ├─ config.json
  └─ settings.json

<project>/.blade/    # 项目级
  ├─ config.json
  ├─ settings.json
  └─ settings.local.json   # 个人覆盖，自动加入 .gitignore
```

优先级（高 → 低）：环境变量插值 > settings.local.json > 项目 settings.json > 用户 settings.json > 项目 config.json > 用户 config.json > 默认值。

## config.json（基础配置）

**模型与连接**

Blade 支持多种模型提供商，包括 OpenAI 兼容接口、Anthropic、Gemini、Azure OpenAI，以及 OAuth 方式的 Antigravity/Copilot。

### 内置免费模型（0.1.0+）

从 0.1.0 起，Blade 会在启动时把一个“内置免费模型”注入到模型列表中（默认使用智谱 GLM-4.7），并在你未设置 `currentModelId` 时自动选中它。

- 内置模型会出现在 UI 的模型选择器里，但不一定会写入你的 `config.json`（除非你手动保存配置）
- 你仍然可以通过 `/model add` 添加自有模型，并随时切换

> 内置模型用于开箱即用体验：不需要你提供 API Key。它的 `apiKey` 在内部使用固定标记值（例如 `blade-free-tier`），用于区分“内置免费额度”和用户自配密钥。

**支持的 Provider**

- `openai-compatible`: OpenAI 兼容接口（DeepSeek, Ollama, Qwen 等）
- `anthropic`: Anthropic Claude
- `gemini`: Google Gemini
- `antigravity`: Google Antigravity（OAuth 登录，统一网关访问 Claude/Gemini/GPT-OSS）
- `copilot`: GitHub Copilot（OAuth 登录，访问 GPT/Claude/Gemini 等）
- `azure-openai`: Azure OpenAI Service
- `gpt-openai-platform`: 字节跳动内部 GPT 平台

> `antigravity` / `copilot` 使用 OAuth：先执行 `/login`，再用 `/model add` 添加模型配置；`baseUrl` 为固定端点，`apiKey` 可留空。

**模型配置示例**

```json
{
  "currentModelId": "qwen-main",
  "models": [
    {
      "id": "qwen-main",
      "name": "Qwen",
      "provider": "openai-compatible",
      "apiKey": "${QWEN_API_KEY}",
      "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
      "model": "qwen-max",
      "temperature": 0,
      "maxContextTokens": 128000
    },
    {
      "id": "gemini-pro",
      "name": "Gemini Pro",
      "provider": "gemini",
      "apiKey": "${GEMINI_API_KEY}",
      "model": "gemini-1.5-pro-latest",
      "maxContextTokens": 1000000
    },
    {
      "id": "deepseek-r1",
      "name": "DeepSeek R1",
      "provider": "openai-compatible",
      "apiKey": "${DEEPSEEK_API_KEY}",
      "baseUrl": "https://api.deepseek.com",
      "model": "deepseek-reasoner",
      "supportsThinking": true,
      "thinkingBudget": 16000
    }
  ]
}
```

**模型字段说明**

| 字段 | 说明 | 备注 |
| --- | --- | --- |
| `provider` | 提供商类型 | 见上文支持列表 |
| `supportsThinking` | 是否支持思维链 (CoT) | 用于 DeepSeek R1 等推理模型 |
| `thinkingBudget` | 思维链 Token 预算 | 仅当 `supportsThinking` 为 true 时生效 |
| `apiVersion` | API 版本 | Azure OpenAI / GPT Platform 必填 |
| `projectId` | 项目 ID | Antigravity 可选（自动获取后可写入） |

**通用参数**

| 字段 | 说明 | 默认值 |
| --- | --- | --- |
| `temperature` | 默认采样温度 | `0.0` |
| `maxContextTokens` | 上下文窗口（用于压缩判断） | `128000` |
| `maxOutputTokens` | 单次回复输出上限 | `32768` |
| `stream` | 是否请求流式输出 | `true` |
| `topP` / `topK` | 采样参数 | `0.9` / `50` |
| `timeout` | LLM 请求超时（ms） | `180000` |
| `theme` / `language` / `fontSize` | UI 主题/语言/字号 | `github` / `zh-CN` / `14` |
| `debug` | `false` / `true` / `"agent,ui"` 形式的过滤字符串 | `false` |
| `mcpEnabled` | 是否启用 MCP | `false`（当前未作为硬开关） |
| `mcpServers` | MCP 服务器配置字典 | `{}` |

> `fallbackModel`、`allowedTools`、`settingSources` 等字段仅存在于运行时类型中，当前未接入主要流程。

### Token 用量统计（0.0.48+）

从 0.0.48 起，Blade 在流式响应中也会尽量收集各提供商返回的 `usage` 信息，并用于：

- 更准确地计算“上下文剩余比例”（状态栏的百分比）
- 更可靠地触发自动压缩（优先使用真实 Token 数据而非估算）

## settings.json / settings.local.json（行为配置）

| 字段 | 说明 | 默认值（见 `defaults.ts`） |
| --- | --- | --- |
| `permissions` | `allow/ask/deny` 规则数组 | 预置了常见只读 Bash 命令白名单与危险命令黑名单 |
| `permissionMode` | `default` / `autoEdit` / `yolo` / `plan` | `default` |
| `hooks` | Hook 配置（启用、超时、Pre/Post 列表等） | 全部关闭 |
| `env` | 注入到会话的环境变量 | `{}` |
| `disableAllHooks` | 全局禁用 Hooks | `false` |
| `maxTurns` | 触发询问的轮次阈值（0 禁用，-1 为默认值，受安全上限 100 保护） | `-1` |

`settings.local.json` 用于个人偏好或临时授权，`ConfigManager` 会自动把它写入 `.gitignore`。

## 环境变量插值

所有字符串字段支持 `$VAR` / `${VAR}` / `${VAR:-default}` 插值，解析发生在加载阶段。例如：

```json
{
  "apiKey": "${QWEN_API_KEY}",
  "baseUrl": "${BLADE_BASE_URL:-https://apis.iflow.cn/v1}"
}
```

## 典型文件示例

**用户配置：`~/.blade/config.json`**
```json
{
  "currentModelId": "qwen",
  "models": [
    {
      "id": "qwen",
      "name": "Qwen",
      "provider": "openai-compatible",
      "apiKey": "${QWEN_API_KEY}",
      "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
      "model": "qwen-max"
    }
  ],
  "theme": "github",
  "language": "zh-CN",
  "debug": false
}
```

**项目共享配置：`.blade/settings.json`**
```json
{
  "permissions": {
    "allow": [
      "Bash(git status*)",
      "Bash(ls *)",
      "Read(file_path:**/*.ts)"
    ],
    "deny": [
      "Read(file_path:**/.env*)",
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "enabled": true,
    "PostToolUse": [
      {
        "matcher": { "tools": "Write", "paths": "**/*.ts" },
        "hooks": [
          { "type": "command", "command": "npx prettier --write {{file_path}}" }
        ]
      }
    ]
  },
  "env": {
    "NODE_ENV": "development"
  }
}
```

**个人覆盖：`.blade/settings.local.json`**
```json
{
  "permissions": {
    "allow": ["Bash(npm run build*)"]
  },
  "permissionMode": "autoEdit"
}
```

## 配置入口

- **首次启动**：从 0.1.0 起可直接使用内置免费模型开始对话；如需自定义模型，输入 `/model add` 打开配置向导。
- **手工编辑**：直接修改上述文件即可，保存后下次启动生效。
- **自动写入**：在权限确认弹窗中选择“会话内记住”会把抽象后的规则写入 `.blade/settings.local.json`。

## 实现要点

- 加载与合并：`src/config/ConfigManager.ts`（包含环境变量插值与 gitignore 处理）。
- 写入与同步：`configActions()` 统一入口，持久化到文件并同步 Zustand Store。
- 校验：`ConfigManager.validateConfig` 会在 Agent 创建前校验模型配置是否存在。
