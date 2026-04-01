# 飞书 lark-cli 安装笔记

## 第一章 概述

### 1.1 工具简介

lark-cli 是飞书提供的命令行工具，用于：
- 自动化飞书应用配置
- 管理飞书审批流程
- 调用飞书 API
- Webhook 事件调试

### 1.2 环境要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows/Linux/macOS |
| Node.js | 16.x+ |
| npm | 8.x+ |

---

## 第二章 在线安装

### 2.1 npm 安装

```bash
npm install -g @larksuiteoapi/lark-cli
```

### 2.2 验证安装

```bash
lark-cli --version
```

---

## 第三章 离线安装

### 下载 `lark-cli` 安装包

方式一：通过 `npm view` 命令获取下载链接，然后通过浏览器下载或者 `wget` 等命令下载安装包

```bash
# 获取 lark-cli 安装包下载链接
npm view @larksuite/cli dist.tarball
```

方式二：通过 `npm pack` 命令直接下载 `lark-cli` 安装包

```bash
# 下载 lark-cli-*.tgz 安装到当前目录 
npm pack @larksuite/cli

# 将安装下载到指定目录
npm pack @larksuite/cli --pack-destination /dist/to/lark-cli-*.tgz
```

### 离线安装 `lark-cli`

方式一：`npm install -g ` 命令, 离线全局安装

```bash
npm install -g /path/to/lark-cli-*.tgz
```

方式二：手动解压全局安装

1. 查看全局 node_modules 目录
```bash
npm root -g
```

2. 根据命令 npm install -g @larksuite/cli 在全局 node_modules 目录下，创建 lark-cli 安装目录

```bash
# 如全局 /path/nodejs/node_modules ，创建目录结果如下：
/path/nodejs/node_modules/@larksuite/cli
```

3. 将下载的 `lark-cli-*.tgz` 安装包，解压到全局目录 `node_modules/@larksuite/cli` 下

注意： `lark-cli-*.tgz` 解压后, 将 `package.json` 根目录下的所有文件复制到 `node_modules/@larksuite/cli` 目录下即可。

4. 安装 lark-cli.exe 运行命令 

```bash
# 下载 lark-cli.exe 运行命令
https://github.com/larksuite/cli/releases/download/v1.0.1/lark-cli-1.0.1-windows-amd64.zip
# 在 node_modules/@larksuite/cli 目录下创建 bin 目录，然后将 exe 文件复制到 bin 目录下即可。
```

---

## 第四章 配置

### 4.1 登录

```bash
# 交互式登录
lark-cli login

# 或使用 token 登录
lark-cli login --token <your_token>
```

### 4.2 配置应用

```bash
# 设置 App ID
lark-cli config set app_id <app_id>

# 设置 App Secret
lark-cli config set app_secret <app_secret>
```

### 4.3 查看配置

```bash
lark-cli config list
```

---

## 第五章 常用命令

### 5.1 应用配置

```bash
# 查看应用信息
lark-cli app info

# 列出所有 API
lark-cli api list

# 测试 API 调用
lark-cli api call <api_name>
```

### 5.2 审批流程

```bash
# 列出审批定义
lark-cli approval list

# 获取审批实例
lark-cli approval get <instance_id>
```

### 5.3 消息发送

```bash
# 发送消息
lark-cli message send --to <user_id> --msg-type text --content '{"text":"Hello"}'
```

### 5.4 Webhook

```bash
# 启动本地 Webhook 服务
lark-cli webhook start --port 8080

# 调试 Webhook
lark-cli webhook debug
```

---
