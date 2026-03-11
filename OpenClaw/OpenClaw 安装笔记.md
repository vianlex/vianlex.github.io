# OpenClaw 安装笔记

## 手动安装

### 前置条件

- 系统必须安装 NodeJs 环境，并且版本必须大于 22.x

### 安装命令

```bash
npm install -g openclaw@latest
```

### 初始化配置

### 初始化命令

```bash
# 初始化 openclaw，不安装网关后台进程服务和网关开机自启
# 后续通过 openclaw gateway install 命令再安装，或者不需要后台进程服务，直接前台运行启动即可（命令：openclaw gateway）
openclaw onboard 
# 初始化 openclaw 和安装网关的后台进程服务和开启自动运行网关 (推荐)
openclaw onboard --install-daemon
```

### 初始化向导主要进行以下配置：

第一步：选择运行模式

```text
提示 `I understand this is personal-by-default and shared/multi-user use requires lock-down. Continue?`
选择 Yes 按「个人使用」模式运行即可
```

第二步： 选择 Onboarding mode（向导模式）

```text
提示 QuickStart (Configure details later via openclaw configure.) 和 Manual
选择 QuickStart 快速配置模式
```

第三步：Model/auth provider 配置 AI 模型提供商

```text
选择自定义模型提供商（定义美团模型，目前是免费的）
API Base URL 输入：https://api.longcat.chat/openai
选择现在粘贴密钥信息（ Paste API key now）
API Key 输入：ak_2Od9F79Ud0Gq3Oc8Le7Y16Fx71212
选兼容端点（Endpoint compatibility）
选择 OpenAI-compatible 即可
设置 model ID（模型ID）,输入：longcat
```

第三步：选择 channels（机器人通道）

```text
选择现在跳过（Skip for now），后续再通过 openclaw channels add 命令配置
```

第四步：选择 Search provider 搜索提供商

```text
选择现在跳过（Skip for now），后续再通过 openclaw configure --section web 命令配置
```

第五步：选择 Kills 技能，选择跳过后续再配置

第六步：全部选择 NO 即可

第七步：是否启动 hooks 不知道什么意思，暂时跳过。

第八步：如果初始化机器人（ How do you want to hatch your bot? ）

```text
1. 通过 TUI 终端，打开聊天机器人（Hatch in TUI (recommended)）
2. Web UI 前端页面，打开聊天机器人会话（Open the Web UI）
3. 跳过后续再操作

我们选择 TUI 终端，然后随便输入一个问题，如果有回答，说明大模型配置成功了。
推出 TUI 终端模式，直接使用两次 ctrl+c 即可。
```

### 初始化完成后查看网关运行状态

```bash
openclaw gateway status
```

## channels 通道

### 通道基本命令

```bash
# 查看当前有哪些通道
openclaw channels list
# 添加通道，并指定通道属于哪个 agent 
openclaw channels add --type feishu --agent taizi
```

### 安装飞书通道插件

1、运行以下命令安装（最新版，已内置飞书通道插件，不需要安装，直接配置即可）

```bash
# 运行命令，然后选择安装飞书飞书插件
openclaw channels add 
# 1. 安装成功后，输入密钥信息
# 2. 链接模式（Feishu connection mode）选择 WebSocket 模式
# 3. 指定飞书域名 Which Feishu domain? 国内选择 feishu.cn
# 4. 群组聊天策略，Allowlist（只有群组指定人能跟飞书机器人聊天） 和 Open（表示群组中所有人都可以和飞书机器人聊天）
# 5. requireMention 是否需要 @机器人，才会做回复，为 false, 只要发送消息到群里，不需要@机器人，就会自动回复。 

# 安装好后，重启网关
openclaw gateway restart
```

2、查看 `~/.openclaw/openclaw.json` 配置文件，检查飞书密钥信息是否已配置成功，如果没有手动添加以下配置

```js
// 如果配置格式不对，我们可以通过 openclaw doctor --fix 修复 openclaw 
// 配置完后，要重启网关 openclaw gateway restart
channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "我的AI助手",
        },
      },
    },
  },
```


3、飞书应用事件配置

- 订阅方式选择：长链接（前面飞书插件的配置是 WebSocket）
- 订阅事件：添加接收消息事件


4、在飞书聊天界面，搜索对应的飞书应用，然后打开并 @机器人，随便输入一个内容

```bash
# // 机器人会回复以下内容，我们复制 openclaw pairing approve feishu RWXWA2JW 执行配对即可。
OpenClaw: access not configured.
Your Feishu user id: ou_975ef5cace9269fc93a757fbc235aefc
Pairing code: RWXWA2JW
Ask the bot owner to approve with:
openclaw pairing approve feishu RWXWA2JW
```

5、查看通道详情 

```bash 
# 查看通道详情
openclaw channels status --probe

# 查看飞书是否配对成功
openclaw pairing list --channel feishu
```



## GateWay 网关问题

### 问题1

出现 `Gateway service check failed: Error: systemctl is-enabled unavailable: Command failed: systemctl --user is-enabled openclaw-gateway.service` 问题和`Error: systemctl is-enabled unavailable: Command failed: systemctl --user is-enabled openclaw-gateway.service` 问题的解决方式如下：


```bash
# 1. 确认用户级 systemd 是否正常运行
systemctl --user status

# 2. 查找确认gateway 服务配置是否存在
find /etc/systemd /home/$USER/.config/systemd -name "openclaw-gateway.service"

# 3. 如果配置文件不存，直接手动创建即可（目录是 ~/.config/systemd/user/openclaw-gateway[-<profile>].service 守护进程文件）
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target

# 4. 启动网关
systemctl --user enable --now openclaw-gateway.service

# 5. 查看网关是否启动成功
systemctl --user status openclaw-gateway.service

# 6. 如果还启动不存在，则重新安装网关，会自动生成 openclaw-gateway.service 文件，并启动
# 注意如果不存在旧的 openclaw-gateway.service 文件该命令会运行失败，我可以先创建一个旧的
openclaw gateway install --force
systemctl --user start openclaw-gateway.service

# 6. 查看网关运行日志
journalctl --user -u openclaw-gateway.service -n 20 --no-pager
```

### 问题2

打开 `http://127.0.0.1:18789/`, 如果你看到 `unauthorized`，运行命令 `openclaw dashboard` 然后复制带有 `token` 的链接重新打开即可。

注意：该 token 可以在 ~/.openclaw/openclaw.json 的文件中查看到, 路径为 `gateway.auth.token`。

### 问题3

网关启动失败，通过 `journalctl --user -u openclaw-gateway.service -n 200 --no-pager` 命令查看日志，提示 `Gateway start blocked: set gateway.mode=local (current: unset) or pass --allow-unconfigured.` 问题，解决方式如下：

```bash
# 方式1：~/openclaw/openclaw.json 在找到网关配置，将配置设置为 local
gateway.mode = "local"

# 方式2，启动网关时，跳过检查
openclaw gateway start --allow-unconfigured
```








## 官方文档

```bash
https://docs.openclaw.ai/zh-CN/tools/plugin
```
