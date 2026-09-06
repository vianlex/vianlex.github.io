---
title: PiAgent 插件扩展教程
linkTitle: PiAgent 插件扩展教程
weight: 30
description: Pi 扩展生态深入教程——Sub-Agents、Skills、Prompt Templates、Themes、Extensions 五层全实战
---

> 📌 **与 [PiAgent 使用教程](.) 第六节的关系**
>
> 使用教程的「六、扩展生态」是**导览**——只介绍 5 层是什么、各自装哪条命令、能解决什么问题。
>
> **本文是深入教程**——逐层展开「完整写法 + 实战 demo + 调试排错 + 打包发布」。读本文前建议先通读使用教程第六节打底。

## 0. 5 层扩展全景图

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
                          │  10+ 能力 API   │
                          └────────┬────────┘
                                   │
                            ┌──────▼──────┐
                            │   Themes    │
                            │ JSON 配色   │
                            └─────────────┘
```

**关键设计哲学**：

- **Core 不做权限系统**——第三方包以系统权限运行，靠用户审计 + 容器化兜底
- **5 层独立**——你可以只装 Skills 不装 Sub-Agents，按需扩展
- **Markdown 优先**——前 4 层都用 Markdown 写，零代码门槛
- **TypeScript 是逃生舱**——前 4 层搞不定才上 Extensions

## 1. Sub-Agents 实战

### 1.1 选型矩阵

| 扩展包 | 隔离度 | 上下文继承 | 模型独立 | 实时进度 | 复杂链路 |
|---|---|---|---|---|---|
| `@bytetrue/pi-subagent`（推荐）| worktree 隔离 | spawn / fork 二选一 | ✅ | ✅ | ❌ |
| `@piotr-oles/pi-subagents` | 独立会话 | 父 fork | ✅ | ✅ | ✅ |
| `@rohaquinlop/pi-subagents` | 独立会话 | 父 fork | ✅ | ✅ | ✅ pipeline |
| `@melihmucuk/pi-crew` | 独立会话 | 父 fork | ✅ | ✅ 状态条 | ✅ 团队 |

> **初学者**：选 `@bytetrue/pi-subagent`——极简、spawn/fork 双模式覆盖 90% 场景。
> **要 pipeline**：选 `@rohaquinlop` 或 `@piotr-oles`。
> **多人协作模拟**：选 `@melihmucuk/pi-crew`。

### 1.2 装好第一步

```bash
pi install npm:@bytetrue/pi-subagent
# → ~/.pi/agent/agents/ 下多了 subagent.md
```

### 1.3 写一个完整 agent

```markdown
<!-- ~/.pi/agent/agents/test-writer.md -->
---
name: test-writer                       # 主代理按 name 调用
description: 为 Python 模块写 pytest 单测；用户说"帮我测一下 X"时主动触发
model: anthropic/claude-sonnet-4-20250514
thinking: medium
tools: read, bash, edit, write, grep, find, ls
---

你是一个 pytest 单测编写员。流程：

1. `read` 目标模块（必读）
2. `find` 同目录下已存在的 `test_*.py`，参考命名与风格
3. 用 `bash` 跑 `python -c "import <mod>"` 确认可导入
4. **写单测**：每个公共函数至少 1 个 happy path + 1 个边界用例
5. **跑测试**：`bash pytest -v --tb=short <test_file>`，必须全绿
6. 报告：测试文件路径 + 用例数 + 覆盖率（如装了 `pytest-cov`）

不要改源代码。仅测试文件。
```

**字段详解：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | ✅ | 唯一标识，主代理 `name` 调用 |
| `description` | ✅ | 主代理据此决定何时分派——**写「什么场景下主动使用」**而非「能做什么」 |
| `model` |  | `provider/model-id`；省略继承父模型 |
| `thinking` |  | `off / minimal / low / medium / high / xhigh` |
| `tools` |  | 内置工具白名单（`read, bash, edit, write, grep, find, ls`） |
| `interactive` |  | 完成后保持会话可追问，默认 `false` |
| `skills` |  | 该 agent 自动加载的 skill 列表（v2026 新增） |
| `tags` |  | 用于 `@bytetrue` 等扩展做 agent 分类过滤 |

### 1.4 两种 context 模式对比

```text
# spawn 模式：纯任务字符串，无父上下文
/subagent:task spawn "为 src/auth.py 写 pytest"

# fork 模式：父会话的 fork 快照 + 任务
/subagent:task fork "把刚才讨论的 OAuth 重构也加上测试"
```

**经验法则**：

- **spawn**：任务能一句话讲清、要 token 省、隔离干净 → 一次性检索/审查/测试
- **fork**：需要参考刚才讨论的代码/读过的文件/做过的决策 → 「接着刚才那件事继续做」

### 1.5 调试三板斧

```bash
# 1) 看 agent 注册情况
/subagent:list

# 2) 看 description 是否会被主代理识别
/subagent:preview test-writer

# 3) 干跑一次，看 agent 实际拿到的 prompt 与 tool 白名单
/subagent:dry-run test-writer "示例任务"
```

### 1.6 实战：自动测试覆盖工作流

```text
你: 帮我给 src/ 下所有 Python 模块补 pytest 单测
主代理:（识别 description → 分派 test-writer，spawn 模式）
test-writer:（逐文件 → 写 test_*.py → pytest 跑全绿）
test-writer: 报告：8 个模块，47 个用例，覆盖率 78%
```

如果 `pytest-cov` 装了，主代理会基于覆盖率判断要不要再补一轮。

## 2. Skills 实战

### 2.1 Skill vs Sub-Agent 选哪个

| 维度 | Skill | Sub-Agent |
|---|---|---|
| 触发 | 主代理**当前上下文**直接执行 | 主代理**分派**给独立会话 |
| 上下文开销 | 共享父会话 token | 独立会话，token 隔离 |
| 适用 | 「我想现在做 X」（方法论） | 「帮我去把 Y 干完」（委派） |
| 例子 | 代码审查、提交前检查 | 写测试、跑迁移、独立分析 |

### 2.2 写 `commit-review` skill

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
3. 输出结构化报告：

```
## 审查报告

### ✅ 通过
- path/to/file.py: 功能完整，错误处理得当

### ⚠️ 建议修改
- path/to/file.py:42 — 空指针风险，`x` 可能为 None
- path/to/file.py:67 — 函数名 `do_stuff` 不够具体

### 🚫 必须改
- path/to/file.py:30 — 硬编码 API key，移到环境变量
```

4. **不要自动修改代码**——只输出报告，让用户决定
```

**description 写法对比：**

> ❌ 「审查代码的 skill」——太宽，容易每次都触发
> ✅ 「**代码提交前主动调用**，审查未提交 diff 的正确性、风格、安全性」——精准，触发时机明确

### 2.3 写 `release-notes` skill

```markdown
<!-- ~/.pi/agent/skills/release-notes/SKILL.md -->
---
name: release-notes
description: 发布新版本时主动调用；从上次 tag 到当前 commit 生成 CHANGELOG
---

## 步骤

1. `git describe --tags --abbrev=0` 拿到上一个 tag
2. `git log <tag>..HEAD --oneline` 拿到所有 commit
3. 按 Conventional Commits 分类：
   - `feat:` → ### ✨ 新功能
   - `fix:` → ### 🐛 修复
   - `perf:` → ### ⚡ 性能
   - `refactor:` → ### 🔨 重构
   - `docs:` / `test:` / `chore:` → 合并到「其他变更」
4. 输出到 CHANGELOG.md，按 Keep a Changelog 格式
```

### 2.4 让 Pi 自动加载的 description 写法

主代理在会话开始时会读所有 skill 的 description 一次，匹配时再加载完整 `SKILL.md`。**description 是触发器，要写得像「用户会说的话」**：

```text
✅ 用户说「commit 前帮我看下」  → commit-review 触发
✅ 用户说「要发版了」          → release-notes 触发
✅ 用户说「我有个 bug」         → 啥 skill 也不触发（bug 不是 skill，是任务）
```

## 3. Prompt Templates 实战

### 3.1 模板 vs Skill 区别

- **Skill**——可复用的方法论，任意会话可触发；通常不带参数
- **Prompt Template**——固定流程工作流，自带参数（`$@`）；触发后是一整套动作

### 3.2 写 `daily-standup` 模板

```markdown
<!-- ~/.pi/agent/prompts/daily-standup.md -->
---
description: 生成每日站会报告；从 git log 与日历聚合
model: claude-sonnet-4-20250514
skill: tmux
---

生成今日站会：

$@

格式：
**昨日完成**：<从 git log 昨日 commits 提取>
**今日计划**：<从日历今日事件提取>
**阻塞**：<从 issues/todo 标记 @blocked 提取>
```

### 3.3 写 `pr-prep` 模板

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

### 3.4 高级：模板组合 skill

```markdown
<!-- ~/.pi/agent/prompts/code-review-batch.md -->
---
description: 批量审查多个 PR；并行启动 code-review skill
agent: code-reviewer          # 指定 agent
skills: [code-review, pr-stats]
---

启动并发审查：

$@

每个 PR 跑一次 /skill:code-review，结果汇总成表格：
| PR | 作者 | 阻塞问题 | 建议数 | 状态 |
```

## 4. Themes 实战

### 4.1 51 个 token 全解

```json
<!-- ~/.pi/agent/themes/dim.json -->
{
  "name": "dim",
  "vars": {
    "background":       "#1a1b26",
    "backgroundAlt":    "#16161e",
    "foreground":       "#c0caf5",
    "border":           "#414868",
    "borderFocused":    "#7aa2f7",
    "accent":           "#7aa2f7",
    "muted":            "#565f89",

    "toolTitle":        "#bb9af7",
    "toolOutput":       "#9ece6a",
    "toolError":        "#f7768e",
    "toolBg":           "#1f2335",

    "userMessage":      "#7dcfff",
    "assistantMessage": "#c0caf5",
    "systemMessage":    "#565f89",

    "diffAdd":          "#9ece6a",
    "diffRemove":       "#f7768e",
    "diffContext":      "#565f89",

    "error":            "#f7768e",
    "warning":          "#e0af68",
    "success":          "#9ece6a",
    "info":             "#7dcfff",

    "syntaxKeyword":    "#bb9af7",
    "syntaxString":     "#9ece6a",
    "syntaxNumber":     "#ff9e64",
    "syntaxComment":    "#565f89",
    "syntaxFunction":   "#7aa2f7",
    "syntaxVariable":   "#c0caf5",
    "syntaxOperator":   "#89ddff",
    "syntaxPunct":      "#c0caf5"
  }
}
```

### 4.2 调试配色

```bash
/theme dim               # 切换
/theme                    # 列表

# 热重载：编辑 dim.json 保存，/theme dim 即生效
```

## 5. Extensions 实战（最强大的一层）

### 5.1 完整 demo：`git-helper` 扩展

这个扩展演示 8 类核心 API：订阅事件、注册工具、注册斜杠命令、注册快捷键、自定义 UI、用户确认弹窗、自定义 TUI 渲染、操作 agent。

```typescript
// ~/.pi/agent/extensions/git-helper/index.ts
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { Type } from "typebox";
import { Text } from "@earendil-works/pi-tui";

export default function (pi: ExtensionAPI) {
  // ============ 1) 订阅事件 ============
  pi.on("session_start", async (_event, ctx) => {
    const branch = await ctx.exec("git branch --show-current").catch(() => "无 git");
    ctx.ui.notify(`当前分支：${branch.output.trim()}`, "info");
  });

  // 拦截危险命令
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName !== "bash") return;
    const cmd = event.input.command as string;
    if (/rm\s+-rf\s+\//.test(cmd)) {
      const ok = await ctx.ui.confirm(
        "危险操作",
        `确认执行：${cmd}\n\n这会删除系统根目录！`,
      );
      if (!ok) return { block: true, reason: "用户阻止 rm -rf /" };
    }
    if (/sudo\s/.test(cmd)) {
      const ok = await ctx.ui.confirm(
        "提权操作",
        `确认执行：${cmd}`,
      );
      if (!ok) return { block: true, reason: "用户阻止 sudo" };
    }
  });

  // ============ 2) 注册 LLM 可调用的工具 ============
  pi.registerTool({
    name: "git_log",
    label: "Git 日志",
    description: "查看 git 提交历史；支持分支/tag/作者过滤",
    parameters: Type.Object({
      range: Type.Optional(Type.String({ description: "分支/tag/range" })),
      author: Type.Optional(Type.String({ description: "作者邮箱/姓名" })),
      limit: Type.Optional(Type.Number({ description: "最多 N 条" })),
    }),
    async execute(_id, params, _signal, _update, ctx) {
      const parts = ["git", "log", "--oneline", "--decorate"];
      if (params.range) parts.push(params.range);
      if (params.author) parts.push(`--author=${params.author}`);
      if (params.limit) parts.push(`-n`, String(params.limit));

      const result = await ctx.exec(parts.join(" "));
      return {
        content: [{ type: "text", text: result.output }],
        details: { count: result.output.split("\n").length },
      };
    },
  });

  // ============ 3) 注册斜杠命令 ============
  pi.registerCommand("checkpoint", {
    description: "把当前改动 stash 到 git",
    handler: async (_args, ctx) => {
      const msg = `pi-checkpoint-${Date.now()}`;
      await ctx.exec(`git stash push -m "${msg}"`);
      ctx.ui.notify(`已 stash：${msg}`, "success");
    },
  });

  pi.registerCommand("restore", {
    description: "恢复最近的 checkpoint",
    handler: async (_args, ctx) => {
      const r = await ctx.exec("git stash list | head -1");
      if (!r.output.trim()) {
        ctx.ui.notify("没有 checkpoint", "warning");
        return;
      }
      const hash = r.output.match(/pi-checkpoint-(\d+)/)?.[1];
      await ctx.exec(`git stash pop`);
      ctx.ui.notify(`已恢复 checkpoint ${hash}`, "success");
    },
  });

  // ============ 4) 注册键盘快捷键 ============
  pi.registerShortcut("ctrl+shift+g", {
    description: "打开 git status 面板",
    handler: async (ctx) => {
      const r = await ctx.exec("git status --short");
      ctx.ui.notify(r.output || "工作区干净", r.output ? "info" : "success");
    },
  });

  // ============ 5) 自定义 TUI 渲染 ============
  pi.on("tool_call", async (event, ctx) => {
    if (event.toolName === "bash") {
      const cmd = (event.input.command as string).slice(0, 60);
      return {
        renderCall: () =>
          Text(`🔧 bash: ${cmd}${cmd.length >= 60 ? "…" : ""}`, 0, 0),
      };
    }
  });

  // ============ 6) 自定义 provider ============
  // 假设你的公司有内部 LLM 代理
  // pi.registerProvider("my-proxy", {
  //   baseUrl: "https://llm-proxy.corp.com/v1",
  //   apiKey: process.env.MY_PROXY_KEY,
  //   models: [
  //     { id: "internal-gpt", name: "Internal GPT", contextWindow: 128000 }
  //   ],
  // });
}
```

### 5.2 加载与调试

```bash
# 1) 快速测试（一次性加载，不入库）
pi -e ./git-helper/index.ts

# 2) 正式放到自动发现位置
mkdir -p ~/.pi/agent/extensions/git-helper/
mv git-helper/ ~/.pi/agent/extensions/
# 然后在会话里 /reload 热加载

# 3) 加依赖（用到 npm 包时）
cd ~/.pi/agent/extensions/git-helper/
npm init -y
npm install node-fetch           # 示例
# 把 index.ts 改成 default export + 引用即可
```

### 5.3 TypeBox 参数校验完整示例

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
  dueDate: Type.Optional(Type.String({ format: "date-time" })),
  tags: Type.Array(Type.String(), { maxItems: 10 }),
});

pi.registerTool({
  name: "create_todo",
  parameters: TodoItem,
  async execute(_id, params, _signal, _update, ctx) {
    // params 在此处类型完全推导为 TodoItem 类型
    const r = await fetch("https://api.example.com/todos", {
      method: "POST",
      body: JSON.stringify(params),
    });
    return { content: [{ type: "text", text: `已创建：${params.title}` }] };
  },
});
```

TypeBox 把 JSON Schema 与 TS 类型统一——LLM 拿 JSON Schema 校验入参，你拿 TS 类型自动补全。

## 6. 调试与排错

### 6.1 三种调试入口

```bash
# 1) 看所有扩展/skill/prompt/theme 加载情况
pi --debug

# 2) 干跑扩展（不入库）
pi -e ./git-helper/index.ts

# 3) 热加载（修改 ~/.pi/agent/extensions/* 后）
/reload
```

### 6.2 日志位置

```text
~/.pi/agent/logs/
  ├─ session_2026-09-06T09-12-34.log   # 会话日志
  ├─ extension_git-helper.log          # 扩展日志
  └─ errors.log                        # 错误聚合
```

### 6.3 常见排错

| 现象 | 排查 |
|---|---|
| skill 不触发 | `/skill:list` 看 description；改写得更具体 |
| 扩展不加载 | `pi --debug` 看注册结果；路径必须在约定目录 |
| 类型错误 | `pi -e ./ext.ts` 干跑，看 TS 错误 |
| 工具调用失败 | TypeBox schema 放宽，或换更小模型 |
| 拦截不生效 | `ctx.ui.confirm` 返回值检查；某些事件不支持 block |

### 6.4 VS Code attach

```bash
# 在 VS Code 里 F5 → Node.js: Attach to Process → 选 pi 进程
# 断点打在扩展代码里，LLM 调用扩展时即停
```

## 7. 打包与发布

### 7.1 本地路径包（团队内）

```bash
# 直接装本地路径
pi install file:~/work/my-pi-pack
```

### 7.2 Git 远程包

```bash
# 公开仓库
pi install git:github.com/your-org/pi-pack

# 私有仓库
pi install git:git@github.com:your-org/pi-pack.git

# 钉 tag
pi install git:github.com/your-org/pi-pack@v1.2.0
```

### 7.3 npm 发布（最通用）

```bash
mkdir pi-pack
cd pi-pack
npm init -y
npm pkg set keywords="[\"pi-package\"]"
npm pkg set pi.extensions="./extensions"
npm pkg set pi.skills="./skills"
npm pkg set pi.prompts="./prompts"
npm pkg set pi.themes="./themes"

mkdir extensions skills prompts themes
# 把刚才写的 md / ts 放进去

npm login
npm publish --access public

# 别人装：
pi install npm:@your-namespace/pi-pack
```

**`pi-package` keyword 是关键**——Pi 据此识别为分发包而非普通 npm 包。

### 7.4 版本管理

```bash
pi list                    # 已装包
pi update                  # Pi 本体
pi update --all            # Pi + 所有包
pi update --extensions     # 仅扩展
pi update --models         # 刷新模型目录
pi remove npm:@foo/pi-pack # 卸载
pi config                  # 启用/禁用各层
```

**钉版本陷阱**：`pi update --all` 会跳过钉了 `@v1.2` tag 的包；想升级需手动 `pi install git:...@新版本`。

### 7.5 公开市场：Pi Hub

[pi.dev/hub](https://pi.dev/hub) 是 Pi 的官方扩展市场——`/skill`、`/prompt` 市场可一搜就装，无需记命令。包作者可提交 PR 到 [github.com/pgsty/pi-hub](https://github.com/pgsty/pi-hub)。

## 8. 安全

### 8.1 第三方包威胁模型

Pi 故意**没有**权限系统——`ctx.exec` 直接拿系统 shell 执行。这意味：

> ⚠️ **第三方包以完整用户权限运行。安装前必须审源码。**

### 8.2 审计清单（装包前 60 秒）

```text
[ ] 1) 看 package.json：依赖是否合理、是否含可疑 postinstall
[ ] 2) 看 TypeScript 文件：有没有 ctx.exec 调外网？调用了哪些工具？
[ ] 3) 看 Markdown 文件：有没有引导 LLM 执行危险命令的提示？
[ ] 4) 看 keywords：是不是 "pi-package"？冒充标签的包别装
[ ] 5) 看 commit 历史：作者是否可信？最近是否有可疑 commit？
[ ] 6) 看 issues/discussions：有没有投诉？
```

### 8.3 容器化兜底

不想审计每个包？跑 Pi 整个在容器里：

```bash
# Docker 兜底
docker run -it --rm \
  -v $(pwd):/work \
  -v ~/.pi:/root/.pi \
  node:22 \
  pi

# Gondolin（Pi 官方 sandbox，v2026 推荐）
npm install -g gondolin
gondolin run pi

# OpenShell（多 agent 隔离）
pi install npm:@openshell/agent-sandbox
```

容器坏了不影响宿主。

### 8.4 拦截器模板

如果你用不可信包，至少在自己的扩展里加一层拦截：

```typescript
// ~/.pi/agent/extensions/safety-net/index.ts
pi.on("tool_call", async (event, ctx) => {
  const cmd = (event.input.command as string) || "";
  const dangerous = [
    /rm\s+-rf\s+\//,
    /sudo\s/,
    /curl.*\|\s*(?:bash|sh)/,
    /:\(\)\s*{\s*:\|:&\s*};:/,    // fork bomb
    /chmod\s+-R\s+777/,
    />\s*\/etc\//,
  ];
  for (const re of dangerous) {
    if (re.test(cmd)) {
      const ok = await ctx.ui.confirm(
        "⚠️ 危险命令",
        `检测到危险模式：${re}\n命令：${cmd}\n\n是否继续？`,
      );
      if (!ok) return { block: true, reason: `safety-net 拦截 ${re}` };
    }
  }
});
```

> 📌 **官方容器化文档**：[pi.dev/docs/latest/security](https://pi.dev/docs/latest/security)

---

## 下一步

- 通读 → [PiAgent 使用教程](.) 第六节打底
- 看更多 demo → [Pi Hub](https://pi.dev/hub)
- 跟 Claude Code 对比 → [ClaudeCode 使用教程](claudecode使用教程/)
- 跟 OpenCode 对比 → [OpenCode 使用教程](opencode使用教程/)