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
npm install -g @mariozechner/pi-coding-agent
```

> 包名 scope 曾多次变更，以 https://pi.dev 官网当前为准。

认证两种方式：

```bash
export ANTHROPIC_API_KEY=sk-ant-...   # 方式一：API Key
pi /login                             # 方式二：订阅登录（浏览器授权）
```

## 二、基本使用

### 启动模式

> 以下都是**启动 Pi 进程时的命令行参数**（在终端里执行），不是会话命令——进程启动后使命就结束了。

```bash
pi                          # 交互模式（默认）
pi -p "总结这个代码库"          # print 模式，一次性输出
cat README.md | pi -p "总结"   # 支持 stdin 管道
pi --mode json "..."          # JSONL 事件流，脚本处理
pi --mode rpc                 # RPC 协议，进程集成
```

### 切换模型

> 这里 `--model` 也是**命令行参数**，随启动指定；与交互界面里的会话命令 `/model`、快捷键 `Ctrl+L` 不同。

```bash
pi --model openai/gpt-4o "帮我重构这段代码"
pi --model sonnet:high "解决这个复杂问题"   # 带 thinking level
```

交互模式内按 `Ctrl+L` 打开模型选择器。

### 默认工具

`read` / `write` / `edit` / `bash` 四件套，需要更多能力靠扩展。

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

此外还支持**截图反馈**：Windows 按 `Alt+V`、macOS 按 `Ctrl+V` 粘贴图片，配合文字描述需求。

### 会话命令（输入 `/` 触发）

| 命令 | 作用 |
|------|------|
| `/model` | 切换模型 |
| `/new` `/resume` | 新建/恢复会话 |
| `/tree` | 会话树（按两次 `Esc` 打开） |
| `/fork` | 从当前节点开新会话 |
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

**一句话区分**：

| 场景 | 用哪种 |
|------|--------|
| 现在方向就错了 | Steering（`Enter`） |
| 方向没错，做完还有下一件事 | Follow-up（`Alt+Enter`） |
| 整个任务都不该继续 | `Esc` 中止 |

辅助按键：`Esc` 中止并把排队消息退回编辑器；`Alt+↑` 取回排队消息编辑而不中止。

> Windows Terminal 默认把 `Alt+Enter` 绑成全屏快捷键，需先在终端设置里解除/重映射，或用 `/hotkeys` 核对当前绑定。

## 四、会话树：Pi 的独门设计

会话以 JSONL 存于 `~/.pi/agent/sessions/`，天然分支树。

- `/tree` 浏览全部历史节点，跳转到任意节点继续
- `/fork` 从历史节点开新分支（比如「方案 A 跑不通，回到上一步试方案 B」）
- `/compact` 长会话逼近上下文上限时压缩

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

跑通后按 `Esc Esc` 打开 `/tree`，跳回实现前的节点 `/fork`，让 Pi 用软删除方案重做，对比两种实现。

### 第 6 步：脚本化批量操作

```bash
# 用 print 模式做一次性操作
cat data/announcements.json | pi -p "找出所有草稿状态的公告 id"

# 用 json 模式接入自己的脚本
pi --mode json "分析这个接口的边界情况" > analysis.jsonl
```

## 六、扩展生态

| 层 | 作用 |
|---|---|
| Skills | Markdown 能力包，`/skill:name` 触发 |
| Prompt Templates | 可复用提示，`/` 展开 |
| Extensions | TypeScript 模块，加工具/命令/UI |
| Packages | 打包分享，`pi install git:...` |

⚠️ 第三方包以完整系统权限运行，安装前先审源码。

> 官网：https://pi.dev
