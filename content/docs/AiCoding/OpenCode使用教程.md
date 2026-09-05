---
title: "OpenCode 使用教程"
linkTitle: "OpenCode 使用教程"
weight: 30
description: "OpenCode 安装、使用与公告系统实战"
---

# OpenCode 使用教程

**OpenCode** 是 100% 开源、提供商中立（provider-agnostic）的终端 AI 编码 agent。原仓库 [anomalyco/opencode](https://github.com/anomalyco/opencode)（sst 团队）。形态与 Claude Code 高度相似但更开放：可接 Claude / OpenAI / Google / 本地模型；内置 LSP；TUI 由 neovim 用户打造。**官方文档**：https://opencode.ai/docs/

本教程讲「怎么用」，并以开发一个**公告系统**为例。

## 一、安装与认证

### 安装

```bash
# 推荐：官方安装脚本（自动选二进制，自动加 PATH）
curl -fsSL https://opencode.ai/install | bash

# 包管理器
npm install -g opencode-ai            # Node.js（npm/bun/pnpm/yarn 通用）
brew install sst/tap/opencode         # macOS/Linux（推荐，更新及时）
brew install anomalyco/tap/opencode   # 同上，官方 tap
brew install opencode                 # Homebrew 官方（更新慢）
scoop install opencode                # Windows
choco install opencode                # Windows
paru -S opencode-bin                  # Arch Linux
mise use -g opencode                  # 跨平台
docker run -it --rm ghcr.io/anomalyco/opencode  # Docker
```

> Windows 上官方脚本自动安装目前不太稳，建议用 `scoop` 或 `choco`，或直接从 Releases 拉二进制。

安装脚本支持的安装路径（按优先级）：`$OPENCODE_INSTALL_DIR` > `$XDG_BIN_DIR` > `$HOME/bin` > `$HOME/.opencode/bin`。

### 认证

```bash
opencode auth login    # 交互式选 provider（Anthropic/OpenAI/Google/Bedrock/Azure/Groq/DeepSeek 等）
opencode auth list     # 查看已登录的 provider
opencode auth logout   # 退出登录
```

也支持环境变量直接注入：

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GEMINI_API_KEY=...
export GITHUB_TOKEN=ghp_...        # GitHub Copilot 模型
```

如果嫌选 provider 麻烦，官方推荐 **OpenCode Zen**：内置经官方测试过的精选模型列表，TUI 里 `/connect` 选 opencode，去 `opencode.ai/auth` 注册拿 key。

## 二、基本使用

### 启动交互模式

> 以下都是**启动 OpenCode 进程时的命令行参数**（在终端里执行），不是会话命令——进程启动后使命就结束了。

```bash
opencode                    # 在当前目录打开交互模式
opencode /path/to/project   # 在指定目录打开
```

首次进入某个项目，建议先初始化：

```bash
opencode
/init   # 在 TUI 里输入：让 OpenCode 分析项目并生成 AGENTS.md（项目根）
```

`AGENTS.md` 是 OpenCode 给当前项目写的「说明书」：项目结构、构建命令、代码风格。后续每次启动都会读这份文件。

### 非交互模式命令

> 非交互模式一次性输出，跑完退出，不进入 TUI。

```bash
opencode run "总结这个代码库"                    # 非交互模式，类似 Pi 的 -p
cat README.md | opencode run "总结"             # 支持 stdin 管道
opencode run --model anthropic/claude-sonnet-4 "解释这段代码"  # 指定模型
opencode run --share "重构 utils.ts"            # 输出分享链接
```

`opencode run` 适合脚本化、CI、批处理。

### 模式对比

| 命令 | 模式 | 输出给谁 |
|------|------|---------|
| `opencode` | 交互 TUI | 人（终端里看着） |
| `opencode run "..."` | 非交互 | 人（一次性纯文本输出） |
| `opencode auth login` | 子命令 | 走配置流程 |

简单记：`run` = 一次性脚本化；默认交互 = 边聊边看。

### 内置双 Agent（Tab 切换）

OpenCode 内置两个 agent，`Tab` 键切换：

| Agent | 用途 | 权限 |
|-------|------|------|
| `build` | 默认，全权限开发 agent | 完整 |
| `plan` | 只读分析和规划 | 默认拒绝编辑，bash 会先问权限 |

底部状态栏右下角有当前模式指示器。

还有一个内置 subagent `@general` 用于复杂搜索和多步任务，消息里 `@general` 即可调用。

### 默认能力

- **LSP**：自动加载项目语言服务器，AI 看代码更准
- **多 provider**：75+ LLM 提供商（含本地模型），通过 Models.dev 维护
- **多 agent 并行**：可在同一项目上让多个 agent 并行干活
- **分享会话**：任何会话都能生成可分享链接

## 三、输入前缀

TUI 输入框里输入的第一个字符，决定走哪条通道。OpenCode 只有 **3 个真前缀**——比 Claude Code/Pi 都克制。

| 前缀 | 触发什么 | 一句话说明 |
|------|---------|-----------|
| `/` | **斜杠命令** | 调用内置命令或自定义命令（输入 `/` 弹菜单） |
| `!` | **Shell 命令** | 直接跑 shell；输出加入对话上下文 |
| `@` | **文件/目录引用** | `@src/auth.ts` 模糊搜索引用文件；也可 `@general` 调子 agent |
| 无前缀 | **自然语言** | 普通任务指令 |

> 和 Claude Code 的 `! !! @ # &` 6 个前缀、Pi 的 `! !! @ /` 4 个前缀相比，OpenCode 故意只留 3 个：**够用就行**。Shell 输出没有「仅本人看」的 `!!` 变体，也没有 `#` 写记忆、`&` 后台这种扩展机制——这些都需要到 `/` 命令里找（如 `/init` 写 `AGENTS.md`、`/share` 外部分享）。

### Shell 模式（`!`）

`!` 开头的行直接当 shell 跑，输出自动进对话：

```text
! ls -la
! npm test
! git status
```

退出方式：按 `Esc` 或清空输入。OpenCode 的 shell 模式没有「只看自己」的子变体——任何 shell 输出 AI 都会看到。

### 文件引用（`@`）

```text
@src/auth.ts
@packages/api/
@general 找出所有 REST 端点       # 子 agent
```

`@` 后接文件名触发模糊搜索补全，匹配的文件内容自动注入上下文。`@` 后接 agent 名（如内置的 `@general`）可调用子 agent。

## 四、会话命令

输入 `/` 触发。下面是常用命令的子集——完整列表见 [官方 TUI 文档](https://opencode.ai/docs/tui/)。

| 命令 | 作用 | 快捷键 |
|------|------|--------|
| `/init` | 分析当前项目并生成 `AGENTS.md` | `ctrl+x i` |
| `/connect` | 配置 provider（首次或切换） | — |
| `/models` | 列出可用模型并切换 | `ctrl+x m` |
| `/themes` | 列出主题 | `ctrl+x t` |
| `/new` | 开始新会话（别名 `/clear`） | `ctrl+x n` |
| `/sessions` | 列出并切换会话（`/resume`、`/continue`） | `ctrl+x l` |
| `/share` | 生成分享链接 | `ctrl+x s` |
| `/unshare` | 取消分享 | — |
| `/compact` | 压缩上下文（别名 `/summarize`） | `ctrl+x c` |
| `/undo` | 撤销上条消息及文件变更（需 Git） | `ctrl+x u` |
| `/redo` | 重做已撤销的（需 Git） | `ctrl+x r` |
| `/editor` | 打开外部编辑器写消息（依赖 `$EDITOR`） | `ctrl+x e` |
| `/export` | 导出对话为 Markdown 并打开 | `ctrl+x x` |
| `/details` | 切换工具执行详情显示 | `ctrl+x d` |
| `/thinking` | 切换模型思考过程显示 | — |
| `/help` | 显示帮助对话框 | `ctrl+x h` |
| `/exit` | 退出 OpenCode（`/quit`、`/q`） | `ctrl+x q` |

> 很多命令都有 `ctrl+x <字母>` 形式的快捷键（leader 键机制）。`ctrl+x` 后 2 秒内必须按下第二键（`leader_timeout: 2000`，可在 `tui.json` 调）。整套快捷键都能在 `tui.json` 里改。

## 五、TUI 与快捷键

**编辑与导航：**

| 快捷键 | 作用 |
|------|------|
| `Enter` | 发送消息（部分情况需按两次） |
| `Shift+Enter` / `Ctrl+Enter` / `Ctrl+J` | 换行不发送 |
| `Ctrl+C` | 清空编辑器（再按退出） |
| `Esc` | 中止输出流 |
| `Tab` | 切换 agent（build ↔ plan ↔ ...） |
| `Shift+Tab` | 反向切换 agent |
| `Ctrl+P` | 打开命令面板（按名字过滤） |

**控制与显示：**

| 快捷键 | 作用 |
|------|------|
| `ctrl+x` | leader 键，按住 2 秒内跟一字母触发命令快捷键 |
| 拖拽图片到终端 | 图片作为输入发给 AI |

> OpenCode 支持直接在 TUI 里拖图片——拖一张截图进终端窗口，OpenCode 自动识别并把它加进当前 prompt，不需要任何前缀。

### 提问、加功能、改 bug

OpenCode 官方建议三种典型用法：

**1. 提问了解代码库**

> [!EXAMPLE] 输入 AI
> How is authentication handled in @packages/functions/src/api/index.ts
{icon="fa-solid fa-keyboard"}

`@文件名` 把文件内容带入上下文，比描述半天更准。

**2. 加新功能（先用 plan 模式）**

> [!EXAMPLE] 输入 AI
> Switch to plan mode, then design: when a user deletes a note, flag it as deleted in the database; add a screen showing recently deleted notes with undelete/permanent-delete options.
{icon="fa-solid fa-keyboard"}

plan 模式下 OpenCode 不改代码，只给方案。满意后 `Tab` 切回 build 模式：

> [!EXAMPLE] 输入 AI
> Sounds good, go ahead and implement.
{icon="fa-solid fa-keyboard"}

**3. 基于截图改设计**

OpenCode 能扫任何图片：拖一张设计稿进终端，配一句说明：

> [!EXAMPLE] 输入 AI
> 用这张图作参考，重做这个屏的样式
{icon="fa-solid fa-keyboard"}

## 六、会话管理

会话默认存在 `~/.local/share/opencode/`（SQLite 持久化），支持分享链接与继续上次会话。

**启动时的会话选项：**

```bash
opencode -c                # --continue：继续最近会话
opencode -s <session-id>   # --session：用指定会话 ID 继续
opencode --share           # 跑完自动生成分享链接
```

**交互模式中的会话管理：**

| 命令 | 作用 |
|------|------|
| `/share` | 生成当前会话的可分享链接（任何会话都能分享给他人查阅/调试） |
| `/clear` | 清空当前会话上下文 |
| `/compact` | 手动压缩上下文（接近上限时） |
| `opencode auth list` | 查看已配置的 provider |

OpenCode 支持**多 agent 并行**：在同一项目里可以让多个 OpenCode 实例同时跑不同任务（例如一边重构、一边补测试、一边修 lint），互不干扰。

底部状态栏实时显示 token/缓存命中率/成本/当前 agent。

## 七、实战：开发一个公告系统

用 OpenCode 从零开发一个 Node.js（Express）公告系统，每步给出你该说的话。先选 plan 模式做整体规划。

### 第 1 步：初始化项目

> [!EXAMPLE] 输入 AI
> 帮我初始化一个 Express 公告系统项目，使用 npm，入口 app.js
{icon="fa-solid fa-keyboard"}

OpenCode 执行 `npm init`、安装 `express`、生成 `app.js` 和 `package.json`。然后建议立刻跑一次 `/init`：

> [!EXAMPLE] 输入 AI
> /init 生成项目级 AGENTS.md
{icon="fa-solid fa-keyboard"}

让 OpenCode 写一份项目说明书，后续每次启动都自动加载。

### 第 2 步：设计数据模型（先用 plan 模式）

按 `Tab` 切到 plan agent。

> [!EXAMPLE] 输入 AI
> 设计公告数据模型：id、标题、内容、分类、置顶标志、
> 状态(草稿/已发布/已撤回)、发布人、发布时间、创建时间、更新时间。
> 用 JSON 文件做存储，放到 data/announcements.json
{icon="fa-solid fa-keyboard"}

plan agent 给方案，满意后 `Tab` 切回 build：

> [!EXAMPLE] 输入 AI
> 方案 OK，按这个实现
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

OpenCode 执行 `node app.js`，写 curl 命令验证并报告结果。发现问题直接说：

> [!EXAMPLE] 输入 AI
> 分页没有按状态过滤，帮我修一下
{icon="fa-solid fa-keyboard"}

### 第 5 步：分屏并行做方案对比

OpenCode 的优势之一是**多 agent 并行**。在同一个项目里开两个终端：

```bash
# 终端 A：方案一（状态字段流转）
opencode

# 终端 B：方案二（软删除）
opencode
```

两个 OpenCode 实例独立工作，对比哪个实现更合适。

### 第 6 步：脚本化批量操作

```bash
# 用非交互模式做一次性操作
cat data/announcements.json | opencode run "找出所有草稿状态的公告 id"

# 输出分享链接给同事 review
opencode run --share "解释这个项目的启动流程"

# 升级 OpenCode
opencode upgrade
opencode upgrade v0.1.48   # 升级到指定版本
```

## 八、扩展与生态

| 层 | 作用 |
|---|---|
| 配置文件 | `~/.config/opencode/opencode.json`（全局）+ `./opencode.json`（项目） |
| 主题 | TUI 主题与键位可在配置里改 |
| MCP | 接外部工具（数据库/Jira/Slack 等） |
| Plugins | 加载插件扩展能力 |
| Custom Commands | 自定义命令与 named placeholders |
| LSP | 项目语言服务器自动加载 |
| Desktop App | OpenCode 还提供桌面应用（Beta）：`brew install --cask opencode-desktop` 或 `scoop install extras/opencode-desktop` |
| Client/Server | 客户端/服务端架构，TUI 只是客户端之一，可远程驱动 |

### 配置示例

```json
{
  "$schema": "https://opencode.ai/config.json",
  "theme": "opencode",
  "model": "anthropic/claude-sonnet-4-20250514",
  "autoshare": false,
  "autoupdate": true
}
```

可在 `~/.config/opencode/opencode.json`（全局，影响主题/provider/键位）或项目根 `./opencode.json`（项目级，影响 provider/modes，可提交 git）放。

### 与 Claude Code 的取舍

| 维度 | Claude Code | OpenCode |
|------|-------------|----------|
| 开源 | 否 | 是 |
| Provider 锁定 | Anthropic | 任意（75+） |
| LSP | 内置 | 内置 |
| 多 agent 并行 | 通过 subagents | 原生支持 |
| 桌面/IDE 扩展 | 桌面/VS Code/JetBrains/Slack | TUI/桌面（Beta） |
| Skills | 完整支持 | 通过 Plugins/MCP |

> 官方：https://opencode.ai/docs/
> GitHub：https://github.com/anomalyco/opencode