---
title: Docker 常用命令
---

## docker images 命令查看本地镜像仓库中的镜像
docker images 语法格式 `docker images [OPTIONS] [REPOSITORY[:TAG]]`，可选参数如下：
- docker images -a	列出本地所有的镜像
- docker images --digests	显示镜像的摘要信息
- docker images -f	显示满足条件的镜像
- docker images --format	指定返回值的模板文件
- docker images --no-trunc	显示完整的镜像信息
- docker images -q	列出显示镜像 ID
```bash
# 查看所有镜像
docker images 

# 只查看某个镜像 docker images [REPOSITORY[:TAG]]
docker images nginx # 查看所有版本
docker images nginx:latest

# 只是查看镜像的 id
docker images -q
```

## docker 容器启动、删除等相关命令
1. docker run 命令，用于创建并启动 Docker 容器，常用参数如下：
- -d, --detach=false 指定容器以后台进程的方式运行
- -i, --interactive=false 以交互模式运行容器，通常与 -t 同时使用
- -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
- --name="myNginx"	指定容器名字，方便管理
- --dns 8.8.8.8 指定容器使用的 DNS 服务器，默认和宿主一致
- --dns-search= example.com 指定容器的 dns 搜索域名，会写入到容器的 /etc/resolv.conf 文件 
- -h "linux-hostname" 指定容器的 hostname
- -w, --workdir="/opt" 指定容器的工作目录
- -e username=centeos, --env=[] 设置容器的环境变量
- --env-file=[] 从指定文件读入环境变量
- -v, --volume=[]	给容器挂载存储卷，挂载到容器的某个目录。
- –dns=[]	指定容器的 dns 服务器。
- -P 随机端口映射，容器内部端口随机映射到主机的端口
- -p 指定端口映射，格式为：主机(宿主)端口:容器端口
- –entrypoint=""	覆盖 image 的入口点。
- –env-file=[]	指定环境变量文件，文件格式为每行一个环境变量。
- –expose=[]	指定容器暴露的端口，即修改镜像的暴露端口。
- –link=[]	指定容器间的关联，使用其他容器的 IP、env 等信息。
- -net="bridge"	指定容器网络配置
- --privileged=false	指定容器是否为特权容器，特权容器拥有所有的 capabilities。
- --restart="no"	指定容器停止后的重启策略:
- --rm=false	指定容器停止后自动删除容器(不支持以 docker run -d 启动的容器)。
```bash
#创建并启动 nginx 容器
docker run -p 81:80  -v /data:/usr/share/nginx/html --name mynginx -d nginx:latest
```

## docker save 命令将 Docker 镜像保存成 tar 包
docker save 命令语法 ` docker save [ -o | --output | > ]  filename.tar [ reposity:tag | imageId ] `
```bash
#第一种方式
docker save -o flannel.tar rancher/mirrored-flannelcni-flannel:v0.20.0
#第二种方式
docker save -o > flannel-cni-plugin.tar fcecffc7ad4a
```
## docker load 命令将镜像归档文件导入镜像仓库
docker load 命令语法 `docker load [ --input | < ] filename.tar` 
```bash
# 第一种方式
docker load --input filename.tar

# 第二种方式
docker load < filename.tar
```
