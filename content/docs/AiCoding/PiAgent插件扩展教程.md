---
title: PiAgent 插件扩展教程
linkTitle: PiAgent 插件扩展教程
weight: 30
description: Pi 扩展生态专业教程——五层架构、Sub-Agents/Skills/Prompts/Themes/Extensions 全实战 + 常用插件使用说明
---

> 📌 **阅读前提**：先通读 [PiAgent 使用教程](.) 了解基本用法。本文聚焦**扩展生态**——从架构原理到五层扩展的完整写法，再到常用插件的安装与使用。

## 一、什么是 Pi 的扩展生态

Pi 的核心刻意保持极简——默认只有 `read / write / edit / bash` 四个工具，**没有** MCP、**没有** Sub-Agent、**没有** Plan Mode。这不是功能缺失，而是设计哲学：**够用就行，不想要的都不默认塞进来**。

所有"额外能力"都通过扩展生态补齐，共分五层：

| 层 | 形态 | 门槛 | 能做什么 |
|---|---|---|---|
| **Sub-Agents** | Markdown + YAML | 零代码 | 把任务分派给专职子代理 |
| **Skills** | Markdown（`SKILL.md`） | 零代码 | 可复用的方法论/流程 |
| **Prompt Templates** | Markdown（带 frontmatter） | 零代码 | 固定流程 + 带参数的命令 |
| **Themes** | JSON | 零代码 | 终端配色（51 个 token） |
| **Extensions** | TypeScript | 需要代码 | 订阅事件/注册工具/命令/UI，最强的一层 |

**关键认知**：前四层都用 Markdown/JSON 写，零代码门槛；只有 Extensions 需要 TypeScript，它是"逃生舱"——前四层搞不定时才上。

## 二、扩展分层架构

```
                    ┌─────────────────────────────────────┐
                    │  Pi Core（极简，~4000 LOC）          │
                    │  read · write · edit · bash         │
                    │  无 MCP / 无 Sub-Agent / 无 Plan    │
                    └──────────────┬──────────────────────┘
                                   │
                ┌──────────────────┼──────────────────┐
                │                  │                  │
          ┌─────▼─────┐     ┌──────▼──────┐    ┌──────▼──────┐
          │Sub-Agents │     │   Skills    │    │   Prompts   │
          │（MD+YAML）│     │  SKILL.md   │    │  .md 模板   │
          │任务分派   │     │  可复用方法 │    │  带 $@ 参数 │
          └─────┬─────┘     └──────┬──────┘    └──────┬──────┘
                │                  │                  │
                └──────────────────┼──────────────────┘
                                   │
                          ┌────────▼────────┐
                          │   Extensions    │
                          │  TypeScript 模块│
                          │  事件/工具/命令 │
                          └────────┬────────┘
                                   │
                            ┌──────▼──────┐
                            │   Themes    │
                            │ JSON 配色   │
                            └─────────────┘
```

**设计要点**：

- **Core 不做权限系统**——第三方包以系统权限运行，靠用户审计 + 容器化兜底
- **五层独立**——可以只装 Skills 不装 Sub-Agents，按需扩展
- **约定目录自动发现**——文件放进约定目录即生效，无需注册

## 三、快速上手：安装 · 查看 · 常用命令

### 安装

**① 装扩展包**

| 源 | 命令 |
|---|---|
| npm | `pi install npm:@foo/pi-tools` |
| git | `pi install git:github.com/user/repo` |
| 本地路径 | `pi install file:~/work/my-pack` |
| 项目级（不污染全局） | `pi install -l npm:@foo/tools` |
| 钉版本 | npm 加 `@1.2.3`、git 加 `@v1.2` |

**② 装单个 skill**

```bash
npx skills add <github用户/仓库> --skill <skill名>
```

**③ 约定目录自动发现**（文件放进去即生效）

| 类型 | 全局目录 | 项目目录 |
|---|---|---|
| Extensions | `~/.pi/agent/extensions/` | `.pi/extensions/` |
| Skills | `~/.pi/agent/skills/{name}/SKILL.md` | `.pi/skills/{name}/SKILL.md` |
| Prompts | `~/.pi/agent/prompts/` | `.pi/prompts/` |
| Themes | `~/.pi/agent/themes/` | `.pi/themes/` |
| Agents（子代理） | `~/.pi/agent/agents/*.md` | `.pi/agents/*.md` |

### 查看

| 想查什么 | 命令 |
|---|---|
| 已装的包 | `pi list` |
| 已装的 skills | `/skill:list` |
| 已注册的子代理 | `/agents`（需装 subagent 扩展） |
| 可用主题 | `/theme`（无参数即列出） |
| 各层启用/禁用状态 | `pi config` |
| 扩展加载情况 | `pi --debug`（启动时打印注册结果） |
| 扩展日志 | `cat ~/.pi/agent/logs/extension_*.log` |

### 常用命令速查

```text
# 包管理
pi install npm:...        # 装（npm: / git: / file: 三种源）
pi list                   # 列出已装
pi update                 # 更新 Pi 本体
pi update --all           # Pi + 所有包
pi update --extensions    # 仅扩展包
pi update --models        # 刷新模型目录
pi remove npm:@foo/pkg    # 卸载
pi config                 # 启用/禁用各层

# 会话内（/ 触发）
/reload                   # 热加载扩展/skill 改动
/skill:name               # 触发 skill
/模板名                    # 触发 prompt 模板
/theme <name>             # 切换主题

# 扩展开发
pi -e ./my-ext.ts         # 干跑测试扩展（一次性加载，不入库）
pi --debug                # 调试模式启动
```

## 四、Sub-Agents（子代理）

### 4.1 核心概念

Pi 内置**没有** sub-agent。要使用子代理，先装扩展（见第九节"常用插件"）。子代理的本质是：**把一个专注任务分派给独立的子会话**，子会话有自己的上下文窗口和模型/工具配置，做完把结果带回父会话。

### 4.2 Agent 定义文件

每个 agent 是一个 Markdown 文件，YAML frontmatter + 系统提示词正文：

```markdown
<!-- ~/.pi/agent/agents/test-writer.md -->
---
name: test-writer                       # 唯一标识，主代理按 name 调用
description: 为 Python 模块写 pytest 单测；用户说"帮我测一下 X"时主动触发
model: anthropic/claude-sonnet-4-20250514
thinking: medium
tools: read, bash, edit, write, grep, find, ls
maxTurns: 50
---

你是一个 pytest 单测编写员。流程：

1. `read` 目标模块（必读）
2. `find` 同目录下已存在的 `test_*.py`，参考命名与风格
3. 用 `bash` 跑 `python -c "import <mod>"` 确认可导入
4. **写单测**：每个公共函数至少 1 个 happy path + 1 个边界用例
5. **跑测试**：`bash pytest -v --tb=short <test_file>`，必须全绿
6. 报告：测试文件路径 + 用例数 + 覆盖率

不要改源代码。仅测试文件。
```

**frontmatter 字段**：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | ✅ | 唯一标识，无空格、用连字符 |
| `description` | ✅ | 主代理据此决定何时分派——**写「什么场景下主动使用」** |
| `model` | | `provider/model-id`；省略继承父模型 |
| `thinking` | | `off / minimal / low / medium / high / xhigh / max` |
| `tools` | | 逗号分隔工具白名单（`read, bash, edit, write, grep, find, ls`） |
| `skills` | | 该 agent 加载的 skill 列表 |
| `maxTurns` | | 最大轮数限制（默认 50） |
| `interactive` | | 完成后保持会话可追问，默认 `false` |
| `context` | | 是否继承父会话上下文（`true`/`false`） |

**发现优先级**（同名冲突时后者覆盖）：项目 `.pi/agents/` > 用户 `~/.pi/agent/agents/` > 扩展内置。

### 4.3 内置 agent 角色

主流 subagent 扩展都内置一套通用角色，开箱即用：

| Agent | 用途 | 典型工具 |
|---|---|---|
| `scout` | 快速代码库侦察：相关文件、入口、数据流 | read, grep, find, ls |
| `researcher` | 联网/文档调研，带来源的研究简报 | read, grep, bash |
| `planner` | 从上下文产出具体实现计划（只读不改） | read, grep, find, ls |
| `worker` | 实现代码，验证并升级未批准决策 | read, write, edit, bash |
| `reviewer` | 代码审查 + 小修，对照任务/计划/测试 | read, bash, grep |
| `oracle` | 行动前的第二意见，挑战假设不改代码 | read, grep, ls |
| `delegate` | 轻量通用委托，行为接近父会话 | 继承默认 |

**经验法则**：搞懂代码前用 `scout`，信任外部事实前用 `researcher`，大改动前用 `planner`，实现用 `worker`，检查用 `reviewer`，决策本身有风险时用 `oracle`。

### 4.4 分派方式（自然语言为主）

装好 subagent 扩展后，**不需要学命令**，直接用自然语言：

```text
Use reviewer to review this diff.
Ask oracle for a second opinion on my current plan.
Run parallel reviewers: one for correctness, one for tests, one for complexity.
Use scout to understand the auth flow, then have planner turn that into a plan.
```

Pi 会自动决定调用哪个 agent、单个还是并行、是否 chain。

### 4.5 上下文模式

| 模式 | 行为 | 适用 |
|---|---|---|
| **isolated**（默认） | 不复制父会话，只拿到任务字符串 | 一次性检索/审查/测试，token 省 |
| **main / fork** | 传入父会话上下文 | 「接着刚才讨论的事继续做」 |

> 具体命令因扩展而异（`pi-subagents` 用 `/run`，`pi-sub-agent` 用工具参数 `context`），见第九节。

## 五、Skills（技能）

### 5.1 Skill vs Sub-Agent

| 维度 | Skill | Sub-Agent |
|---|---|---|
| 触发 | 主代理**当前上下文**直接执行 | 主代理**分派**给独立会话 |
| 上下文开销 | 共享父会话 token | 独立会话，token 隔离 |
| 适用 | 「我想现在做 X」（方法论） | 「帮我去把 Y 干完」（委派） |
| 例子 | 代码审查、提交前检查 | 写测试、跑迁移、独立分析 |

### 5.2 写一个 Skill

```markdown
<!-- ~/.pi/agent/skills/commit-review/SKILL.md -->
---
name: commit-review
description: 代码提交前主动调用；审查未提交 diff 的正确性、风格、安全性
---

## 步骤

1. `git diff --staged` 拿到暂存区改动（无暂存则 `git diff`）
2. 逐文件检查：
   - **正确性**：边界条件、空指针、错误处理
   - **风格**：命名、注释、复杂度
   - **安全**：硬编码密钥、SQL 注入、XSS
3. 输出结构化报告（✅ 通过 / ⚠️ 建议 / 🚫 必须改）
4. **不要自动修改代码**——只输出报告，让用户决定
```

**description 写法是关键**——主代理会话开始时读所有 skill 的 description 一次，匹配时才加载完整 `SKILL.md`：

> ❌ 「审查代码的 skill」——太宽，容易每次误触发
> ✅ 「**代码提交前主动调用**，审查未提交 diff 的正确性、风格、安全性」——精准

**触发方式**：

```text
/skill:commit-review                 # 显式调用
/skill:commit-review 当前分支改动   # 带参数
# 或让 Pi 按 description 自动匹配
```

## 六、Prompt Templates（提示词模板）

### 6.1 模板 vs Skill

- **Skill**——可复用方法论，任意会话可触发，通常不带参数
- **Prompt Template**——固定流程工作流，自带参数（`$@`），触发后是一整套动作

### 6.2 写一个模板

```markdown
<!-- ~/.pi/agent/prompts/pr-prep.md -->
---
description: 准备 PR；本地跑测试 + 写 PR 描述
model: claude-sonnet-4-20250514
---

为当前分支准备 PR：

$@

步骤：
1. `git diff main...HEAD --stat` 看改动范围
2. `bash npm test` 跑测试（全绿才能继续）
3. `bash npm run lint` 跑 lint
4. 生成 PR 描述（What / Why / How to test / Screenshots）
5. `gh pr create --draft` 创建 draft PR
```

**frontmatter 可扩展字段**（可选，由扩展如 `pi-prompt-template-model` 支持）：

| 字段 | 作用 |
|---|---|
| `model` | 执行时切换到指定模型，结束后恢复 |
| `skill` | 加载指定 skill 上下文 |
| `thinking` | 设置思考级别 |

### 6.3 触发

```text
/pr-prep                          # 无参数
/pr-prep 实现用户登录功能         # $@ 收到参数
```

## 七、Themes（主题）

### 7.1 51 个 color token

主题是 JSON 文件，覆盖终端 UI 全要素：

```json
<!-- ~/.pi/agent/themes/dim.json -->
{
  "name": "dim",
  "vars": {
    "background":       "#1a1b26",
    "foreground":       "#c0caf5",
    "border":           "#414868",
    "accent":           "#7aa2f7",
    "muted":            "#565f89",
    "toolTitle":        "#bb9af7",
    "toolOutput":       "#9ece6a",
    "toolError":        "#f7768e",
    "userMessage":      "#7dcfff",
    "assistantMessage": "#c0caf5",
    "diffAdd":          "#9ece6a",
    "diffRemove":       "#f7768e",
    "error":            "#f7768e",
    "warning":          "#e0af68",
    "success":          "#9ece6a"
  }
}
```

**切换与热重载**：`/theme dim` 切换，编辑 JSON 保存即热重载。

## 八、Extensions（TypeScript 扩展）

最强大的一层，能做事件订阅、工具注册、命令、快捷键、UI 渲染、自定义 provider。

### 8.1 完整骨架

```typescript
// ~/.pi/agent/extensions/git-helper/index.ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";
import { Text } from "@earendil-works/pi-tui";

export default function (pi: ExtensionAPI) {
  // 1) 订阅事件
  pi.on("session_start", async (_event, ctx) => {
    const branch = await ctx.exec("git branch --show-current");
    ctx.ui.notify(`当前分支：${branch.output.trim()}`, "info");
  });

  // 2) 拦截危险命令
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName !== "bash") return;
    const cmd = event.input.command as string;
    if (/rm\s+-rf\s+\//.test(cmd)) {
      const ok = await ctx.ui.confirm("危险操作", `确认执行：${cmd}`);
      if (!ok) return { block: true, reason: "用户阻止 rm -rf /" };
    }
  });

  // 3) 注册 LLM 可调用的工具
  pi.registerTool({
    name: "git_log",
    label: "Git 日志",
    description: "查看 git 提交历史；支持分支/tag/作者过滤",
    parameters: Type.Object({
      range: Type.Optional(Type.String({ description: "分支/tag/range" })),
      limit: Type.Optional(Type.Number({ description: "最多 N 条" })),
    }),
    async execute(_id, params, _signal, _update, ctx) {
      const result = await ctx.exec(`git log --oneline --decorate ${params.range ?? ""} -n ${params.limit ?? 20}`);
      return { content: [{ type: "text", text: result.output }] };
    },
  });

  // 4) 注册斜杠命令
  pi.registerCommand("checkpoint", {
    description: "把当前改动 stash 到 git",
    handler: async (_args, ctx) => {
      const msg = `pi-checkpoint-${Date.now()}`;
      await ctx.exec(`git stash push -m "${msg}"`);
      ctx.ui.notify(`已 stash：${msg}`, "success");
    },
  });

  // 5) 注册键盘快捷键
  pi.registerShortcut("ctrl+shift+g", {
    description: "打开 git status 面板",
    handler: async (ctx) => {
      const r = await ctx.exec("git status --short");
      ctx.ui.notify(r.output || "工作区干净", r.output ? "info" : "success");
    },
  });
}
```

### 8.2 加载与调试

```bash
# 1) 快速测试（一次性加载，不入库）
pi -e ./git-helper/index.ts

# 2) 正式放到自动发现位置 → /reload 热加载
cp -r git-helper/ ~/.pi/agent/extensions/

# 3) 加依赖（用到 npm 包时）
cd ~/.pi/agent/extensions/git-helper/
npm init -y
npm install node-fetch
```

### 8.3 TypeBox 参数校验

```typescript
import { Type } from "typebox";

const TodoItem = Type.Object({
  id: Type.String({ format: "uuid" }),
  title: Type.String({ minLength: 1, maxLength: 200 }),
  priority: Type.Union([
    Type.Literal("low"),
    Type.Literal("medium"),
    Type.Literal("high"),
  ]),
  tags: Type.Array(Type.String(), { maxItems: 10 }),
});

pi.registerTool({
  name: "create_todo",
  parameters: TodoItem,
  async execute(_id, params, _signal, _update, ctx) {
    // params 类型完全推导为 TodoItem
    return { content: [{ type: "text", text: `已创建：${params.title}` }] };
  },
});
```

TypeBox 把 JSON Schema 与 TS 类型统一——LLM 拿 JSON Schema 校验入参，你拿 TS 类型自动补全。

## 九、常用插件使用说明

> 本节是重点。按"用途分类 + 安装 + 使用"逐个说明。安装前务必先审源码（Pi 包以系统权限运行）。

### 9.1 Sub-Agents 类

#### ⭐ pi-subagents（最主流，推荐）

Pi 生态下载量最大的子代理扩展（作者 nicopreme，月下载 36 万+），支持单代理、并行、链式、后台、评审循环等完整工作流。

```bash
pi install npm:pi-subagents
```

**装完即用**——不用建 agent、不用学命令，直接自然语言：

```text
Use reviewer to review this diff.
Ask oracle for a second opinion on my current plan.
Run parallel reviewers: one for correctness, one for tests, one for complexity.
Have worker implement this approved plan, then run reviewers and apply the feedback.
```

**内置 agent**：`scout / researcher / planner / worker / reviewer / context-builder / oracle / delegate`。

**常用命令**：

| 命令 | 作用 |
|---|---|
| `/run agent task` | 单代理执行 |
| `/chain a "t1" -> b "t2"` | 顺序执行，每步接上一步输出 |
| `/parallel a "t1" -> b "t2"` | 并行执行 |
| `/agents` | 打开 Agents 管理器 |
| `/subagents-fleet` | 实时查看子代理运行状态 |
| `/council` | 多模型顾问团对比决策 |

**推荐工作流**：`clarify → scout → worker → fresh reviewers → worker`（澄清 → 侦察 → 实现 → 独立评审 → 修正）。

#### pi-sub-agent（mazli，内置 9 agent）

另一个成熟的子代理扩展，内置 9 个专职 agent，每个任务跑独立子进程（`pi --mode json -p --no-session`）。

```bash
pi install npm:pi-sub-agent
```

**内置 agent**：`scout / planner / worker / reviewer / debugger / verifier / security-auditor / docs-writer / refactorer`。

**三种模式**：

```text
# 单代理
Use the scout subagent to locate authentication entry points.

# 并行（最多 8 任务 / 4 并发）
{ "tasks": [{"agent":"scout","task":"Review models"},{"agent":"planner","task":"Review CLI"}] }

# 链式（最多 8 步，{previous} 接上一步输出）
{ "chain": [{"agent":"scout","task":"Find OAuth code"},{"agent":"planner","task":"Plan using: {previous}"}] }
```

**配置命令**：`/sub-agent-settings` 查看/编辑每个子代理的模型与思考级别。

#### @ryan_nookpi/pi-extension-subagent（异步）

异步子代理委托，支持 run/batch/chain/continuation，子代理无确认提示、后台运行。

```bash
pi install npm:@ryan_nookpi/pi-extension-subagent
```

```text
/sub:isolate worker implement the requested change and run tests
/sub:main worker continue this thread
```

### 9.2 全家桶类

#### pi-toolbox（17 扩展 + 11 主题 + 团队编排）

一站式扩展工具包，含 UI、工作流纪律、多代理编排、代码审查、安全审计五大类。

```bash
pi install npm:pi-toolbox        # 全局
pi install -l npm:pi-toolbox     # 项目级（可随 git 分享）
```

```bash
# 初始化：复制 agent 模板 / 团队编排 / 安全规则到项目 .pi/
npx pi-toolbox setup
```

**核心扩展**：

| 扩展 | 用途 |
|---|---|
| `damage-control` | 安全审计，拦截 `rm -rf`/`git reset --hard`/`DROP DATABASE` 等危险命令 |
| `agent-team` | 多代理调度 + 网格仪表盘 |
| `agent-chain` | 顺序管道编排（planner → builder → reviewer） |
| `cross-agent` | 扫描 `.claude/`/`.gemini/`/`.codex/` 复用它们的命令/skill/agent |
| `theme-cycler` | 主题循环（Ctrl+X / Ctrl+Q） |
| `session-replay` | `/replay` 会话历史时间线 |
| `go-review` | Go 代码审查（对照 100 Go Mistakes 清单） |
| `purpose-gate` | 会话开始时声明意图 |
| `tilldone` | 任务纪律系统 |

**11 个主题**：synthwave、catppuccin-mocha、cyberpunk、dracula、everforest、gruvbox、midnight-ocean、nord、ocean-breeze、rose-pine、tokyo-night。

**选择性加载**（只启用部分扩展）：

```json
{
  "packages": [
    {
      "source": "npm:pi-toolbox",
      "extensions": ["+extensions/minimal.ts", "+extensions/damage-control.ts"],
      "themes": [],
      "skills": []
    }
  ]
}
```

### 9.3 MCP 集成类

#### pi-mcp-adapter

Pi 默认不支持 MCP，此扩展让 agent 直接调用 MCP 服务器工具。

```bash
pi install npm:pi-mcp-adapter
```

在 agent 的 `tools` 字段加 `mcp:` 前缀：

```markdown
---
name: browser-agent
tools: read, bash, mcp:chrome-devtools
---

# mcp:server-name         → 该服务器全部工具
# mcp:server/tool_name    → 单个工具
```

### 9.4 实用小扩展

| 扩展 | 用途 | 安装 |
|---|---|---|
| `pi-history` | shift+↑/↓ 输入历史，跨会话保留 | `pi install npm:pi-history` |
| `pi-trust-git` | 按域名/用户名白名单自动信任项目 | `pi install npm:pi-trust-git` |
| `pi-recap` | 空闲 5 分钟后自动生成会话小结 | `pi install npm:pi-recap` |
| `pi-bash-audit` | bash 命令风险分级审计（低/中/高） | `pi install npm:pi-bash-audit` |

> 这些多为社区扩展，安装前在 [pi.dev/packages](https://pi.dev/packages) 查证最新版本与评价。

## 十、调试与排错

### 10.1 三种调试入口

```bash
pi --debug          # 看所有扩展/skill/prompt/theme 加载情况
pi -e ./ext.ts      # 干跑扩展（不入库）
/reload             # 热加载（改 ~/.pi/agent/extensions/* 后）
```

### 10.2 日志位置

```text
~/.pi/agent/logs/
  ├─ session_*.log          # 会话日志
  ├─ extension_*.log        # 扩展日志
  └─ errors.log             # 错误聚合
```

### 10.3 常见排错

| 现象 | 排查 |
|---|---|
| skill 不触发 | `/skill:list` 看 description；改写得更具体 |
| 扩展不加载 | `pi --debug` 看注册结果；路径必须在约定目录 |
| 子代理不工作 | 确认 subagent 扩展已装；`/agents` 看是否注册 |
| 类型错误 | `pi -e ./ext.ts` 干跑看 TS 错误 |
| 工具调用失败 | TypeBox schema 放宽，或换更小模型 |
| 拦截不生效 | `ctx.ui.confirm` 返回值检查；部分事件不支持 block |

## 十一、打包与发布

### 11.1 三种分发方式

```bash
# 本地路径（团队内）
pi install file:~/work/my-pi-pack

# Git 远程（公开/私有）
pi install git:github.com/your-org/pi-pack
pi install git:git@github.com:your-org/pi-pack.git
pi install git:github.com/your-org/pi-pack@v1.2.0

# npm（最通用）
pi install npm:@your-namespace/pi-pack
```

### 11.2 npm 发布

```bash
mkdir pi-pack && cd pi-pack
npm init -y
npm pkg set keywords="[\"pi-package\"]"      # 关键：Pi 据此识别分发包
npm pkg set pi.extensions="./extensions"
npm pkg set pi.skills="./skills"
npm pkg set pi.prompts="./prompts"
npm pkg set pi.themes="./themes"

mkdir extensions skills prompts themes
# 把 md / ts 放进去

npm login && npm publish --access public
```

**`pi-package` keyword 是关键**——Pi 据此识别为分发包而非普通 npm 包。

### 11.3 版本管理

```bash
pi list                    # 已装包
pi update                  # Pi 本体
pi update --all            # Pi + 所有包
pi update --extensions     # 仅扩展
pi remove npm:@foo/pi-pack # 卸载
pi config                  # 启用/禁用各层
```

**钉版本陷阱**：`pi update --all` 会跳过钉了 `@v1.2` tag 的包；升级需手动 `pi install ...@新版本`。

### 11.4 官方市场

[pi.dev/packages](https://pi.dev/packages) 是官方扩展市场——可搜索、查看下载量、直接复制安装命令。包作者可提交 PR 到 [github.com/pgsty/pi-hub](https://github.com/pgsty/pi-hub)。

## 十二、安全

### 12.1 威胁模型

Pi 故意**没有**权限系统——`ctx.exec` 直接拿系统 shell 执行：

> ⚠️ **第三方包以完整用户权限运行。安装前必须审源码。**

### 12.2 审计清单（装包前 60 秒）

```text
[ ] 1) 看 package.json：依赖是否合理、是否含可疑 postinstall
[ ] 2) 看 TypeScript 文件：有没有 ctx.exec 调外网？调用了哪些工具？
[ ] 3) 看 Markdown 文件：有没有引导 LLM 执行危险命令的提示？
[ ] 4) 看 keywords：是不是 "pi-package"？冒充标签的包别装
[ ] 5) 看 commit 历史：作者是否可信？最近是否有可疑 commit？
[ ] 6) 看 issues/discussions：有没有投诉？
```

### 12.3 容器化兜底

```bash
# Docker 兜底
docker run -it --rm -v $(pwd):/work -v ~/.pi:/root/.pi node:22 pi

# Gondolin（Pi 官方 sandbox）
npm install -g gondolin && gondolin run pi
```

容器坏了不影响宿主。

### 12.4 拦截器模板

```typescript
// ~/.pi/agent/extensions/safety-net/index.ts
pi.on("tool_call", async (event, ctx) => {
  const cmd = (event.input.command as string) || "";
  const dangerous = [
    /rm\s+-rf\s+\//,
    /sudo\s/,
    /curl.*\|\s*(?:bash|sh)/,
    /chmod\s+-R\s+777/,
    />\s*\/etc\//,
  ];
  for (const re of dangerous) {
    if (re.test(cmd)) {
      const ok = await ctx.ui.confirm("⚠️ 危险命令", `检测到：${re}\n命令：${cmd}\n\n是否继续？`);
      if (!ok) return { block: true, reason: `safety-net 拦截 ${re}` };
    }
  }
});
```

> 📌 **官方文档**：[pi.dev/docs/latest](https://pi.dev/docs/latest) · 扩展市场：[pi.dev/packages](https://pi.dev/packages)

---

## 下一步

- 通读 → [PiAgent 使用教程](.) 第六节打底
- 找插件 → [Pi 扩展市场](https://pi.dev/packages)
- 跟 Claude Code 对比 → [ClaudeCode 使用教程](claudecode使用教程/)
- 跟 OpenCode 对比 → [OpenCode 使用教程](opencode使用教程/)
