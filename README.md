# Clawdbot/Moltbot 配置指南与工作区模板

> 基于实战踩坑经验整理，帮你少走弯路

## 这是什么

[Clawdbot](https://github.com/nicholasoxford/clawdbot)（现已更名为 [Moltbot](https://molt.bot)）是一个 AI 助手网关，可以将 Claude/GPT 等大模型连接到 WhatsApp、Discord、Telegram 等聊天平台。

本仓库包含：
- 🗂️ **工作区模板** - 开箱即用的 agent 配置文件
- 📚 **配置教程** - 多 agent Discord 路由等进阶玩法
- 🚫 **踩坑记录** - 避免你重复踩坑

## 仓库结构

```
.
├── AGENTS.md           # Agent 行为指南（核心！）
├── SOUL.md             # Agent "性格" 配置
├── USER.md             # 用户信息模板
├── IDENTITY.md         # Agent 身份模板
├── BOOTSTRAP.md        # 首次启动引导
├── TOOLS.md            # 工具配置笔记
├── HEARTBEAT.md        # 心跳任务配置
├── canvas/             # Web UI 模板
├── coder/              # coder agent 工作区示例
├── researcher/         # researcher agent 工作区示例
└── docs/
    └── clawdbot-multi-agent-discord-guide.md
```

## 快速开始

### 1. 克隆到你的工作区

```bash
git clone https://github.com/logicrw/clawdbot-guides.git ~/clawd
```

### 2. 配置 Clawdbot

```bash
# 安装
npm install -g clawdbot

# 初始化（指定工作区）
clawdbot setup --workspace ~/clawd

# 启动
clawdbot gateway start
```

### 3. 个性化配置

编辑以下文件，填入你的信息：
- `USER.md` - 你的名字、时区等
- `IDENTITY.md` - 给 agent 起个名字

## 教程列表

| 教程 | 说明 |
|------|------|
| [多 Agent Discord 路由](docs/clawdbot-multi-agent-discord-guide.md) | 一个 Bot 根据频道路由到不同 Agent |

## 关键踩坑点

### bindings 配置位置

❌ 错误：放在 `agents` 下
```json
{ "agents": { "bindings": [...] } }
```

✅ 正确：放在配置文件**顶层**
```json
{ "agents": {...}, "bindings": [...] }
```

### peer 格式

❌ 错误：字符串
```json
{ "peer": "123456789" }
```

✅ 正确：对象
```json
{ "peer": { "kind": "channel", "id": "123456789" } }
```

更多详见 [多 Agent Discord 路由教程](docs/clawdbot-multi-agent-discord-guide.md)。

## 注意事项

以下目录/文件已在 `.gitignore` 中排除，不会上传：
- `memory/` - 对话记忆（可能含隐私）
- `MEMORY.md` - 长期记忆
- `.env` - 环境变量

## 相关链接

- [Moltbot 官方文档](https://docs.molt.bot)
- [Moltbot GitHub](https://github.com/nicholasoxford/clawdbot)
- [Discord 频道配置](https://docs.molt.bot/channels/discord)
- [多 Agent 路由](https://docs.molt.bot/concepts/multi-agent)

## License

MIT
