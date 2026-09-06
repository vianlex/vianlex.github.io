---
title: "Pi Agent 使用教程"
linkTitle: "Pi Agent 使用教程"
weight: 20
description: "Pi Agent 安装、使用与公告系统实战"
---

# Pi Agent 使用教程

**Pi**（Pi Agent / Pi Coding Agent）是一个开源、极简的终端 AI 编码 agent。核心理念：把工作流适配到 Pi 上，而不是被庞大平台绑定。本教程讲「怎么用」，并以开发一个**公告系统**为例。

## 一、安装与认证

```bash
# 推荐：安装脚本（自动拉 Node 与 Pi）
curl -fsSL https://pi.dev/install.sh | sh

# 或：npm 全局安装
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

> `--ignore-scripts` 是因为 Pi 的依赖生命周期脚本在 npm 全局安装时不需要，关掉可避免奇怪的 postinstall 失败。
> npm scope 是当前（2026 年）官方发布渠道——`@earendil-works/pi-coding-agent`；老 scope `@mariozechner/pi-coding-agent` 与 `@styrene-lab/pi-coding-agent` 都是更早版本/分叉，新装请以 https://pi.dev 为准。

认证两种方式：

```bash
export ANTHROPIC_API_KEY=sk-ant-...   # 方式一：API Key
pi /login                             # 方式二：订阅登录（浏览器授权）
```

## 二、基本使用

### 启动交互模式

> 以下都是**启动 Pi 进程时的命令行参数**（在终端里执行），不是会话命令——进程启动后使命就结束了。

```bash
pi    # 打开新的交互模式对话（默认）
pi -c     # 从最近的一次会话，进入交互模式对话
pi -r    # 选择某个历史会话，进入交互模式对话
```

### 非交互模式命令

> 非交互模式即 `print` 模式，通过 `-p` 参数表示一次性输出，跑完直接退出，不进入 TUI 交互对话。

```bash
pi -p "总结这个代码库"          # print 模式，一次性输出
cat README.md | pi -p "总结"  # print 模式，支持 stdin 管道
```


### 启动时，指定对话模型

> 启动交互模式时，通过参数`--model` 直接指定模型，任务作为首条消息，进 TUI 继续对话。

```bash
pi --model openai/gpt-4o "帮我重构这段代码"
pi --model sonnet:high "解决这个复杂问题"   # 带 thinking level
```


### 模式对比

| 命令 | 模式 | 输出给谁 |
|------|------|--------|
| `pi "帮我总结代码仓库"` | 交互 TUI | 人（终端里看着） |
| `pi -p "帮我总结代码仓库"` | print | 人（一次性纯文本输出） |
| `pi --mode json "帮我总结代码仓库"` | json | 程序/脚本（JSONL 事件流） |
| `pi --mode rpc` | rpc | 进程间 RPC 协议集成 |

简单记：`json` 模式 = 给机器读的；`-p` = 给人看的；默认交互 = 边聊边看。


### 默认工具

`read` / `write` / `edit` / `bash` 四件套，没有 MCP、Sub-Agent、Plan Mode、Build In Todo 等等能力，需要更多能力靠安装扩展来实现。

## 三、会话交互

以下是**进入交互界面后**在会话中的操作，与上面「基本使用」里启动进程的命令行参数不同。

### 输入前缀

| 前缀 | 作用 |
|------|------|
| `!` | 执行命令，输出进模型上下文（AI 看得到） |
| `!!` | 执行命令，输出不进上下文（只有你看得到） |
| `@` | 引用文件/路径，把内容作为上下文 |
| `/` | 会话命令（见下节表格） |

```bash
!npm run dev        # AI 能看到运行输出
!!git status        # 只有你自己看得到输出
```

> 记法：`!` 我看、模型也看；`!!` 只有我看。

此外还支持**截图反馈**：先用系统截图工具截好图（如 Windows `Win+Shift+S`），再回 Pi 里粘贴，图片会作为消息附件发给 AI，配合一句文字（如「帮我看看这个报错」）让 AI 看图理解问题。快捷键：Windows 按 `Alt+V`、macOS/Linux 按 `Ctrl+V` 粘贴剪贴板图片。

> 注意：Windows Terminal 默认把 `Ctrl+V` 绑成「粘贴文本」会抢键，所以 Windows 上推荐用 `Alt+V`；macOS 在 Finder 里对图片按 `Cmd+C` 复制的是文件路径而非图片内容，需先用预览工具打开图片再复制，或直接把文件拖进终端。

### 会话命令（输入 `/` 触发）

| 命令 | 作用 |
|------|------|
| `/model` | 切换模型 |
| `/new` `/resume` | 新建/恢复会话 |
| `/tree` | 会话树（就地导航，回到任意历史节点） |
| `/fork` | 从某条用户消息派生新会话（保留原分支） |
| `/compact` | 手动压缩上下文 |
| `/session` | 查看 token/成本 |
| `/quit` | 退出 |

> 会话命令与「启动模式」里的命令不同：启动模式命令是**在终端里启动 Pi 进程时的命令行参数**（如 `pi -p`、`pi --mode json`），决定本次运行的模式，进程启动后使命就结束了；会话命令是**进入交互界面后**在对话里用 `/` 触发的，作用于当前会话运行过程中（如 `/tree`、`/fork`）。前者管「怎么启动」，后者管「启动后怎么操作」。

### 运行中插话：纠偏（Steering）与排队（Follow-up）

Pi 干活时你不用干等，随时可以插话。两种插话方式，对应「现在就要改方向」和「做完再说」：

**纠偏（Steering）**：按 `Enter` 发送。发现 AI 做偏了，当场掰回来——这条消息会在当前工具调用结束后、下一次模型调用前生效，并**打断**本轮剩余的工具调用。

> [!EXAMPLE] 输入 AI
> 后端不要用 Express，沿用现有 Next.js Route Handlers；数据库用 PGlite
{icon="fa-solid fa-keyboard"}

适合短指令：不要新增依赖 / 你找错目录了 / 先验证根因再动手 / 保留我未提交的改动。

**排队（Follow-up）**：按 `Alt+Enter`（macOS `Option+Enter`）发送。不打断 AI 现在干的活，等它把这轮全部做完，再接着处理你排的这条。

> [!EXAMPLE] 输入 AI
> 当前修复和测试全部完成后，再更新 docs/troubleshooting.md
{icon="fa-solid fa-keyboard"}

**一句话区分**（都在会话未结束、Pi 正在运行时发送输入指令）：

| 场景 | 用哪种 |
|------|--------|
| 现在方向就错了 | Steering（`Enter`） |
| 方向没错，做完还有下一件事 | Follow-up（`Alt+Enter`） |
| 整个任务都不该继续 | `Esc` 中止 |

辅助按键：`Esc` 中止并把排队消息退回编辑器；`Alt+↑` 取回排队消息编辑而不中止。

> Windows Terminal 默认把 `Alt+Enter` 绑成全屏快捷键，需先在终端设置里解除/重映射，或用 `/hotkeys` 核对当前绑定。

### TUI 与快捷键

**编辑与导航常用快捷键：**

| 快捷键 | 作用 |
|------|------|
| `Enter` | 发送消息 |
| `Shift+Enter` | 换行（`Ctrl+Enter` 于 WSL） |
| `Ctrl+W` | 向后删除一个单词 |
| `Ctrl+U` | 删除到行首 |
| `Ctrl+K` | 删除到行尾 |
| `Ctrl+A` / `Home` | 行首 |
| `Ctrl+E` / `End` | 行尾 |
| `↑` | 空行时浏览历史 |
| `Esc` | 取消补全 / 中止输出流 |
| `Ctrl+C` | 清空编辑器（再次按下退出） |
| `Ctrl+G` | 调用外部编辑器（`$VISUAL`/`$EDITOR`） |

**模型与显示切换：**

| 快捷键 | 作用 |
|------|------|
| `Ctrl+P` | 循环模型（受 `--models` 约束） |
| `Ctrl+O` | 展开 / 收起工具输出 |
| `Ctrl+T` | 切换思考块（thinking）可见性 |
| `Shift+Tab` | 循环思考级别 |

## 四、会话管理

会话默认以 JSONL 树状结构保存在 `~/.pi/agent/sessions/`（按工作目录组织）。

**启动时的会话选项：**

```bash
pi -c                        # --continue：继续最近会话
pi -r                        # --resume：浏览历史会话并选择
pi --session <id>            # 使用指定会话文件或 ID（支持部分 UUID）
pi --fork <id>               # 从指定会话 fork 出新会话
pi --name "重构登录"          # -n：启动时命名会话
pi --session-dir ./sessions  # 自定义会话存储目录
```

**交互模式中的分支与导出：**

| 命令 | 作用 |
|------|------|
| `/tree` | 在会话树中就地导航，回到任意历史节点继续 |
| `/fork` | 从某条用户消息派生新会话文件（保留原分支） |
| `/clone` | 复制当前分支为新会话 |
| `/export my-session.html` | 导出为 HTML |
| `/import session.jsonl` | 从 JSONL 导入恢复 |
| `/share` | 上传为私有 GitHub gist 并生成可分享链接 |

压缩是有损的，但完整历史保留在 JSONL 中，可用 `/tree` 随时回顾。

底部状态栏实时显示 token/缓存命中率/成本。

## 五、实战：开发一个公告系统

用 Pi 从零开发一个 Node.js（Express）公告系统，每一步给出你该说的话。

### 第 1 步：初始化项目

> [!EXAMPLE] 输入 AI
> 帮我初始化一个 Express 公告系统项目，使用 npm，入口 app.js
{icon="fa-solid fa-keyboard"}

Pi 会执行 `npm init`、安装 `express`，生成 `app.js` 和 `package.json`。

### 第 2 步：设计数据模型

> [!EXAMPLE] 输入 AI
> 设计公告数据模型：id、标题、内容、分类、置顶标志、
> 状态(草稿/已发布/已撤回)、发布人、发布时间、创建时间、更新时间。
> 用 JSON 文件做存储，放到 data/announcements.json
{icon="fa-solid fa-keyboard"}

### 第 3 步：生成接口

> [!EXAMPLE] 输入 AI
> 实现公告接口：
> 1. POST /announcements 新增
> 2. GET /announcements 分页查询（支持分类、状态、关键字过滤）
> 3. PUT /announcements/:id 修改
> 4. DELETE /announcements/:id 删除
> 5. POST /announcements/:id/publish 发布
> 6. POST /announcements/:id/revoke 撤回
{icon="fa-solid fa-keyboard"}

### 第 4 步：跑起来验证

> [!EXAMPLE] 输入 AI
> 启动服务，用 curl 测试新增、分页、发布、撤回接口
{icon="fa-solid fa-keyboard"}

Pi 执行 `node app.js`，写 curl 命令验证并报告结果。发现问题直接说：

> [!EXAMPLE] 输入 AI
> 分页没有按状态过滤，帮我修一下
{icon="fa-solid fa-keyboard"}

### 第 5 步：用会话树做方案对比

> [!EXAMPLE] 输入 AI
> 现在发布接口有两个方案：软删除 vs 状态字段流转。
> 先按状态字段实现，跑通后我 fork 回去试软删除方案
{icon="fa-solid fa-keyboard"}

跑通后输入 `/tree` 打开会话树，跳回实现前的节点 `/fork`，让 Pi 用软删除方案重做，对比两种实现。

### 第 6 步：脚本化批量操作

```bash
# 用 print 模式做一次性操作
cat data/announcements.json | pi -p "找出所有草稿状态的公告 id"

# 用 json 模式接入自己的脚本
pi --mode json "分析这个接口的边界情况" > analysis.jsonl
```

## 六、扩展生态

Pi 的核心刻意保持小，把"够用就行"之外的定制能力下放到五层扩展。先看全景，再分层讲。

| 层 | 形态 | 加载位置 | 触发方式 |
|---|---|---|---|
| **Sub-Agents** | Markdown + YAML frontmatter | `~/.pi/agent/agents/*.md` 或 `.pi/agents/*.md` | 子代理机制（需装扩展，Pi 内置没有） |
| **Skills** | Markdown（`SKILL.md`） | `~/.pi/agent/skills/` 或 `.pi/skills/` | `/skill:name` 显式 / 按 description 自动加载 |
| **Prompt Templates** | Markdown（带 frontmatter） | `~/.pi/agent/prompts/` 或 `.pi/prompts/` | `/模板名`（文件名即命令名） |
| **Themes** | JSON（51-token 配色） | `~/.pi/agent/themes/` 或 `.pi/themes/` | `/theme` 切换；修改文件即热重载 |
| **Extensions** | TypeScript 模块 | `~/.pi/agent/extensions/` 或 `.pi/extensions/` | 注册到 `~/.pi/agent/settings.json` |

> ⚠️ **第三方包以完整系统权限运行**，安装前先审源码。Pi 故意不做权限系统——靠容器化（Docker / Gondolin / OpenShell 见官方文档 Containerization 章节）兜底。

### 1. Sub-Agents（子代理）

> Pi 内置**没有** sub-agent——这是 Pi 与 Claude Code 等的核心差异之一（Pi 哲学：够用就行，不想要的都不默认塞）。需要的话装扩展。

最常见的实现：

```bash
pi install npm:@bytetrue/pi-subagent       # 推荐：极简、fresh/fork 双模式
pi install npm:@piotr-oles/pi-subagents    # 隔离会话、独立模型/工具
pi install npm:@rohaquinlop/pi-subagents   # pipeline + 运行时注册
pi install npm:@melihmucuk/pi-crew         # 多 agent 团队 + 实时状态条
```

每个 agent 是一个 Markdown 文件：

```markdown
<!-- ~/.pi/agent/agents/code-reviewer.md -->
---
name: code-reviewer            # 必填，主代理按这个名字调用
description: 读 diff 并找出具体 bug；提交前主动触发
model: anthropic/claude-sonnet-4-20250514
thinking: high
tools: read, grep, find, ls    # 内置工具白名单
---

你是一个代码审查子代理。逐文件检查最新 diff，输出
「文件路径 + 行号 + 问题 + 修复建议」列表。不写代码。
```

**frontmatter 字段：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | ✅ | 唯一标识，主代理按 `name` 调用 |
| `description` | ✅ | 主代理据此决定何时分派 |
| `model` | | `provider/model-id` 格式；省略则继承父会话模型 |
| `thinking` | | `off / minimal / low / medium / high / xhigh` |
| `tools` | | 内置工具白名单（`read, bash, edit, write, grep, find, ls`） |
| `interactive` | | 完成后保持会话可追问，默认 `false` |

**两种 context 模式：**

- **`spawn`**（默认）：子代理拿到 **纯任务字符串**「Task: ...」，**没有父会话上下文**——token 省、隔离干净，适合一次性的检索/审查/测试
- **`fork`**：子代理拿到 **父会话的 fork 快照 + 任务**——能看到此前的对话与读过的文件，适合「接着刚才讨论的事继续做」

**经验法则**：能从一句话讲清楚的任务用 `spawn`；需要参考刚才讨论、读过的文件、做的决策用 `fork`。

### 2. Skills（技能）

Markdown 包，文件名即技能名，`SKILL.md` 是入口：

```markdown
<!-- ~/.pi/agent/skills/code-review/SKILL.md -->
---
name: code-review
description: 审查当前 diff 的正确性与风格；commit 前主动触发
---

## 步骤
1. `git diff` 拿到改动
2. 逐文件扫描：边界、空指针、错误处理、可测性
3. 输出「文件:行 + 问题 + 建议」结构化列表
```

**触发方式：**

```text
/skill:code-review                 # 显式调用
/skill:code-review 当前分支改动   # 带参数

# 或让 Pi 自动按 description 决定——description 写得越具体，匹配越准
```

**description 写法的关键**（主代理在启动时加载所有 skill 的 description，按相关性决定加载哪个）：

> **写「什么场景下主动使用」**，而不是「这个 skill 能做什么」。比如 ❌「代码审查 skill」 / ✅「**代码提交前主动调用**，审查 diff 正确性与风格」。前者太宽、容易误触发；后者精准。

**两个查找路径：**

- **本机已有**：直接装包 `pi install npm:@aholbreich/agent-skills`（包内多 skill 时用 `pi install -l` 装到项目）
- **公共技能仓库**：`npx skills add <github 用户/仓库> --skill <skill 名>`

### 3. Prompt Templates（提示词模板）

文件名即命令名，frontmatter 描述 + 正文，`$@` 转发参数：

```markdown
<!-- ~/.pi/agent/prompts/quick-debug.md -->
---
description: 快速 debug 当前报错；用轻量模型 + REPL skill
model: claude-sonnet-4-20250514
skill: tmux
---

启动一个 Python REPL 会话，帮 debug：

$@
```

```text
/quick-debug 跑这个 case 时 OOM
```

> Skills vs Prompt Templates：
> - **Skills**——可复用的方法论，任意会话可触发；不带参数
> - **Prompt Templates**——固定流程工作流，自带参数（`$@`）；触发后是一整套动作
>
> 例：审查 PR 用 **code-review skill**（方法论）；/implement 加暗色模式 触发 3 个 sub-agent 链 → **prompt template**（流程）

### 4. Themes（主题）

```json
<!-- ~/.pi/agent/themes/dim.json -->
{
  "name": "dim",
  "vars": {
    "background": "#1a1b26",
    "foreground": "#c0caf5",
    "border": "#414868",
    "accent": "#7aa2f7",
    "muted": "#565f89"
  }
}
```

**51 个 token**（含 `toolTitle / error / warning / success / userMessage / assistantMessage / diffAdd / diffContext / ...`）覆盖 UI 全要素。文件保存即热重载。

```text
/theme dim
```

### 5. Extensions（TypeScript 扩展）

最强大的一层。TypeScript 模块，能做：

| 能力 | API |
|---|---|
| 订阅生命周期事件 | `pi.on("session_start", ...)` / `tool_call` / `agent_end` / ... |
| 注册 LLM 可调用的工具 | `pi.registerTool({ name, parameters, execute })` |
| 注册斜杠命令 | `pi.registerCommand("name", { description, handler })` |
| 注册键盘快捷键 / CLI flag | `pi.registerShortcut` / `pi.registerFlag` |
| 主动操作 agent | `pi.sendMessage` / `pi.setModel` / `pi.exec` |
| 自定义 LLM provider | `pi.registerProvider(...)`（OAuth 流程都可） |
| 自定义 TUI 渲染 | `renderCall` / `renderResult` 返回 `pi-tui` 组件 |
| 用户交互弹窗 | `ctx.ui.select / confirm / input / notify / custom` |

**最简骨架（一个工具）：**

```typescript
// ~/.pi/agent/extensions/jira-search.ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";

export default function (pi: ExtensionAPI) {
  // 1) 订阅事件
  pi.on("session_start", async (_event, ctx) => {
    ctx.ui.notify("Jira 扩展已加载", "info");
  });

  // 2) 拦截危险命令
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName === "bash" && event.input.command?.includes("rm -rf")) {
      const ok = await ctx.ui.confirm("危险操作", "确认执行 rm -rf？");
      if (!ok) return { block: true, reason: "用户阻止" };
    }
  });

  // 3) 注册 LLM 可调用的工具
  pi.registerTool({
    name: "jira_search",         // snake_case，LLM 按这个名字调用
    label: "Jira 搜索",          // TUI 显示名
    description: "在 Jira 项目中按关键词搜索 Issue",
    parameters: Type.Object({
      query: Type.String({ description: "搜索关键词" }),
      project: Type.Optional(Type.String({ description: "项目 Key" })),
    }),
    async execute(_toolCallId, params, _signal, _onUpdate, ctx) {
      const issues = await fetch(`https://your-jira/rest/api/2/search?jql=text~"${params.query}"`)
        .then(r => r.json());
      return {
        content: [{ type: "text", text: JSON.stringify(issues.issues) }],
        details: { count: issues.total },
      };
    },
  });

  // 4) 注册斜杠命令
  pi.registerCommand("checkpoint", {
    description: "把当前改动 stash 到 git",
    handler: async (_args, ctx) => {
      const msg = `pi-checkpoint-${Date.now()}`;
      await ctx.exec(`git stash push -m "${msg}"`);
      ctx.ui.notify(`已 stash: ${msg}`, "success");
    },
  });
}
```

**加载与调试：**

```bash
# 快速测试（一次性加载，不入库）
pi -e ./jira-search.ts

# 正式放到自动发现位置 → 用 /reload 热加载
cp jira-search.ts ~/.pi/agent/extensions/

# 加依赖（用到 npm 包时）
mkdir ~/.pi/agent/extensions/jira-search/
cd ~/.pi/agent/extensions/jira-search/
npm init -y
npm install node-fetch
# 然后把 jira-search.ts 改名 index.ts
```

### 6. Packages（包管理）

把上面 5 层任意组合打包分发：

```bash
# 装
pi install npm:@foo/pi-tools                 # npm 包
pi install npm:@foo/pi-tools@1.2.3           # 钉版本
pi install git:github.com/user/repo          # git
pi install git:github.com/user/repo@v1.2     # 钉 tag/commit
pi install -l npm:@foo/pi-tools              # 项目级（写到 .pi/npm/）
```

**自动发现 vs manifest**：扩展/skill/prompt/theme 只要放进 `~/.pi/agent/extensions/`、`skills/`、`prompts/`、`themes/` 这几个约定目录就**自动发现**，无需注册。需要在 npm 包里显式声明就写 `package.json`：

```json
{
  "name": "@me/my-pi-pack",
  "keywords": ["pi-package"],
  "pi": {
    "extensions": ["./extensions"],
    "skills":     ["./skills"],
    "prompts":    ["./prompts"],
    "themes":     ["./themes"]
  }
}
```

`keywords: ["pi-package"]` 是 Pi 识别分发包的标志。

**日常管理：**

```bash
pi list                       # 列出已装包
pi update                     # 更新 Pi 本身
pi update --all               # Pi + 所有包
pi update --extensions        # 仅扩展
pi update --models            # 刷新模型目录
pi remove npm:@foo/pi-tools   # 卸载
pi config                     # 启用/禁用各层
```

**钉版本陷阱**：`pi update --all` 会跳过钉了 `@v1.2` 这种 tag 的包；想升级钉版包需手动 `pi install git:...@新版本`。

### 7. Custom Models & Providers（高级）

**加一个模型**（已有 provider，但官方目录里没有）：

```jsonc
// ~/.pi/agent/models.json
{
  "my-custom-model": {
    "provider": "anthropic",
    "id": "claude-3-5-sonnet-20241022",
    "thinking": "low",
    "contextWindow": 200000
  }
}
```

**加一个自定义 provider**（新 API、OAuth 流程、第三方代理）通过扩展实现：

```typescript
pi.registerProvider("my-proxy", {
  baseUrl: "https://my-proxy.example.com/v1",
  apiKey: process.env.MY_PROXY_KEY,
  // OAuth 流式授权注册也支持
});
```

`/model` 选择列表会立刻出现新模型/新 provider。

---

> 官网：https://pi.dev
> 文档：https://pi.dev/docs/latest
