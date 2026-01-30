# Clawdbot 多 Agent Discord 路由配置指南

> 基于实战踩坑经验整理，避免你走弯路

## 概述

本教程教你如何配置一个**私人专属** Discord Bot，根据不同频道自动路由到不同的 Agent 后端。

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
- 个人私有 AI 助手

## 前置条件

- 已安装 Clawdbot/Moltbot
- 有 Discord 账号

---

## 第零步：创建私有 Discord 服务器

> ⚠️ **重要**：为了防止他人发现你的 Bot，建议创建一个**私有服务器**专门用于 Bot 交互。

### 创建服务器

1. 在 Discord 左侧点击 **+** 创建服务器
2. 选择 **Create My Own** → **For me and my friends**
3. 输入服务器名称（如 "My AI Assistant"）

### 设置服务器为私有

确保服务器不会被他人发现：

1. 右键服务器图标 → **服务器设置**
2. 左侧选择 **社区** → 确保**未启用**社区功能
3. 左侧选择 **Discovery**（发现）→ 确保**未启用**服务器发现
4. 左侧选择 **邀请** → 可以禁用或限制邀请链接

### 私有服务器的好处

| 设置 | 效果 |
|------|------|
| 不启用社区功能 | 服务器不会出现在 Discord 的公开列表中 |
| 不启用服务器发现 | 无法通过搜索找到你的服务器 |
| 不分享邀请链接 | 只有你知道服务器的存在 |

这样即使有人知道你的 Bot 存在，也无法找到你的服务器来尝试与 Bot 交互。

---

## 核心概念：Discord ID 体系

在配置之前，先理解 Discord 的 ID 体系：

```
Discord
├── 服务器 (Guild)          ← Guild ID（服务器唯一标识）
│   ├── #频道1 (Channel)    ← Channel ID（频道唯一标识）
│   ├── #频道2 (Channel)    ← Channel ID
│   └── 成员
│       ├── 用户A           ← User ID（用户全局唯一标识）
│       └── 用户B           ← User ID
└── Bot                     ← Bot ID / Client ID（Bot 唯一标识）
```

### ID 类型说明

| ID 类型 | 作用 | 示例 | 在哪里获取 |
|---------|------|------|-----------|
| **Guild ID** | 标识一个服务器 | `1234567890123456789` | 右键服务器图标 → 复制服务器 ID |
| **Channel ID** | 标识一个频道 | `1234567890123456790` | 右键频道名称 → 复制频道 ID |
| **User ID** | 标识一个用户（全局唯一） | `1234567890123456791` | 右键用户头像 → 复制用户 ID |
| **Bot ID** | 标识一个 Bot（= Client ID） | `1234567890123456792` | Developer Portal → General Information |

### 配置中各 ID 的用途

```json
{
  "guilds": {
    "Guild ID": {           // ← 允许 Bot 响应的服务器
      "users": ["User ID"]  // ← 允许与 Bot 交互的用户
    }
  },
  "bindings": [{
    "guildId": "Guild ID",  // ← 路由规则：哪个服务器
    "peer": { "id": "Channel ID" }  // ← 路由规则：哪个频道
  }]
}
```

---

## 第一步：创建 Discord Bot

### 1.1 创建应用

1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 **New Application**，输入名称（如 "MyAssistant"）
3. 记下 **Application ID**（也叫 Client ID，后面生成邀请链接要用）

### 1.2 配置 Bot

左侧菜单选择 **Bot**：

1. 点击 **Reset Token** 获取 Bot Token
   > ⚠️ **Token 只显示一次，立即复制保存到安全的地方！**

2. **关闭 Public Bot**（重要！）
   ```
   Public Bot: OFF  ← 关闭后只有你能邀请此 Bot
   ```

3. 开启 **Privileged Gateway Intents**：
   - ✅ **PRESENCE INTENT**（可选，用于显示在线状态）
   - ✅ **SERVER MEMBERS INTENT**（可选，用于获取成员信息）
   - ✅ **MESSAGE CONTENT INTENT**（必须！用于读取消息内容）

4. 点击页面底部 **Save Changes**

### 1.3 生成邀请链接

左侧选择 **OAuth2 → URL Generator**：

**Scopes**（勾选）：
- ✅ `bot`
- ✅ `applications.commands`（可选，用于 Slash 命令）

**Bot Permissions**（建议全选以下权限）：

| 权限 | 说明 |
|------|------|
| ✅ Administrator | 管理员权限（包含所有权限，最方便） |

或者如果不想给管理员权限，至少勾选：

| 权限 | 说明 |
|------|------|
| ✅ View Channels | 查看频道 |
| ✅ Send Messages | 发送消息 |
| ✅ Send Messages in Threads | 在帖子中发送消息 |
| ✅ Embed Links | 嵌入链接 |
| ✅ Attach Files | 附加文件 |
| ✅ Read Message History | 读取消息历史 |
| ✅ Add Reactions | 添加反应 |
| ✅ Use Slash Commands | 使用斜杠命令 |
| ✅ Mention Everyone | @所有人（可选） |
| ✅ Manage Messages | 管理消息（可选，用于删除消息） |

**生成的 URL 格式**：
```
https://discord.com/api/oauth2/authorize?client_id=你的CLIENT_ID&permissions=8&scope=bot%20applications.commands
```

> `permissions=8` 表示管理员权限

### 1.4 邀请 Bot 到服务器

1. 在浏览器打开生成的 URL
2. 选择你的服务器
3. 点击授权

---

## 第二步：获取 Discord ID

在 Discord 客户端开启开发者模式：

1. 设置 → 高级 → 开启 **开发者模式**

然后获取以下 ID：

| 需要获取的 ID | 操作 |
|--------------|------|
| 服务器 ID (Guild ID) | 右键服务器图标 → 复制服务器 ID |
| 频道 ID (Channel ID) | 右键频道名称 → 复制频道 ID |
| 你的用户 ID (User ID) | 右键自己头像 → 复制用户 ID |

---

## 第三步：配置 Discord 频道（安全配置重点！）

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
          "requireMention": true,
          "users": ["你的用户ID"]
        }
      }
    }
  }
}
```

### 安全配置详解

#### `groupPolicy: "allowlist"`

只允许白名单中的服务器使用 Bot。即使 Bot 被拉到其他服务器，也不会响应。

#### `guilds` 对象

```json
"guilds": {
  "服务器ID": {
    "requireMention": true,  // 必须 @Bot 才响应
    "users": ["用户ID"]      // 只有这些用户的消息会被处理
  }
}
```

- **只配置你自己的服务器 ID**：其他服务器的消息会被忽略
- **`requireMention: true`**（强烈建议）：
  - 设为 `true` 后，Bot **只会响应 @提及消息**
  - 普通消息（不带 @）会被完全忽略
  - 这是防止误触发的重要保护层
  - 即使是白名单用户，不 @ 也不会得到响应
- **`users` 数组**：只有列出的用户 ID 才能与 Bot 交互

#### `dm` 对象（私信配置）

```json
"dm": {
  "policy": "allowlist",
  "allowFrom": ["你的用户ID"]
}
```

只允许白名单中的用户发私信给 Bot。

### 安全防护总结

| 防护层 | 配置项 | 效果 |
|--------|--------|------|
| **服务器隐私** | 私有服务器 + 不启用发现 | 他人无法找到你的服务器 |
| **Bot 邀请** | Public Bot: OFF | 只有你能邀请 Bot |
| **服务器层** | `groupPolicy: "allowlist"` + `guilds` | 只响应指定服务器 |
| **用户层** | `guilds.*.users` | 只响应指定用户 |
| **触发层** | `requireMention: true` | **只有 @Bot 才响应**，普通消息忽略 |
| **私信层** | `dm.policy: "allowlist"` | 只允许指定用户私信 |

即使有人把 Bot 拉到别的服务器（如果 Public Bot 没关），由于：
1. 该服务器不在 `guilds` 白名单中
2. 该用户不在 `users` 白名单中

Bot 也**不会响应**任何消息。

---

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

---

## 第五步：配置频道绑定（重点！）

### 踩坑点 1：bindings 位置

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

### 踩坑点 2：peer 格式

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

---

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

---

## 第七步：测试

在 Discord 不同频道 **@机器人** 发消息：
- #main 频道 @Bot → main agent 响应
- #code 频道 @Bot → coder agent 响应
- #research 频道 @Bot → researcher agent 响应

> 由于配置了 `requireMention: true`，直接发消息不会触发响应，必须 @Bot。

---

## 常见问题

### Q: 配置报错 "Unrecognized key"
运行 `clawdbot doctor --fix` 自动移除无效键，然后检查配置格式。

### Q: Bot 不响应
1. 检查 `clawdbot status` 中 Discord 是否为 ON/OK
2. 确认 Bot 已被邀请到服务器
3. 确认你的用户 ID 在 `guilds.服务器ID.users` 列表中
4. 确认开启了 MESSAGE CONTENT INTENT
5. 如果配置了 `requireMention: true`，确保消息中 @了 Bot

### Q: 所有频道都由 main 响应
检查 bindings 中的 `peer.id` 是否与频道 ID 完全匹配。

### Q: Agent 之间能共享记忆吗？
不能。每个 agent 有独立的会话历史。但如果工作区有父子关系（如 `~/clawd` 包含 `~/clawd/coder`），父 agent 可以读取子目录的文件。

### Q: 别人能把我的 Bot 拉到其他服务器吗？
如果你关闭了 Public Bot（参见第一步 1.2），只有你能邀请 Bot。即使有人通过其他方式邀请了 Bot，由于 `groupPolicy: "allowlist"` 和 `guilds` 白名单的限制，Bot 也不会响应。

### Q: 别人在我的服务器里 @Bot 会被响应吗？
不会。只有 `guilds.服务器ID.users` 列表中的用户才会被响应。

---

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
          "requireMention": true,
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

---

## 安全检查清单

配置完成后，确认以下安全设置：

### Discord 端设置
- [ ] Discord Developer Portal 中 **Public Bot** 已关闭
- [ ] 服务器**未启用**社区功能（防止出现在公开列表）
- [ ] 服务器**未启用**服务器发现（防止被搜索到）
- [ ] 服务器邀请链接未公开分享

### Clawdbot 配置
- [ ] `groupPolicy` 设置为 `"allowlist"`
- [ ] `guilds` 中只包含你自己的服务器 ID
- [ ] `guilds.*.users` 中只包含你自己的用户 ID
- [ ] `guilds.*.requireMention` 设置为 `true`（Bot 只响应 @提及）
- [ ] `dm.policy` 设置为 `"allowlist"`
- [ ] `dm.allowFrom` 中只包含你自己的用户 ID

### 安全习惯
- [ ] Bot Token 没有泄露给任何人
- [ ] 配置文件不在公开的 Git 仓库中

---

## 参考链接

- [Moltbot 官方文档](https://docs.molt.bot)
- [多 Agent 路由](https://docs.molt.bot/concepts/multi-agent)
- [Discord 频道配置](https://docs.molt.bot/channels/discord)
- [Discord Developer Portal](https://discord.com/developers/applications)

---

*最后更新: 2026-01-30*
