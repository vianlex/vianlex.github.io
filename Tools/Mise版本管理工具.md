# mise 完整详细教程

> **mise**（mise-en-place）是一个现代化的开发环境管理工具，集版本管理（类 nvm/pyenv/asdf）、环境变量管理、任务运行器于一体，支持 Windows、macOS、Linux。

---

## 一、安装

### Linux
```bash
# 默认安装在 ~/.local/bin 目录，
# 注意：如果 ~/.local/bin 未在 Path 环境变量中，也我们不需要手动添加环境变量，因为 mise 激活时，会自动添加环境变量
curl https://mise.run | sh
```

### Windows（PowerShell 管理员）
```powershell
# 方式1
winget install jdx.mise
# 方式2
scoop install mise
```

### 手动安装
```bash
# 1. 通过地址压缩包
https://github.com/jdx/mise/releases
# 2. 解压文档到目录，配置好，环境变量即可。
```

### 默认目录说明：

- 默认数据目录： ~\AppData\Local\mise 安装版本工具的目录，可通过 MISE_DATA_DIR 环境变量修改默认值  
- 默认配置目录：~\.config\mise 存放 mise 配置信息的的目录，可通过 MISE_CONFIG_DIR 环境变量修改默认值
- 默认缓存目录：~\AppData\Local\Temp\mise 版本工具下载缓存目录，可通过 MISE_CACHE_DIR 环境变量修改默认值
- 默认 shims 可执行垫片目录： ~\AppData\Local\mise\shims 可通过 MISE_SHIMS_DIR 环境变量修改默认值


---

## 二、激活 mise（推荐）

激活后，切换目录时 mise 会自动加载对应工具版本和环境变量。

**bash：**
```bash
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
source ~/.bashrc
```

**PowerShell：**
```bash
if (!(Test-Path $PROFILE)) { New-Item -Path $PROFILE -ItemType File -Force }
Add-Content $PROFILE 'eval "$(mise activate powershell) | Out-String | Invoke-Expression "'
```

**Cmd：**
```bash
# 将 mise 的 shims 的默认路径是：%LOCALAPPDATA%\mise\shims，我们只需要将该路径添加到系统环境变量即可。
```

激活后运行诊断确认配置正确：
```bash
mise dr
```

> **不想激活？** 也可以每次用 `mise exec` 或 `mise run` 直接运行，不影响全局 shell 环境。

---

## 三、管理开发工具

### 3.1 安装并激活工具（推荐用 `mise use`）

```bash
# 在当前项目目录安装 Node 24，临时配置当前终端生效
cd my-project
# 会安装 24 的所有小版本，最好指定完整的版本号
mise use node@24

# 全局安装
mise use --global node@24
mise use --global python@3.12
mise use --global go@latest
```

> ⚠️ **重要**：`mise install` 只安装不激活，必须用 `mise use` 才能让工具在 shell 中可用。

### 3.2 临时运行（不修改配置）

```bash
mise exec python@3 -- python          # 临时启动 Python 3 REPL
mise exec node@24 -- node -v          # 临时运行 Node 24
mise exec -- node my-script.js        # 用当前项目配置运行
```

> 💡 可以设置别名：`alias x="mise x --"` 简化输入

### 3.3 多生态系统后端

mise 支持从多个包生态系统安装工具：

```bash
# npm 生态
mise use --global npm:@anthropic-ai/claude-code
mise use npm:@antfu/ni

# PyPI (pipx)
mise use --global pipx:black
mise use --global pipx:ruff

# GitHub Releases（直接从 Release 下载）
mise use --global github:BurntSushi/ripgrep
mise use --global github:cli/cli

# asdf 插件生态（兼容 asdf 所有插件）
mise use terraform@latest
```

### 3.4 查看与升级工具

```bash
mise ls                    # 列出已安装/激活的工具
mise ls-remote node        # 查看 node 所有可用版本
mise outdated              # 检查哪些工具有新版本

mise upgrade node          # 升级 node（保持主版本号，如 24.x → 24.latest）
mise upgrade --bump node   # 升级到最新主版本（如 24 → 26）

mise uninstall node@22     # 卸载指定版本
```

---

## 四、配置文件 `mise.toml`

### 4.1 文件结构

```toml
# mise.toml
[tools]
node = "24"
python = "3.12"
ruby = "latest"

[env]
NODE_ENV = "production"
DATABASE_URL = "postgres://localhost/mydb"

[tasks]
build = "npm run build"
test = "npm test"
```

### 4.2 配置层级（从宽到窄覆盖）

```
~/.config/mise/config.toml          # 全局默认
~/work/mise.toml                    # 工作目录级别
~/work/project/mise.toml            # 项目级别（提交到 git）
~/work/project/mise.local.toml      # 个人私有配置（加入 .gitignore）
```

下层配置会覆盖上层，实现精细化版本控制。

### 4.3 兼容 asdf `.tool-versions`

```
# .tool-versions（asdf 格式，mise 完全兼容）
nodejs 24.0.0
python 3.12.0
```

### 4.4 版本锁定

```bash
mise use --pin node@24     # 锁定精确版本，写入 mise.toml
```

或在 `mise.toml` 中开启 lockfile：
```toml
[settings]
lockfile = true
```

### 4.5 工具安装后钩子

```toml
[tools]
node = { version = "22", postinstall = "corepack enable" }
```

### 4.6 平台限定工具

```toml
[tools]
ripgrep = { version = "latest", os = ["linux", "macos"] }
"npm:windows-terminal" = { version = "latest", os = ["windows"] }
```

---

## 五、环境变量管理

### 5.1 设置变量

```bash
mise set MY_VAR=123
echo $MY_VAR   # 123
```

或直接写 `mise.toml`：
```toml
[env]
NODE_ENV = "production"
AWS_REGION = "us-east-1"
RUST_TEST_THREADS = "1"
```

### 5.2 修改 PATH

```toml
[env]
# 将 node_modules/.bin 加入 PATH（相对于 mise.toml 所在目录）
_.path = "./node_modules/.bin"
```

### 5.3 加载 .env 文件

```toml
[env]
_.file = ".env"
```

---

## 六、任务运行器

### 6.1 在 `mise.toml` 中定义任务

```toml
[tasks]
build = "npm run build"
test = "npm test"
lint = "eslint src/"
dev = "npm run dev"
```

运行：
```bash
mise run build
mise run test
mise r dev      # 简写
```

### 6.2 独立任务文件（`mise-tasks/` 目录）

```bash
# mise-tasks/deploy
#!/bin/bash
set -e
npm run build
rsync -avz dist/ user@server:/var/www/
```

```bash
chmod +x mise-tasks/deploy
mise run deploy
```

### 6.3 带参数的高级任务（usage spec）

```bash
# mise-tasks/greet
#!/usr/bin/env bash
#MISE description="Greet a user"
#USAGE flag "-u --user <user>" help="The user to greet"
#USAGE arg "<message>" help="Greeting message"

echo "${usage_greeting:-Hello}, ${usage_user}! ${usage_message}"
```

```bash
mise run greet --user Alice "Good morning!"
mise run greet --help    # 自动生成帮助文档
```

### 6.4 文件监听自动运行

```bash
mise watch build    # 文件变更时自动重新运行 build 任务
```

---

## 七、自动补全

```bash
# bash
mise completion bash >> ~/.bashrc
```

---

## 八、常用命令速查表

| 命令 | 说明 |
|------|------|
| `mise use node@24` | 安装并激活 Node 24（当前项目） |
| `mise use --global node@24` | 全局安装并激活 |
| `mise exec node@24 -- node -v` | 临时运行，不修改配置 |
| `mise ls` | 列出已安装工具 |
| `mise ls-remote node` | 查看可用版本列表 |
| `mise outdated` | 检查可升级工具 |
| `mise upgrade node` | 升级工具 |
| `mise uninstall node@22` | 卸载指定版本 |
| `mise run <task>` | 运行任务 |
| `mise set KEY=VALUE` | 设置环境变量 |
| `mise dr` | 诊断配置 |
| `mise self-update` | 更新 mise 自身 |
| `mise config ls` | 查看当前生效的配置文件 |
| `mise edit` | 交互式编辑配置（TUI） |

---

## 九、注意事项

### GitHub API 限流
mise 很多工具依赖 GitHub API，建议配置 Token：
```bash
export MISE_GITHUB_TOKEN=your_personal_access_token
# 或
export GITHUB_TOKEN=your_personal_access_token
```

### mise vs asdf
- mise 完全兼容 asdf 插件和 `.tool-versions` 文件
- mise 比 asdf **快得多**（Rust 实现，零 shim 开销）
- mise 额外支持环境变量管理和任务运行器

---

## 十、典型工作流示例

```bash
# 1. 克隆项目
git clone https://github.com/example/project && cd project

# 2. 安装项目所需所有工具（读取 mise.toml）
mise install

# 3. 运行开发任务
mise run dev

# 4. 运行测试
mise run test

# 5. 查看当前环境
mise ls
```

---

> 官方文档：https://mise.jdx.dev
> 导出时间：2026-03-20
