---
title: "Claude Code 使用教程"
linkTitle: "Claude Code 使用教程"
weight: 10
description: "Claude Code 安装、使用与公告系统实战"
---

# Claude Code 使用教程

Claude Code 是 Anthropic 官方的命令行 AI 编码工具，直接在终端里读代码、改代码、跑命令、提交 Git。本教程对应 **Claude Code 2.1.x**（2026 年至今的活跃版本）。

> **官方文档**：https://code.claude.com/docs  
> **完整命令清单**：https://code.claude.com/docs/en/cli-reference（始终反映当前版本）

## 一、安装与登录

### 安装

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# 包管理器
brew install --cask claude-code          # macOS Homebrew
winget install Anthropic.ClaudeCode      # Windows WinGet
npm install -g @anthropic-ai/claude-code # npm 全局
```

更新到最新版本：

```bash
claude update                 # 内置升级
npm update -g @anthropic-ai/claude-code   # npm 升级
```

### 登录

首次启动 `claude` 会引导浏览器登录订阅账户。用 API 计费则：

```bash
claude auth login --console    # 走 Console/API key
claude auth status             # 查看当前登录状态
claude auth logout             # 退出
```

## 二、基本使用

### 启动

```bash
cd 你的项目目录
claude                      # 交互模式
claude "修一下构建错误"        # 带初始提示启动
claude -p "解释这个函数"       # 一次性查询后退出（headless / print）
claude -c                    # 继续当前目录最近对话（--continue）
claude -r <id|name> "…"      # 恢复指定会话并可附带新任务（--resume）
claude --teleport            # 把当前云端会话拉到本地继续
tail -200 app.log | claude -p "有异常就告诉我"   # 管道输入
```

### 常用启动旗标

| 旗标 | 作用 |
|------|------|
| `--model <name>` | 指定模型 |
| `--effort low\|medium\|high\|max` | 设置思考深度 |
| `--permission-mode plan\|acceptEdits\|default` | 启动时设定 |
| `--add-dir <path>` | 临时增加可访问目录 |
| `--worktree` / `-w` | 在独立 git worktree 启动（隔离修改） |
| `--agent <name>` | 用指定的子 agent 启动 |
| `--bare` | 最简 headless（不加载 hooks/LSP/插件/skills），适合脚本 |
| `--channels` | 允许 channel server 把权限提示推到手机 |
| `--output-format json\|stream-json` | 配合 `-p` 输出 JSON 供脚本消费 |
| `--max-budget-usd <n>` | 设定会话花费上限（美元） |
| `--max-turns <n>` | 限制最大回合数 |

## 三、权限模式

按 `Shift+Tab` 在 CLI 中循环切换；VS Code / Desktop / Web 在 UI 中点状态栏切换。

| 模式（CLI key） | UI 标签 | 做什么 |
|----------------|--------|--------|
| `default` | Manual | 每步手动确认（新手推荐） |
| `acceptEdits` | Edit automatically | 自动批准文件编辑与 `mkdir/touch/rm/mv/cp/sed` 等命令（仅工作目录内） |
| `plan` | Plan | 只读分析、给计划；不改代码；满意后切回其他模式执行 |
| `auto` | Auto | 后台分类器审查每个动作，符合请求的自动放行（需 Opus 4.6+ / Sonnet 4.6+ / Fable 模型） |
| `bypassPermissions` | Bypass permissions | 完全跳过权限提示；仅在沙箱容器/VM 中使用 |
| `dontAsk` | — | 未预批的工具一律拒绝，仅 CLI 可用 |

> 在 Pro/Max/Team 计划上，`auto` 是新会话的默认模式。  
> `acceptEdits` 模式下 PowerShell 工具还会自动批 `Set-Content` / `Add-Content` / `Remove-Item` 等作用域内命令。

可在 `~/.claude/settings.json` 设置项目级默认值：

```json
{
  "permissions": { "defaultMode": "plan" }
}
```

## 四、会话命令

输入 `/` 触发。下面是常用命令的子集——完整列表（约 100 条内置命令）见 [官方命令参考](https://code.claude.com/docs/en/commands)。

### 会话管理

| 命令 | 描述 |
|------|------|
| `/clear` | 清空对话历史（别名 `/reset`、`/new`） |
| `/compact [聚焦指令]` | 压缩上下文，可附带「保留某部分」指令 |
| `/context [all]` | 可视化当前上下文占用（彩色网格 + 优化建议） |
| `/resume [id\|name]` | 恢复之前会话（别名 `/continue`） |
| `/fork [name]` | 从当前点分叉出新会话（保留原分支） |
| `/rewind` | 回滚对话与代码到某个检查点（别名 `/checkpoint`） |
| `/rename [name]` | 重命名当前会话 |
| `/export [file]` | 导出会话为文本 |
| `/exit` | 退出（别名 `/quit`） |

### 配置与状态

| 命令 | 描述 |
|------|------|
| `/help` | 显示所有可用命令 |
| `/status` | 版本、模型、账户、连接状态 |
| `/cost` | 当前会话 token 用量与花费 |
| `/usage` | 套餐配额与速率限制 |
| `/doctor` | 诊断安装与配置（别名 `/checkup`） |
| `/model [name]` | 切换模型（←→ 调 effort） |
| `/effort [level]` | 设置思考深度：low/medium/high/xhigh/max/ultracode |
| `/theme` | 切换主题 |
| `/config` | 打开配置面板（别名 `/settings`） |
| `/memory` | 编辑 `CLAUDE.md` 与自动记忆 |

### 代码评审与质量

| 命令 | 描述 |
|------|------|
| `/code-review [level]` | 评审当前 diff/PR（`--fix` 自动修，`ultra` 走云端深度评审） |
| `/simplify` | 清理过度设计（v2.1.154 起不再找 bug） |
| `/security-review` | 只读安全审计 |
| `/diff` | 交互式 diff 查看器 |

### 工作流

| 命令 | 描述 |
|------|------|
| `/init` | 分析项目生成 `CLAUDE.md` |
| `/plan [描述]` | 进入 plan 模式（也可 `Shift+Tab`） |
| `/btw <问题>` | 一次性侧问，不进主会话历史 |
| `/loop [interval] <提示>` | 周期执行提示（无 interval 则 Claude 节奏化） |
| `/goal [条件]` | 跨回合自动工作直到条件达成（如 `all tests pass and build is green`） |
| `/batch <指令>` | 大批量改动：用 5–30 个并行 worktree agent 拆任务，每个开 PR |
| `/subtask <指令>` | 把侧任务派给子 agent，结果回主会话 |
| `/background` / `/fork <指令>` | 把会话分离成后台 agent 自己跑 |
| `/add-dir <路径>` | 临时增加可访问目录 |
| `/mcp` | 管理 MCP 服务器连接 |
| `/chrome` | 配置 Chrome 浏览器集成 |
| `/remote-control` | 把本机会话暴露到 claude.ai（别名 `/rc`） |

### 实战常用

| 命令 | 描述 |
|------|------|
| `/voice` | 按住空格说话输入（v2.1.66 起，20 种语言） |
| `/copy [N]` | 复制最后一次回复（`/copy 3` 复制倒数第三次） |
| `/install-github-app` | 给仓库装 Claude GitHub Actions |
| `/release-notes` | 浏览 changelog |
| `/fewer-permission-prompts` | 扫描已批准历史，提议出最小允许列表（别名 `/less-permission-prompts`） |

## 五、快捷键

| 快捷键 | 说明 |
|--------|------|
| `Tab` | 补全命令/文件路径 |
| `Shift+Tab` | 循环切换权限模式（default → acceptEdits → plan → 可选 auto/bypassPermissions） |
| `Esc` | 打断当前生成 |
| `Esc Esc` | 回滚（rewind） |
| `Ctrl+C` | 取消输入/中断生成 |
| `Ctrl+D` | 退出 |
| `Ctrl+L` | 清屏 |
| `Ctrl+R` | 反向搜索历史（v2.1.129 起跨项目） |
| `Ctrl+O` | 切换详细输出（看思考过程） |
| `Ctrl+T` | 切换任务列表 |
| `Ctrl+B` | 把当前任务扔到后台跑 |
| `Ctrl+G` | 在外部编辑器里打开 prompt |
| `Ctrl+V` | 粘贴图片（拖拽图片到终端也可） |
| `Shift+Enter` | 多行换行 |
| `Alt+P` / `Option+P`（macOS） | 切换模型 |
| `↑` / `↓` | 浏览历史输入 |
| `@文件名` | 引用文件/目录为上下文 |

## 六、CLAUDE.md 与 Auto Memory

`CLAUDE.md` 是项目级记忆。每次会话自动加载。多级叠加：`./CLAUDE.md`（子目录）→ 项目根 → `~/.claude/CLAUDE.md`（用户级）。

```markdown
# CLAUDE.md
## 编码规范
- 遵循阿里巴巴 Java 开发手册
- 命名驼峰，Controller 以 Controller 结尾
## 常用命令
- 构建：mvn clean package
- 启动：mvn spring-boot:run
## 注意
- 不要修改 common 模块
- 构建命令见 CONTEXT.md，架构决策见 docs/adr/
```

**Auto Memory** 会自动在工作过程中沉淀构建命令、调试经验等，跨会话保留。需要查看或编辑时用 `/memory`。

## 七、进阶：Skills / MCP / 子代理

| 层 | 作用 |
|----|------|
| **Skills** | `.claude/skills/<name>/SKILL.md` —— 打包一套固定流程，`/name` 触发或自动加载 |
| **MCP** | 接外部工具（数据库 / Jira / Slack / 内部 API），`/mcp` 管理 |
| **Hooks** | 在 PreToolUse / PostToolUse / Stop 等生命周期事件跑 shell 命令（如跑测试、lint、CI 钩子） |
| **Subagents** | `/agents` 管理独立子 agent 配置，含 `disallowedTools`、`maxTurns`、`effort` 等 frontmatter |
| **Plugins** | `/plugin` 装载插件扩展（v2.1.83 起支持从 `.zip` 或 URL 加载） |
| **Worktree 隔离** | `--worktree` 把修改隔离到独立 git worktree，适合并行安全地做改动 |

## 八、实战：用 grill-with-docs 开发公告系统

下面演示如何用 **grill-with-docs** 这个 skill，从「模糊想法」到「清晰可落地的设计」再到「代码」。grill-with-docs 会在写代码前对你「拷问式访谈」，把术语写进 `CONTEXT.md`、把难逆转的决策写成 `docs/adr/`（架构决策记录）。

### 第 1 步：安装 skill

```bash
npx skills add mattpocock/skills --skill grill-with-docs
```

装好后重启 Claude Code，它会在任务匹配时自动加载，也可以主动调用：

> [!EXAMPLE] 输入 AI
> /grill-with-docs 我要做一个公告系统
{icon="fa-solid fa-keyboard"}

### 第 2 步：启动 grilling 会话，拷问设计

你只需给出一个粗糙的想法，skill 会**一次只问一个问题**，逐步收敛术语、边界、依赖。例如：

> 你：我要做一个公告系统，管理员能发公告，用户能看。
>
> **grill**：你说的「公告」和「通知」是一个东西吗？系统里是否还存在推送、站内信这类概念？
> 你：只有公告，不涉及通知和推送。
>
> **grill**：公告有哪些状态？草稿、已发布、已撤回之间怎么流转？已撤回的公告能被再次发布吗？
> 你：状态就三个：草稿 → 已发布 → 已撤回；撤回后允许再次发布。
>
> **grill**：「发布人」是什么身份？是登录用户，还是系统管理员角色？和「创建人」会不是同一个人吗？
> 你：发布人就是当前登录的管理员，创建人也是他，是同一个人，字段可合并。
>
> **grill**：置顶公告最多几条？多条置顶时排序规则是什么？
> 你：置顶不限数量，按更新时间倒序。

每答完一轮，skill 会把确认的术语写入 `CONTEXT.md`，把难逆转的决策写成 ADR。

### 第 3 步：产出文档

会话结束后，项目根目录会多出：

```
├── CONTEXT.md              # 术语表：公告/状态/发布人 等统一定义
└── docs/
    └── adr/
        └── 0001-公告状态机采用草稿已发布已撤回三态.md
```

> 这一步的价值：**先对齐语言和决策，再写代码**，避免后期返工。

### 第 4 步：进入开发

设计清晰后回到常规开发流程。建议**先在 plan 模式**让 Claude 出实现方案：

> [!EXAMPLE] 输入 AI
> /plan 帮我创建 Spring Boot 3 公告系统，Maven 构建，包名 com.example.announcement，JDK 17
{icon="fa-solid fa-keyboard"}

满意后切到 **acceptEdits** 模式（`Shift+Tab`），让 Claude 边改边自动批：

> [!EXAMPLE] 输入 AI
> 按公告状态机实现接口：
> 1. 新增、修改、删除
> 2. 分页查询（按分类、状态、关键字过滤）
> 3. 发布（草稿→已发布，记录发布时间）
> 4.撤回（已发布→已撤回，允许再次发布）
{icon="fa-solid fa-keyboard"}

中途可用 `/diff` 看修改，用 `/compact 保留 API 设计决策` 压缩长上下文。

### 第 5 步：验证与测试

> [!EXAMPLE] 输入 AI
> 启动项目，用 curl 测新增、分页、发布、撤回接口
{icon="fa-solid fa-keyboard"}

> [!EXAMPLE] 输入 AI
> 给发布和撤回逻辑写单元测试，覆盖：草稿→发布、撤回→再次发布、重复发布
{icon="fa-solid fa-keyboard"}

跑 `/code-review` 做一遍自审：

> [!EXAMPLE] 输入 AI
> /code-review
{icon="fa-solid fa-keyboard"}

### 第 6 步：提交 Git

> [!EXAMPLE] 输入 AI
> 查看改了哪些文件，用一个描述性的信息提交
{icon="fa-solid fa-keyboard"}

或在 Claude 之外直接 `git diff` 后自己提交——`/diff` 同样能看交互式 diff。

### 为什么用 grill-with-docs

- **在写代码前把方案拷问清楚**，术语冲突、边界模糊、决策盲区都在设计阶段暴露
- **边聊边沉淀文档**，`CONTEXT.md` 和 ADR 让团队/未来的 AI 共用一套语言
- 尤其适合**已有代码库**、术语容易漂移的复杂项目；全新无文档的项目用它的兄弟 skill `grill-me` 更合适

> 官方文档：https://code.claude.com/docs