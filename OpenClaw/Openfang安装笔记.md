# Openfang 安装笔记

## 安装介绍

### Linux 安装

`openfang` 是一个二进制文件，通过脚本安装 `openfang` 时会默认将 `openfang` 安装到用户 `~/.openfang/bin` 目录下，同时会默认将 `~/.openfang/bin` 目录添加到用户级别的环境变量。

注意：如果终端执行 `openfang` 提示未找到命令，我们手动将 `~/.openfang/bin` 手动添加到环境变量即可。 

```bash
# 终端执行安装命令
curl -fsSL https://openfang.sh/install | sh

# 安装成功后，执行以下命令查看版本
openfang --version
```

### Window 安装 

```bash
irm https://openfang.sh/install.ps1 | iex
```

### 初始化 

运行 `init` 命令,在 `~/.openfang/`目录下，创建默认配置文件和工作目录以及数据存放目录：

```bash
openfang init 
```

初始化成功后，通过以下命令启动 `openfang`

```bash
# 启动
openfang start 

# 查看状态
openfang status

# 通过 --help 可以查看其他命令
openfang --help 
```

#### 内置 Agent 智能体模板

`init` 初始化后，会在目录 `~/.openfang/agents` 下生成 30 套内置 agent 智能体模板，我们可以通过模板快捷的初始化 agent 智能体，具体操作命令如下：

```bash 
# 通过内置的 hello-world 智能体模板，生成 hello-world 智能体。
openfang agent spawn agents/hello-world/agent.toml
```



## 文档链接

1. https://www.openfang.sh/docs/getting-started