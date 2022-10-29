---
title: docker 安装笔记
---
## 使用 YUM 命令安装
1. 安装 docker 依赖软件
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```
2. 添加 docker 软件源
```bash 
# 查看是否 docker 软件源文件, 不存在，则用 wget 下载
ls -il /etc/yum.repos.d/docker-ce.repo 
# 下载 docker 软件源文件，并保存到 /etc/yum.repos.d/docker-ce.repo
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```
3. 安装 docker 
```
# 查看可安装的版本 
yum --showduplicates list docker-ce
# 安装指定的版本
yum -y install docker-ce-19.03.13-3.el8
```
4. 设置 docker 开机自启,并启动 docker
```
systemctl enable docker && systemctl start docker
```
5. 查看 docker 版本，如果显示成功，表示安装成功
```
docker --version
```

## CentOS8 使用 dnf 安装最新版
1. 安装 Docker 存储驱动的依赖包，使用的 dnf 命令，centos8 开始使用dnf替代yum管理软件
```
dnf install -y device-mapper-persistent-data lvm2
```
2. 添加 Docker 软件源
```
dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
3. 查看已添加的 Docker 软件源, 如果命令结果有返回 docker-ce.x86_64 说明添加成功
```
dnf list docker-ce
```
4. 开始安装 Docker
```
dnf install -y docker-ce --nobest
```
5. docker 启动相关命令
```bash
#重新加载配置文件
systemctl daemon-reload   
#运行Docker守护进程
systemctl start docker     
#停止Docker守护进程
systemctl stop docker      
#重启Docker守护进程
systemctl restart docker  
#设置Docker开机自启动 
systemctl enable docker   
```

## 手动离线安装
1. 根据需要下载指定的 docker 安装包
```
https://download.docker.com/linux/static/stable/x86_64/docker-19.03.10.tgz
```
2. 解压 docker 安装包
```
tar -xvf docker-19.03.10.tgz
```
3. 将解压出来的 docker 目录复制或者移动到 /usr/bin 目录中
```
mv docker /usr/bin/
``` 
4. 创建 docker.service 文件，并将文件复制到目录 `/etc/systemd/system/` 中，并添加可执行权限 `chmod +x  /etc/systemd/system/docker.service`，目的是使用 systemd 管理和控制 docker ，以守护进程的方式运行，方便设置开机自启。docker.service 的文件内容，如下：
```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
```
5. 设置开机自启和启动 docker 
```
systemctl enable docker && systemctl start docker
```
6. 卸载，直接删除 docker.service 文件和复制到 /usr/bin 中的 docker 目录即可
