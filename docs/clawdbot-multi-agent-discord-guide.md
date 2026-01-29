# Clawdbot 多 Agent Discord 路由配置指南

> 基于实战踩坑经验整理，避免你走弯路

## 概述

本教程教你如何配置一个 Discord Bot，根据不同频道自动路由到不同的 Agent 后端。

```
Discord Bot (1个)
    │
    ├── #main 频道 ──→ main agent
    ├── #code 频道 ──→ coder agent
    └── #research 频道 ──→ researcher agent
```

**适用场景**：
- 想让编程讨论和日常闲聊分开
- 不同项目用不同的 agent 上下文
- 团队成员分配专属 agent

## 前置条件

- 已安装 Clawdbot/Moltbot
- 有 Discord 账号

## 第一步：创建 Discord Bot

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 **New Application**，输入名称
3. 左侧菜单选择 **Bot**
4. 点击 **Reset Token** 获取 Bot Token（**只显示一次，立即复制保存**）
5. 开启以下 Intents：
   - ✅ MESSAGE CONTENT INTENT
   - ✅ SERVER MEMBERS INTENT（可选）
6. 左侧选择 **OAuth2 → URL Generator**：
   - Scopes: 勾选 `bot`
   - Bot Permissions: 勾选 `Send Messages`, `Read Message History`
7. 复制生成的 URL，在浏览器打开，邀请 Bot 到你的服务器

## 第二步：获取 Discord ID

在 Discord 客户端：
1. 设置 → 高级 → 开启 **开发者模式**
2. 右键服务器图标 → **复制服务器 ID**（Guild ID）
3. 右键频道名称 → **复制频道 ID**（Channel ID）
4. 右键自己头像 → **复制用户 ID**（User ID）

## 第三步：配置 Discord 频道

编辑 `~/.clawdbot/clawdbot.json`，在 `channels` 中添加：

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "你的Bot Token",
      "groupPolicy": "allowlist",
      "dm": {
        "policy": "allowlist",
        "allowFrom": ["你的用户ID"]
      },
      "guilds": {
        "你的服务器ID": {
          "users": ["你的用户ID"]
        }
      }
    }
  }
}
```

### 踩坑点 1：安全配置

| 配置项 | 错误写法 | 正确写法 |
|--------|----------|----------|
| DM 开放 | `"policy": "open"` | `"policy": "allowlist", "allowFrom": ["*"]` |
| 群组白名单 | 只写 `"groupPolicy": "allowlist"` | 必须配合 `guilds` 对象 |

## 第四步：添加 Agent

使用 CLI 添加 agent：

```bash
# 添加 coder agent
clawdbot agents add coder \
  --workspace ~/clawd/coder \
  --non-interactive

# 添加 researcher agent
clawdbot agents add researcher \
  --workspace ~/clawd/researcher \
  --non-interactive

# 验证
clawdbot agents list
```

## 第五步：配置频道绑定（重点！）

### 踩坑点 2：bindings 位置

❌ **错误**：放在 `agents` 下面
```json
{
  "agents": {
    "bindings": [...]  // 错！会报 Unrecognized key
  }
}
```

✅ **正确**：放在配置文件**顶层**
```json
{
  "agents": { ... },
  "channels": { ... },
  "bindings": [...]  // 顶层！
}
```

### 踩坑点 3：peer 格式

❌ **错误**：peer 用字符串
```json
{
  "peer": "1466336834400424057"  // 错！
}
```

✅ **正确**：peer 是对象
```json
{
  "peer": { "kind": "channel", "id": "1466336834400424057" }
}
```

### 完整的 bindings 配置

在 `~/.clawdbot/clawdbot.json` **顶层**添加：

```json
{
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "main频道ID" }
      }
    },
    {
      "agentId": "coder",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "code频道ID" }
      }
    },
    {
      "agentId": "researcher",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "research频道ID" }
      }
    }
  ]
}
```

## 第六步：启动并验证

```bash
# 验证配置无误
clawdbot agents list --json
# 每个 agent 应显示 "bindings": 1

# 启动 gateway
clawdbot gateway start

# 检查状态
clawdbot status
# Discord 应显示 ON / OK
```

## 第七步：测试

在 Discord 不同频道 @机器人 发消息：
- #main 频道 → main agent 响应
- #code 频道 → coder agent 响应
- #research 频道 → researcher agent 响应

## 常见问题

### Q: 配置报错 "Unrecognized key"
运行 `clawdbot doctor --fix` 自动移除无效键，然后检查配置格式。

### Q: Bot 不响应
1. 检查 `clawdbot status` 中 Discord 是否为 ON/OK
2. 确认 Bot 已被邀请到服务器
3. 确认你的用户 ID 在 `guilds.用户ID.users` 列表中
4. 确认开启了 MESSAGE CONTENT INTENT

### Q: 所有频道都由 main 响应
检查 bindings 中的 `peer.id` 是否与频道 ID 完全匹配。

### Q: Agent 之间能共享记忆吗？
不能。每个 agent 有独立的会话历史。但如果工作区有父子关系（如 `~/clawd` 包含 `~/clawd/coder`），父 agent 可以读取子目录的文件。

## 完整配置示例

<details>
<summary>点击展开完整 clawdbot.json 示例</summary>

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5"
      },
      "workspace": "/Users/你的用户名/clawd"
    },
    "list": [
      { "id": "main" },
      {
        "id": "coder",
        "name": "coder",
        "workspace": "/Users/你的用户名/clawd/coder",
        "agentDir": "/Users/你的用户名/.clawdbot/agents/coder/agent"
      },
      {
        "id": "researcher",
        "name": "researcher",
        "workspace": "/Users/你的用户名/clawd/researcher",
        "agentDir": "/Users/你的用户名/.clawdbot/agents/researcher/agent"
      }
    ]
  },
  "channels": {
    "discord": {
      "enabled": true,
      "token": "你的Bot Token",
      "groupPolicy": "allowlist",
      "dm": {
        "policy": "allowlist",
        "allowFrom": ["你的用户ID"]
      },
      "guilds": {
        "你的服务器ID": {
          "users": ["你的用户ID"]
        }
      }
    }
  },
  "plugins": {
    "entries": {
      "discord": { "enabled": true }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "main频道ID" }
      }
    },
    {
      "agentId": "coder",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "code频道ID" }
      }
    },
    {
      "agentId": "researcher",
      "match": {
        "channel": "discord",
        "guildId": "你的服务器ID",
        "peer": { "kind": "channel", "id": "research频道ID" }
      }
    }
  ]
}
```

</details>

## 参考链接

- [Moltbot 官方文档](https://docs.molt.bot)
- [多 Agent 路由](https://docs.molt.bot/concepts/multi-agent)
- [Discord 频道配置](https://docs.molt.bot/channels/discord)

---

*最后更新: 2026-01-29*
