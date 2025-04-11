# Docker 权限配置



## 新增 docker 用户组

查看 docker 用户组是否存在

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
