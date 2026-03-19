# Window OpenSSH 服务安装

## OpenSSH 安装和启动服务

### 安装方式一

1. 从地址 `https://github.com/PowerShell/Win32-OpenSSH/releases` 下载 ZIP 安装包
2. 将 ZIP 解压到 `C:\Program Files\OpenSSH` 目录下
3. 运行命令 `powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1` 安装即可。
4. 运行命令：`Set-Service -Name sshd -StartupType Automatic` 设置开机自动启动服务
5. 启动 ssh 服务：`` 后，即可通过 ssh 客户连接。

### 安装方式二

1. 从地址 `https://github.com/PowerShell/Win32-OpenSSH/releases` 下载 msi 安装包，然后双击运行安装即可。默认会安装在 `C:\Program Files\OpenSSH` 目录下。
2. 运行命令：`Set-Service -Name sshd -StartupType Automatic` 设置开机自动启动服务
3. 启动 ssh 服务：`Start-Service sshd` 后，即可通过 ssh 客户连接。

## OpenSSH 服务配置

OpenSSH 服务的工作目录，默认是：`C:\ProgramData\ssh`
OpenSSH 服务的配置文件是：`C:\ProgramData\ssh\sshd_config`，端口等等其他配置，需要修改时，直接修改该文件，然后运行命令 `Restart-Service sshd` 重启 sshd 服务即可。

注意：在 sshd_config 指定了密钥登录时是，客户端的公钥是默认保存在 `C:\ProgramData\ssh\administrators_authorized_keys` 文件中，不是 ~/.ssh/authorized_keys 文件中，如果想保存到用户目录下需要注释掉以下配置。   

```bash
# 使用用户目录中 authorized_keys 的文件需要注释 sshd_config 文件中以下配置
#Match Group administrators
      #AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

Window OpenSSH 服务默认禁用了使用 SHA-1 哈希算法的 RSA 签名，需要在 sshd_config 配置文件中开启，RSA 密钥支持，配置如下：

```bash
# 允许基于公钥的认证
PubkeyAuthentication yes
# 显式启用 ssh-rsa 算法 (针对主机密钥和用户密钥)
# 允许 ssh-rsa 作为用户公钥算法
PubkeyAcceptedAlgorithms +ssh-rsa
# 允许 ssh-rsa 作为主机密钥算法 (如果服务器本身使用的是 rsa 密钥)
HostKeyAlgorithms +ssh-rsa
```


## OpenSSH 服务常用命令

```bash
# 设置服务开机自启
Set-Service -Name sshd -StartupType Automatic

# 启动服务
Start-Service sshd

# 重启服务
Restart-Service sshd
```

## OpenSSH 防护墙配置

OpenSSH 服务的防火墙入站规则，安装 OpenSSH 服务的时候，已经默认配置好了，如果没有配置，可以通过以下两种方式配置：

### 图形界面方式配置

1. 通过命令 `wf.msc` 快捷打开防火墙配置界面
2. 查看【入站规则】列表中，中是否存在名为 `OpenSSH` 的入站规则，如果没有，在【操作】列表中点击【新建规则】，然后配置即可。
3. 入站配置好后，需要运行命令 `Restart-Service sshd` 重启服务

### 命令行方式配置

1. 通过以下命令行配置 inbound rule(入站规则)

```bash
# -Name 指定入站规则的名称为：OpenSSH Server 
# -Enabled 表示启动
# -Direction 防火墙的拦截方向，表示入站规则
New-NetFirewallRule -Name sshd -DisplayName "OpenSSH Server" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

2. 入站配置好后，需要运行命令 `Restart-Service sshd` 重启服务
