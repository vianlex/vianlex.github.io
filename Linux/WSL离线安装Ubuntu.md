# WSL 离线安装 Ubuntu


## 前置条件

1、虚拟化要求

- BIOS/UEFI 已开启 Intel VT-x / AMD-V 硬件虚拟化

2、启用 WSL 与虚拟机平台（管理员 PowerShell）

```bash
# 启用虚拟机子系统
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
# 启用 WSL 2 必需的虚拟机平台
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
# 参考地址：https://learn.microsoft.com/zh-cn/windows/wsl/install-manual 
```

3、下载内核安装包（已安装过，则忽略）

下载地址：https://github.com/microsoft/WSL/releases 下载完成后，双击安装，安装完成后重启即可。

注意：如果系统已经存在 WSL setting 应用，则说明已经安装过内核了，不需要再安装。


## 镜像下载地址

Ubuntu Releases 站点：https://releases.ubuntu.com/noble/


## 导入安装镜像

镜像安装命令格式：`wsl --import  <自定义系统名称> <安装路径> <镜像文件路径>`

```bash

wsl --import Ubuntu-24.04 D:\WSL\Ubuntu2404 C:\Users\Amias\Downloads\ubuntu-24.04.4-wsl-amd64.wsl

```

## 启动系统和用户配置

1、手动导入镜像后，通过以下命令进入系统 

```bash
# Ubuntu-24.04 是我们导入时，指定的系统名称
wsl -d Ubuntu-24.04
```

2、手动导入镜像，我们进入系统时，默认是以 root 账号登录的，出于安全考虑，我们可以创建一个普通用户，创建普通用户方式如下：


```bash
# 创建名为 sysadmin 的用户，-m 表示强制创建用户目录（默认路劲为： /home/sysadmin） -d 指定 home 目录，-s 指定 shell
sudo useradd -m -d /home/sysadmin -s /bin/bash sysadmin

# 表示将用户添加到 sudo 用户组，表示用户可以通过 sudo 执行需要 root 权限的操作
usermod -aG sudo sysadmin

# 查看用户是否创建成功
id sysadmin
cat /etc/passwd | grep sysadmin

# 设置用户的密码
passwd sysadmin 
# 设置用户 sysadmin 的密码为空白密码
passwd -d sysadmin


# 删除用户, -r 表示用户信息全部删除
userdel -r sysadmin  

```

3、指定默认登陆用户

```bash
# 第一种方式：在 /etc/wsl.confi 文件中添加以下内容
[user]
default=sysadmin 
# 配置文件修改好后，需要通过以下关闭系统，然后重新进入
wsl --terminate Ubuntu-24.04

# 第二种方式：进入系统时，直接指定用户
wsl -d Ubuntu-24.04 --user sysadmin
# 或者
wsl --distribution Ubuntu-24.04 --user sysadmin
```

4、普通用户忘记密码时，直接通过以下命令，以 root 用户进入系统，重置修改即可。

```bash
# 指定进入系统的用户
wsl --distribution Ubuntu-24.04 --user root
```

5、切换 root 用户，需要密码时，通过以下命令切换即可

```bash 
sudo -i 
```


注意：让用户 sudo 不需要密码

```bash
# 打开 /etc/sudoers 文件
visudo 
# 在文件中添加，表示用户指定 sudo 命令时，不需要输入密码
sysadmin ALL=(ALL) NOPASSWD: ALL
```



## 导出备份和迁移系统

导出系统备份命令： `wsl --export <指定导出系统的名称> <导出保存路径>`

```bash
# 导出备份，后续我们可以通过 wsl --import 迁移导入备份系统
wsl --export Ubuntu-24.04 D:\Backup\ubuntu_Ubuntu24.04.bak.tar
```

## 删除系统

如果我们不想要系统了，或者

```bash 
# 注意：会物理删除系统，及其所有文件和配置信息，并且办法再恢复！
wsl --unregister Ubuntu-24.04
```

## 跨系统访问文件

在 WSL 系统中，会将 Windows 的磁盘分区，都挂载到 /mnt 中，如果我们在 WSL 系统中想访问 Windows 的 C 直接通过 /mnt/c 访问即可。

在 Windows 系统中，如果我们想访问 WSL 系统的目录，直接资源管理地址栏中输入 `\\WSL$` 即可看到所有运行中的 Linux 目录。


## 网络代理

1、临时生效（当前终端，关闭即失效）

```bash
# WSL2 与宿主机共享网络协议栈，访问宿主机上的代理服务时，通过 cat /etc/resolv.conf 查看宿主机的自动分配 IP。
export WIN_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
export http_proxy=http://$WIN_IP:7890
export https_proxy=http://$WIN_IP:7890
export all_proxy=socks5://$WIN_IP:7891
export no_proxy=localhost,127.0.0.1,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

2、apt 工具临时走代理

```bash
sudo apt -o Acquire::http::Proxy="http://$WIN_IP:7890" update
```

3、永久生效（所有终端 / 重启后都生效）

在 ~/.bashrc 文件中添加以下内容，然后  `source ~/.bashrc` 使其生效

```bash
export WIN_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}') 
export http_proxy=http://$WIN_IP:7890 
export https_proxy=http://$WIN_IP:7890  
export all_proxy=socks5://$WIN_IP:7891 
export no_proxy=localhost,127.0.0.1,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

## WSL 启动 bash 配置文件问题

### Shell 加载配置文件的规则

WSL 中 bash 有两种核心启动模式，加载的文件完全不同：

- 登录 shell（Login）：	`bash -l 命令或者 WSL 默认启动` shell 默认加载以下配置文件   `/etc/profile → ~/.bash_profile → ~/.bash_login → ~/.profile`
- 交互 shell（Interactive）：`手动打开 bash 终端或者终端手动输 bash` 启动 shell 才会加载文件 `~/.bashrc`


WSL 启动时默认是「登录 shell」，只会加载 ~/.profile（而非 .bashrc），只有手动打开交互 shell 终端才会加载 .bashrc。

如想要在 WSL 每次启动子系统加载 `.bashrc` 文件，我们只需要在 `.bash_profile` 文件中（不存在，则创建），添加如下配置即可：

```bash
export LOCAL_BIN_PATH="/home/sysadmin/.local/bin"
export PATH="$LOCAL_BIN_PATH:$PATH"

# 检查 .bashrc 是否存在，存在则加载
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```