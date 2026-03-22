# OpenClaw 基础知识

> 本教程介绍 OpenClaw 中 Agent（智能体）的概念、作用、如何配置多个 Agent，以及 Agent 之间如何协作。
> 消息通道：仅使用 **飞书（Feishu/Lark）**
> 基于 OpenClaw 官方文档编写，版本持续更新中。

---

## 一、核心概念：什么是 Agent？

在 OpenClaw 中，**Agent（智能体）** 是一个完整的大脑，具备：

| 组成部分 | 说明 |
|---------|------|
| **Workspace（工作区）** | 文件系统目录，存放 AGENTS.md、SOUL.md、USER.md 等人格和记忆文件 |
| **AgentDir（状态目录）** | 存放认证配置、模型注册表、每个 Agent 的独立配置 |
| **Session Store（会话存储）** | 聊天历史和路由状态，位于 `~/.openclaw/agents/<agentId>/sessions/` |
| **身份（Identity）** | 名称、头像、emoji、主题风格 |
| **技能（Skills）** | 每个 Workspace 下可有自己的技能目录 |

> **重要**：Agent 的 Workspace 是**默认工作目录**，不是硬沙箱。相对路径在 Workspace 内解析，但绝对路径仍可访问主机其他位置（除非启用沙箱）。

### 默认 Agent（单 Agent 模式）

默认情况下，OpenClaw 运行一个 Agent：
- `agentId` 默认为 **`main`**
- Workspace 位于 `~/.openclaw/workspace`
- 会话键为 `agent:main:<mainKey>`

---

## 二、Workspace 文件结构

每个 Agent 的 Workspace 是它的"家"，包含以下文件：

```
~/.openclaw/workspace/          # 工作区根目录
├── AGENTS.md        # 操作规则（每次会话开头加载）
├── SOUL.md          # 人格、语气、边界定义
├── USER.md          # 用户信息（称呼方式、时区、偏好）
├── IDENTITY.md      # Agent 身份（名字、emoji、头像）
├── TOOLS.md         # 本地工具笔记（不含权限控制）
├── HEARTBEAT.md     # 心跳任务清单（保持简短）
├── BOOT.md          # Gateway 重启时执行的开机清单
├── BOOTSTRAP.md     # 首次启动时的引导仪式（完成后删除）
└── memory/          # 记忆目录
    └── YYYY-MM-DD.md  # 每日记忆日志
```

### 各文件作用

**AGENTS.md** — 操作规则
- 加载时机：每个会话开始时
- 内容：规则、优先级、"如何行为"的细节

**SOUL.md** — 灵魂定义
- 加载时机：每个会话开始时
- 内容：人格、语气、边界

**USER.md** — 用户档案
- 加载时机：每个会话开始时
- 内容：用户是谁、如何称呼

**IDENTITY.md** — Agent 身份
- 内容：名称、emoji、头像、风格

**HEARTBEAT.md** — 心跳任务清单
- 保持简短，避免 token 消耗

**memory/YYYY-MM-DD.md** — 每日记忆日志
- 建议在会话开始时读取今天和昨天的日志

**MEMORY.md** — 长期记忆（可选）
- 仅在主会话（私人对话）中加载
- 从每日日志中提炼精华，存储长期记忆

---

## 三、配置多个 Agent

### 3.1 使用向导快速创建

```bash
# 创建新的隔离 Agent
openclaw agents add coding
openclaw agents add research

# 查看所有 Agent
openclaw agents list

# 查看 Agent 绑定关系
openclaw agents list --bindings
```

### 3.2 手动配置文件

在 `~/.openclaw/openclaw.json` 中配置：

```json5
{
  agents: {
    list: [
      {
        id: "main",                              // Agent ID（唯一标识）
        name: "主助手",                           // 显示名称
        workspace: "~/.openclaw/workspace-main", // 工作区路径
        agentDir: "~/.openclaw/agents/main/agent", // 状态目录
        default: true,                            // 默认 Agent
        identity: {
          name: "小助手",
          emoji: "🤖",
          avatar: "avatars/avatar.png"
        }
      },
      {
        id: "coding",                             // 专门的编程 Agent
        name: "编程助手",
        workspace: "~/.openclaw/workspace-coding",
        agentDir: "~/.openclaw/agents/coding/agent"
      },
      {
        id: "research",                           // 研究专用 Agent
        name: "研究员",
        workspace: "~/.openclaw/workspace-research",
        agentDir: "~/.openclaw/agents/research/agent"
      }
    ]
  }
}
```

### 3.3 为每个 Agent 设置不同模型

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "日常助手",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"      // 快速响应
      },
      {
        id: "deep",
        name: "深度工作",
        workspace: "~/.openclaw/workspace-deep",
        model: "anthropic/claude-opus-4-6"        // 高质量推理
      }
    ]
  }
}
```

---

## 四、路由绑定：将消息分配给不同 Agent（飞书）

**绑定（Binding）**是 OpenClaw 将入站消息路由到对应 Agent 的核心机制。

### 4.1 路由优先级（最具体优先）

```
1. peer 匹配（精确 私聊/群组 ID）
2. parentPeer 匹配（线程继承）
3. accountId 匹配（频道账户）
4. channel 级别匹配（accountId: "*"）
5. 回退到默认 Agent
```

### 4.2 按具体用户路由（私聊）

```json5
{
  bindings: [
    // 特定飞书用户 Open ID 路由到研究员 Agent
    {
      agentId: "research",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxxxxxxxxxxxxxxx" }
      }
    },
    // 其他飞书用户路由到主助手
    { agentId: "main", match: { channel: "feishu" } }
  ]
}
```

### 4.3 按群组路由

```json5
{
  bindings: [
    // 特定飞书群组路由到编程 Agent
    {
      agentId: "coding",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_xxxxxxxxxxxxxxxx" }
      }
    },
    // 其他私聊路由到主助手
    { agentId: "main", match: { channel: "feishu" } }
  ]
}
```

### 4.4 命令行管理绑定

```bash
# 添加绑定（将飞书私聊绑定到指定 Agent）
openclaw agents bind --agent coding --bind feishu

# 将特定用户绑定到 Agent（需指定 Open ID）
openclaw agents bind --agent research --bind feishu:ou_xxxxxxxxxxxxxxxx

# 列出所有绑定
openclaw agents bindings

# 查看特定 Agent 的绑定
openclaw agents bindings --agent main

# JSON 格式输出
openclaw agents bindings --json

# 移除绑定
openclaw agents unbind --agent research --bind feishu

# 移除所有绑定
openclaw agents unbind --agent research --all
```

---

## 五、飞书渠道配置

### 5.1 创建飞书应用

1. 访问 [飞书开放平台](https://open.feishu.cn/app) 登录
2. 点击**创建企业自建应用**，填写名称和描述
3. 在**凭证与基础信息**中复制：
   - **App ID**（格式：`cli_xxx`）
   - **App Secret**

4. 配置权限（批量导入）：
```json
{
  "scopes": {
    "tenant": [
      "im:message",
      "im:message:send_as_bot",
      "im:message:readonly",
      "im:message.p2p_msg:readonly",
      "im:message.group_at_msg:readonly",
      "im:resource",
      "im:chat.members:bot_access",
      "contact:user.employee_id:readonly"
    ]
  }
}
```

5. 在**应用能力 > 机器人**中启用机器人能力

6. 在**事件订阅**中：
   - 选择**使用长连接接收事件**（WebSocket，无需公网地址）
   - 添加事件：`im.message.receive_v1`

7. 发布应用并等待审批

### 5.2 配置 OpenClaw

**方式一：向导配置（推荐）**
```bash
openclaw channels add
```
选择 **Feishu**，输入 App ID 和 App Secret。

**方式二：手动配置**

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",              // 默认：未知用户需要配对码
      accounts: {
        main: {
          appId: "cli_xxxxxxxxxxxxxxxx",
          appSecret: "xxxxxxxxxxxxxxxx",
          botName: "我的 AI 助手"
        }
      }
    }
  }
}
```

### 5.3 飞书多账号配置

```json5
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxxxxxxxxxxxxxxx",
          appSecret: "xxxxxxxxxxxxxxxx",
          botName: "主助手"
        },
        work: {
          appId: "cli_yyyyyyyyyyyyyyyy",
          appSecret: "yyyyyyyyyyyyyyyy",
          botName: "工作助手",
          enabled: false
        }
      }
    }
  }
}
```

### 5.4 访问控制配置

**私聊策略（dmPolicy）：**
```json5
{
  "channels": {
    "feishu": {
      "dmPolicy": "allowlist",
      "allowFrom": ["ou_user1", "ou_user2"]    // 允许的用户 Open ID 列表
    }
  }
}
```

**群组策略：**
```json5
{
  "channels": {
    "feishu": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["oc_group1", "oc_group2"],  // 允许的群组 ID 列表
      "groups": {
        "oc_group1": {
          "requireMention": false                  // 无需 @机器人
        }
      }
    }
  }
}
```

---

## 六、Agent 身份设置

### 6.1 从 IDENTITY.md 加载

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

### 6.2 手动设置

```bash
openclaw agents set-identity --agent main --name "小助手" --emoji "🤖" --avatar avatars/avatar.png
```

### 6.3 IDENTITY.md 文件格式

```markdown
# IDENTITY.md - Who Am I?

- **Name:** 小助手
- **Creature:** AI 助手
- **Vibe:** 聪明、可靠、高效
- **Emoji:** 🤖
- **Avatar:** avatars/avatar.png
```

---

## 七、Agent 之间的协作

OpenClaw 支持多种 Agent 协作方式：

### 7.1 子 Agent（Sub-Agent）— 并行任务执行

子 Agent 是从主 Agent 生成的后台运行，在独立会话中执行，完成后自动将结果通报回请求者的聊天。

**使用方式 1：斜杠命令**

```
/subagents list          # 列出当前会话的子 Agent
/subagents spawn <任务>   # 启动子 Agent
/subagents kill <id>     # 停止子 Agent
/subagents log <id>      # 查看子 Agent 日志
```

**使用方式 2：sessions_spawn 工具**

```json
{
  "task": "研究最新的 AI 技术动态并总结",
  "label": "research-ai",
  "runTimeoutSeconds": 300,
  "model": "claude-sonnet-4-5"
}
```

**典型使用场景：**

- 并行执行多个研究任务，不阻塞主会话
- 后台执行耗时的代码编译或测试
- 长任务分段处理

**配置子 Agent 模型：**

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "claude-sonnet-4-5",    // 默认子 Agent 模型
        thinking: "medium",              // 默认思考级别
        maxConcurrent: 8,               // 最大并发数
        archiveAfterMinutes: 60          // 归档时间
      }
    }
  }
}
```

**子 Agent 工具策略：**

默认情况下，子 Agent 拥有除会话工具外的所有工具：

```json5
{
  tools: {
    subagents: {
      tools: {
        deny: ["gateway", "cron"],      // 禁用危险工具
        // allow: ["read", "exec"]      // 或设置白名单模式
      }
    }
  }
}
```

**子 Agent 认证：**
- 按 `agentId` 解析认证，不是会话类型
- 主 Agent 的认证配置作为回退合并

---

### 7.2 Agent-to-Agent 直接通信

允许不同 Agent 之间直接发送消息：

```json5
{
  tools: {
    agentToAgent: {
      enabled: true,
      allow: ["main", "coding", "research"]           // 允许通信的 Agent 列表
    }
  }
}
```

---

### 7.3 sessions_send — 跨会话消息

通过会话键向其他 Agent 发送消息：

```json
{
  "sessionKey": "agent:coding:subagent:xxx-xxx",
  "message": "任务完成了，汇总报告已生成"
}
```

**使用场景：**
- 主 Agent 协调多个子 Agent 的工作
- 将任务结果传递给下一个处理阶段

---

### 7.4 sessions_spawn — Agent 变体

为主 Agent 创建专门的执行变体（使用不同模型/角色）：

```json
{
  "task": "你是代码审查专家，请审查以下代码...",
  "runtime": "subagent",
  "model": "claude-opus-4-6",
  "mode": "run"
}
```

---

## 八、安全与隔离

### 8.1 沙箱配置

每个 Agent 可以有独立的沙箱配置：

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }         // 不启用沙箱
      },
      {
        id: "guest",
        workspace: "~/.openclaw/workspace-guest",
        sandbox: {
          mode: "all",                    // 始终沙箱化
          scope: "agent"                  // 每个 Agent 一个容器
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history"],
          deny: ["write", "exec", "browser", "canvas"]
        }
      }
    ]
  }
}
```

### 8.2 群组聊天的工具限制

为特定群组设置严格工具策略：

```json5
{
  agents: {
    list: [
      {
        id: "team",
        name: "团队助手",
        workspace: "~/.openclaw/workspace-team",
        groupChat: {
          mentionPatterns: ["@团队助手", "@team"]
        },
        tools: {
          allow: ["read", "exec", "sessions_list", "sessions_history"],
          deny: ["write", "edit", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  }
}
```

---

## 九、完整配置示例

### 场景：飞书多角色 Agent 系统

```json5
{
  // === Agent 定义 ===
  agents: {
    list: [
      {
        id: "main",
        name: "主助手",
        default: true,
        workspace: "~/.openclaw/workspace-main",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "coding",
        name: "编程专家",
        workspace: "~/.openclaw/workspace-coding",
        model: "anthropic/claude-opus-4-6",
        sandbox: { mode: "all", scope: "agent" }
      },
      {
        id: "research",
        name: "研究员",
        workspace: "~/.openclaw/workspace-research",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "team",
        name: "团队助手",
        workspace: "~/.openclaw/workspace-team",
        groupChat: {
          mentionPatterns: ["@团队助手", "@team-bot"]
        },
        tools: {
          allow: ["read", "exec", "sessions_list", "sessions_history"]
        }
      }
    ]
  },

  // === 路由绑定（飞书） ===
  bindings: [
    // 特定用户路由到研究 Agent
    {
      agentId: "research",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxxxxxxxxxxxxxxx" }
      }
    },
    // 团队群组路由到团队 Agent
    {
      agentId: "team",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_xxxxxxxxxxxxxxxx" }
      }
    },
    // 其他私聊默认到主助手
    { agentId: "main", match: { channel: "feishu" } }
  ],

  // === 飞书渠道配置 ===
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxxxxxxxxxxxxxxx"],
      accounts: {
        main: {
          appId: "cli_xxxxxxxxxxxxxxxx",
          appSecret: "xxxxxxxxxxxxxxxx",
          botName: "AI 助手"
        }
      },
      groups: {
        "oc_xxxxxxxxxxxxxxxx": {
          requireMention: false
        }
      }
    }
  },

  // === 子 Agent 配置 ===
  tools: {
    subagents: {
      tools: {
        deny: ["gateway", "cron"]
      }
    }
  }
}
```

---

## 十、工作流程示例

### 场景：并行研究和代码执行（飞书协作）

**用户请求：**"帮我研究 3 个 AI 框架并分别写一个 hello world 示例"

**主 Agent 协作流程（通过飞书）：**

```
用户在飞书向主助手发送请求
    │
    ├─► 子 Agent 1（研究）：研究 TensorFlow
    │      └─► sessions_spawn("研究 TensorFlow 框架")
    │
    ├─► 子 Agent 2（研究）：研究 PyTorch
    │      └─► sessions_spawn("研究 PyTorch 框架")
    │
    ├─► 子 Agent 3（研究）：研究 JAX
    │      └─► sessions_spawn("研究 JAX 框架")
    │
    └─► 主 Agent 等待结果汇总
            │
            ├─► 子 Agent 1 通报结果到飞书
            ├─► 子 Agent 2 通报结果到飞书
            ├─► 子 Agent 3 通报结果到飞书
            │
            └─► 主 Agent 汇总输出到飞书
```

**实现代码（Agent 内部）：**

```json
// 启动三个研究子 Agent
sessions_spawn({ task: "研究 TensorFlow，输出框架介绍和 hello world 示例" })
sessions_spawn({ task: "研究 PyTorch，输出框架介绍和 hello world 示例" })
sessions_spawn({ task: "研究 JAX，输出框架介绍和 hello world 示例" })
```

---

## 十一、获取飞书 ID

### 群组 ID（chat_id）

群组 ID 格式：`oc_xxxxxxxxxxxxxxxx`

**方法 1（推荐）**
1. 启动 Gateway 并在群组中 @机器人
2. 运行 `openclaw logs --follow`，查找 `chat_id`

**方法 2**
使用飞书 API 调试工具列出群组

### 用户 ID（open_id）

用户 ID 格式：`ou_xxxxxxxxxxxxxxxx`

**方法 1（推荐）**
1. 启动 Gateway 并私聊机器人
2. 运行 `openclaw logs --follow`，查找 `open_id`

**方法 2**
查看配对请求中的用户 Open ID：
```bash
openclaw pairing list feishu
```

---

## 十二、常用命令速查

| 命令 | 说明 |
|------|------|
| `openclaw agents list` | 列出所有 Agent |
| `openclaw agents list --bindings` | 查看 Agent 绑定关系 |
| `openclaw agents add <name>` | 创建新 Agent |
| `openclaw agents bind --agent <id> --bind feishu` | 添加飞书路由绑定 |
| `openclaw agents unbind --agent <id> --all` | 移除所有绑定 |
| `openclaw agents set-identity --agent <id> --from-identity` | 从 IDENTITY.md 加载身份 |
| `openclaw agents delete <id>` | 删除 Agent |
| `openclaw agent --agent <id> --message "<text>"` | 直接向指定 Agent 发消息 |
| `openclaw channels add` | 添加飞书渠道 |
| `openclaw gateway restart` | 重启 Gateway |
| `/subagents list` | 查看当前会话的子 Agent |
| `/subagents spawn <task>` | 启动子 Agent |
| `/subagents kill <id>` | 停止子 Agent |

---

## 十三、路径速查

```
~/.openclaw/                    # 状态根目录
├── openclaw.json               # 主配置文件
├── agents/                     # Agent 状态目录
│   └── <agentId>/
│       ├── agent/              # agentDir（认证、模型配置）
│       │   └── auth-profiles.json
│       └── sessions/           # 会话存储
│           └── *.json
├── workspace/                 # 默认工作区
├── credentials/               # 渠道凭证
│   └── feishu/                # 飞书凭证
└── skills/                    # 共享技能
```

---

## 十四、最佳实践

1. **身份分离**：不同 Agent 使用不同飞书机器人，避免消息混淆
2. **权限分级**：团队/访客群组限制工具权限，核心 Agent 保持完整权限
3. **模型选择**：
   - 日常对话 → Sonnet（快速、低成本）
   - 深度推理 → Opus（高质量）
   - 子 Agent → 较便宜的模型节省成本
4. **会话隔离**：不同 Agent 有独立会话存储，互不干扰
5. **记忆管理**：定期将 daily memory 提炼到 MEMORY.md，保持长期记忆精简
6. **群组路由**：使用 `oc_xxx` 格式的群组 ID 精确绑定特定群组到指定 Agent

---

## 十五、飞书渠道 dmPolicy 说明

| 值 | 行为 |
|---|------|
| `"pairing"` | **默认。** 未知用户收到配对码，需要审批 |
| `"allowlist"` | 仅 `allowFrom` 列表中的用户可以聊天 |
| `"open"` | 允许所有用户 |
| `"disabled"` | 禁用私聊 |

---

> 官方文档：https://docs.openclaw.ai
> 飞书开放平台：https://open.feishu.cn/app
> 社区 Discord：https://discord.com/invite/clawd
> 技能市场：https://clawhub.com
