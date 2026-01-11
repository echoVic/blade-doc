# 🚀 Blade Code 快速开始

## 三步上手

### 1）启动

```bash
blade          # 交互式界面
npx blade-code # 零安装体验
```

> 从 0.1.0 起，Blade 内置免费模型（GLM-4.7），无需任何配置即可直接开始对话；如需使用自有模型，随时输入 `/model add`。

### 2）提问

```bash
blade "帮我创建一个 React 组件"          # UI 就绪后自动发送
blade --print "解释什么是 TypeScript"   # 打印模式，适合脚本
blade --permission-mode autoEdit "修复 lint 错误"
```

### 3）可选：全局安装

```bash
npm install -g blade-code   # 或 pnpm/yarn 对等命令
blade
```

## 常用命令

```bash
blade "什么是人工智能？"
blade --resume              # 交互选择历史会话
blade doctor                # 环境检查
blade mcp list              # 查看 MCP 配置
```

## 模型与密钥配置

- **向导**：输入 `/model add` 打开模型配置向导，添加你的 API Key / Base URL 等配置。
- **手工**：编辑 `~/.blade/config.json` / `.blade/config.json`，模型结构见配置章节，密钥可用 `${ENV}` 插值。

## 验证

```bash
blade --version
blade --help
blade --print "测试一下"
```

## 智能工具

```bash
blade "审查 app.js"
blade "查看当前 git 状态"
blade "帮我梳理 src/ui 目录"
```

## 安全确认

写入、命令执行等会根据权限规则弹出确认。可通过权限模式切换：
- `Shift+Tab` 在 `default → autoEdit → plan` 循环
- `--permission-mode yolo` 启动时直接全自动批准（需自担风险）
- **轮次上限**：长时间任务达到阈值（默认 100 轮）时会暂停并询问，防止意外消耗过多 Token。
