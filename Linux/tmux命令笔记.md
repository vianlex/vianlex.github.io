# tmux 命令笔记

## tmux 介绍

tmux 是一个终端复用器（terminal multiplexer）。简单来说，它让你能在一个终端窗口里管理多个会话、窗口和面板，而且所有这些会话都可以在后台保持运行，即使你关掉终端甚至断开 SSH 连接，它们也不会消失。

### 核心概念

- 会话 (Session)：就像 Chrome 浏览器的一个窗口，里面可以开多个标签页
- 窗口 (Window)：就像浏览器里的一个标签页，每个标签页可以干不同的事
- 面板 (Pane)：把一个窗口分成多个格子，可以同时看代码、跑命令、看日志


## tmux 使用场景

### 场景1：SSH 远程连接

```bash 
# 没有 tmux 时：
你 ssh 登录服务器，开始跑一个需要 2 小时的训练脚本
然后你的网络断了 → 脚本进程被杀掉 → 2 小时白跑 😭

# 有 tmux 时：
你 ssh 登录，在 tmux 里跑脚本
然后关掉电脑回家，明天回来：
ssh 登录 → tmux attach → 脚本还在跑！ ✨
```

### 场景2：同时做多件事

```bash 
# 将一个 tmux 终端会话，分割成多个窗口：
┌─────────────────┐
│ vim 写代码      │ ← 左边写代码
├─────────────────┤
│ npm run dev     │ ← 右下跑服务
│                  │
│ tail -f logs    │ ← 左下看日志
└─────────────────┘
```

### 场景3：团队协作

两个人可以同时 attach 到同一个 tmux 会话，实时看到对方的操作，就像在远程结对编程。


## tmux 基本操作

```bash
# 启动新会话，默认会话名称为 0 
tmux                   
# 创建名为 mysession 的会话 
tmux new -s mysession  
# 列出所有会话
tmux ls                 
# 连接到已创建的会话
tmux attach -t mysession
# 删除指定会话
tmux kill-session -t mysession 
# 删除所有会话
tmux kill-session
```

## tmux 快捷键操作

tmux 的所有功能都通过快捷键触发，前缀键默认是 `Ctrl+b`

注意：以下快捷操作都是先按下 `Ctrl+b` 松开后，再按下对应的操作键，如 `c`

注意：所有快捷键必须要在英文输入法状态操作，如果快捷键不起作用，请检查当前输入法状态是否为英文状态。


### 查看所有快捷键

```bash
Ctrl+b ?
```

### 一个终端会话创建多个窗口

```bash
# 启动名为 hello 的会话
tmux new -s hello

# 一个会话下，可以多个窗口，创建窗口的命令如下
Ctrl+b c 

# 切换到指定编号的窗口
Ctrl+b 0-9	

# 重命名当前窗口
Ctrl+b ,
```

### 一个窗口分割多个面板

```bash
# 垂直分割面板，注意 % 输入要按 shift
Ctrl+b % 

# 水平分割面板
Ctrl+b 双引号

# 切换面板
Ctrl+b 方向键

```

### 后台运行会话

```bash
Ctrl+b d
```

## 实战：OpenClaw + Tmux 的黄金组合

```bash
# 在一个 tmux 窗口里运行网关
tmux new -s openclaw-gateway
openclaw gateway --verbose
++
# 按 Ctrl+b d 脱离会话，网关继续运行

# 另一个窗口运行 dashboard
tmux new -s openclaw-web
openclaw dashboard

# 随时回来查看：
tmux attach -t openclaw-gateway
```
