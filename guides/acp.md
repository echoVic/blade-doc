# ACP 集成（IDE）

ACP（Agent Client Protocol）是一种通过 stdio 让 IDE 调用本地 Agent 的协议。Blade 支持以 ACP Agent 模式运行，供支持 ACP 的客户端（如 Zed、JetBrains、Neovim 等）接入。

## 快速开始

1. 在 IDE 中安装/启用 ACP 客户端（具体方式以 IDE 或插件文档为准）。
2. 配置 IDE 启动命令为 `blade --acp`，并设置工作目录为项目根目录。
3. 确保已完成模型配置（`~/.blade/config.json` 或 `.blade/config.json`）。

**参考配置（通用）**

```bash
blade --acp
```

如需日志排查，可加 `--debug`：

```bash
blade --acp --debug agent,tool
```

## 功能说明

- **无 UI**：ACP 模式不启动交互式终端界面，只通过 stdio 与 IDE 通信。
- **权限模式**：IDE 可切换 `default` / `autoEdit` / `yolo` / `plan` 模式，行为与 CLI 一致。
- **文件系统**：若 IDE 提供文件读写能力，Blade 会优先走 IDE 文件系统；未提供时会回退到本地文件系统。
- **二进制限制**：ACP 模式下不支持二进制写入；二进制读取会回退到本地文件系统。

## Zed 配置（无扩展）

当前没有 Blade 的 Zed 扩展，需在 Zed 的 settings.json 中手动注册 ACP Agent：

1. 打开 Zed 设置文件（Settings → Open Settings File）。
2. 在 `agent_servers` 中添加一个 ACP 外部 Agent，启动命令指向 `blade --acp`。
3. 确保已配置好模型与 API Key（`~/.blade/config.json` 或 `.blade/config.json`）。

**示例（Zed settings.json）**

```json
{
  "agent_servers": {
    "Blade": {
      "command": "blade",
      "args": ["--acp"]
    }
  }
}
```

> Zed 的设置键名可能随版本变化，以上示例以 `agent_servers` 为准；请以 Zed 文档/设置提示为准。

## ACP 与 `/ide` 的区别

- **ACP**：IDE 作为 ACP Client 直接启动 `blade --acp`，用于 Zed/JetBrains/Neovim 等支持 ACP 的场景。
- **/ide**：在 Blade 交互式 UI 中连接本地 IDE（VS Code 插件等），与 ACP 是两条独立链路。

## 常见问题

**Q: ACP 模式下为什么不能写二进制文件？**  
ACP 文件系统目前仅支持文本读写，二进制写入会被拒绝。

**Q: IDE 没有提供文件系统能力会怎样？**  
Blade 会回退到本地文件系统操作，请确保工作目录正确且有权限。
