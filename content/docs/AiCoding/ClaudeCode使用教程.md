---
title: "Claude Code 使用教程"
linkTitle: "Claude Code 使用教程"
weight: 10
description: "Claude Code 安装、使用与公告系统实战"
---

# Claude Code 使用教程

Claude Code 是 Anthropic 官方的命令行 AI 编码工具，直接在终端里读代码、改代码、跑命令、提交 Git。本教程讲「怎么用」，并以开发一个**公告系统**为例走完整流程。

## 一、安装与登录

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# 或包管理器
brew install --cask claude-code          # Homebrew
winget install Anthropic.ClaudeCode      # WinGet
```

首次运行 `claude` 会引导浏览器登录。订阅账户直接授权；用 API 计费则：

```bash
claude auth login --console
```

## 二、基本使用

### 启动

```bash
cd 你的项目目录
claude                      # 交互模式
claude "修一下构建错误"        # 带初始提示启动
claude -p "解释这个函数"       # 一次性查询后退出
claude -c                    # 继续当前目录最近对话
tail -200 app.log | claude -p "有异常就告诉我"   # 管道输入
```

### 权限模式

每次改动前会询问是否批准。按 `Shift+Tab` 切换：

- **auto**：分类器自动审查，大多数编辑/命令自动放行
- **manual**：逐步人工确认

### 会话命令

`/help` `/clear` `/model` `/login` `/exit`（或连按两次 `Ctrl+D`）。

## 三、会话命令与快捷键

进入 Claude Code 后，输入 `/` 触发以下会话命令：

| 命令 | 描述 |
|------|------|
| `/help` | 显示所有可用命令 |
| `/clear` | 清空当前对话历史 |
| `/model` | 切换使用的模型 |
| `/login` | 切换或重新登录账户 |
| `/resume` | 恢复之前的会话 |
| `/exit` | 退出（也可连按两次 `Ctrl+D`） |

### 快捷键

| 快捷键 | 说明 |
|--------|------|
| `Tab` | 补全命令 / 文件路径 |
| `Shift+Tab` | 循环切换权限模式（auto / manual） |
| `Esc` | 打断正在运行的 Claude |
| `Ctrl+C` | 取消当前操作 |
| `Ctrl+D` | 退出（连按两次） |
| `↑` / `↓` | 浏览历史输入 |
| `Ctrl+L` | 清屏 |
| `Ctrl+R` | 反向搜索命令历史 |
| `@` | 引用文件/目录作为上下文 |

> 完整命令清单见官方 CLI 参考：https://code.claude.com/docs/en/cli-reference

## 四、CLAUDE.md：让 AI 懂你的项目

项目根目录放 `CLAUDE.md`，每次会话自动读取，固化规范：

```markdown
# CLAUDE.md
## 编码规范
- 遵循阿里巴巴 Java 开发手册
- 命名用驼峰，Controller 以 Controller 结尾
## 常用命令
- 构建：mvn clean package
- 启动：mvn spring-boot:run
## 注意
- 不要修改 common 模块
```

支持多级：`./CLAUDE.md`（子目录）→ 项目根 → `~/.claude/CLAUDE.md`（用户级）。

## 五、实战：用 grill-with-docs 开发公告系统

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

设计清晰后，回到常规开发流程。**不需要手动指定文档路径**——Claude Code 会自动读取 `CONTEXT.md` 和 ADR。直接说需求即可：

> [!EXAMPLE] 输入 AI
> 帮我创建一个 Spring Boot 3 公告系统项目，用 Maven 构建，
> 包名 com.example.announcement，JDK 17，公告字段按约定建表
{icon="fa-solid fa-keyboard"}

> [!EXAMPLE] 输入 AI
> 按公告状态机实现接口：
> 1. 新增、修改、删除
> 2. 分页查询（按分类、状态、关键字过滤）
> 3. 发布（草稿→已发布，记录发布时间）
> 4. 撤回（已发布→已撤回，允许再次发布）
{icon="fa-solid fa-keyboard"}

想确保每次会话都稳定加载这些文档，可在 `CLAUDE.md` 里加一行引用——**不用手写**，直接在会话里说一句，Claude Code 会自己读取并编辑 `CLAUDE.md`：

> [!EXAMPLE] 输入 AI
> 把「领域术语见 CONTEXT.md，架构决策见 docs/adr/」加进 CLAUDE.md
{icon="fa-solid fa-keyboard"}

此外 Claude Code 的 Auto Memory 也会在工作过程中自动沉淀构建命令、调试经验等，跨会话保留。

### 第 5 步：验证与测试

> [!EXAMPLE] 输入 AI
> 启动项目，用 curl 测新增、分页、发布、撤回接口
{icon="fa-solid fa-keyboard"}

> [!EXAMPLE] 输入 AI
> 给发布和撤回逻辑写单元测试，覆盖：草稿→发布、撤回→再次发布、重复发布
{icon="fa-solid fa-keyboard"}

### 第 6 步：提交 Git

> [!EXAMPLE] 输入 AI
> 查看改了哪些文件，用一个描述性的信息提交
{icon="fa-solid fa-keyboard"}

### 为什么用 grill-with-docs

- **在写代码前把方案拷问清楚**，术语冲突、边界模糊、决策盲区都在设计阶段暴露
- **边聊边沉淀文档**，`CONTEXT.md` 和 ADR 让团队/未来的 AI 共用一套语言
- 尤其适合**已有代码库**、术语容易漂移的复杂项目；全新无文档的项目用它的兄弟 skill `grill-me` 更合适

> 官方文档：https://code.claude.com/docs
