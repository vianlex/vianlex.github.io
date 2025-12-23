# Docker 权限配置

Docker 安装过程中，会调用系统命令自动创建一个名为 docker 的用户组，并将 docker 命令所属于组设置为 docker 用户组，如果当前用户提示执行 docker 命令提示没有权限时，可以使用 sudo 来运行 docker 命令，或者将当前用户添加到 docker 组，将用户加入 docker 组后，当前用户才有 docker 组的权限。

在 Docker 中，用户组来实现权限控制，使得系统更为简洁有效。以下是一些主要优点：

1. 安全性: 通过 Docker 组控制谁，谁就可以运行 Docker 命令，确保只有授权用户可以管理容器。
2. 便利性：用户无需显式地使用sudo，提高了用户的操作效率。
3. 易于管理：只需通过组管理即可以控制多个用户的权限，简化了系统管理员的工作。


## 查看 docker 用户组

docker 安装过程中会自动创建一个 docker 用户组，我们通过以下命令 docker 用户组是否已创建

```bash

sudo cat /etc/group | grep docker

``` 

docker 用户组不存在则新建一个

```bash

# 创建用户组
sudo groupadd docker
# 给用户组设置权限
sudo a+rw /var/run/docker.sock

```

## 将当前用户添加 docker 用户组

```bash

sudo gpasswd -a ${USER}  docker

```

## 临时切换到 docker 组

```bash

newgrp  docker

```

## 重启 docker 服务

```bash

sudo service docker restart

```
