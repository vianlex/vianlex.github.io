# Docker 常用命令

## docker 镜像查看、删除等相关命令

### 1. docker images 命令查看本地镜像仓库中的镜像

docker images 语法格式 `docker images [OPTIONS] [REPOSITORY[:TAG]]`，可选参数如下：
- docker images -a	列出本地所有的镜像
- docker images -f 过滤查询镜像
- docker images -q	列出显示镜像 ID

```bash

# 查看所有镜像
docker images 

# 根据 <REPOSITORY>[:TAG] 显式的过滤查看镜像，注意当使用 * 模糊匹配时，无法匹配 / 符号，如匹配 vianlex/nginx:dev-latest 镜像，支持以下方式匹配
docker images vianlex/nginx
docker images vianlex/nginx:dev-latest
docker images "*/nginx*"
docker images "*/nginx:dev-*"


# 只是查看镜像的 id
docker images -q

# 匹配标签(tag)为<none>的镜像
docker images -f="dangling=true"

# 根据 reference=<REPOSITORY>[:TAG] 模糊匹配镜像，注意匹配时，reference=<REPOSITORY>[:TAG] 中的 / 符号不能无法使用 * 模糊匹配，如匹配镜像 vianlex/nginx:dev-latest
docker images -f="reference=*/nginx" 或者 docker images -f "reference=*/nginx"
docker images -f=reference="*/nginx" 或者 docker images -f  reference="*/nginx" 
#或者
docker images -f="reference=*/nginx:dev-*"
docker images -f=reference="*/nginx:dev-*"

# 根据 before 或者 since 过滤镜像，格式为 before=<REPOSITORY>[:TAG] 或者 before=image-id
docker images -f before="eb4d124f64cd" # 根据镜像 ID 查找创建时间大的镜像
docker images -f since="vianlex/nginx:dev-latest" # 根据镜像名称查找创建时间小的镜像

# 根据 label=<key> 或者 label=<key>=<value> 过滤镜像，label 是在 dockerfile 中定义的，可以使用 docker inpsect 命令可以查看镜像的 labels
docker images -f="label=version"
#或者
docker images -f="label=version=v1"

# 格式镜像列表的输出格式
docker images --format "{{.Repository}} -- {{.ID}} -- {{.Size}}"
# 或者
docker images --format "{{.Repository}} {{.ID}} {{.Size}} {{.di}}"
# 或者 
docker images --fotmat "{{.ID}}"

```


### 2. docker rmi 命令删除 docker 镜像

docker rmi 命令语法格式：`docker rmi [ repository:tag | imageId  ] `，可选参数 -f 指定强制删除

```bash

# 根据镜像仓库加标签删除 
docker rmi nginx:latest

# 根据镜像 ID 删除
docker rmi 43154ddb57a8

# 根据镜像 id 批量删除 
docker images -q | xargs docker rmi 
# 或
docker rmi $(docker images -q)
# 删除 tag 是 none 的所有镜像
docker rmi $(docker images -f "dangling=true" -q)
# 删除镜像，xargs 的 -r 参数表示如果参数为空则不执行后面的命令
docker images -f="dangling=true" -q | xargs -r docker rmi

```


## docker 操作容器的常用命令

### 1. docker run 命令

用于创建并启动 Docker 容器，常用参数如下：
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
### 2. docker ps 查看容器

常用的可选参数
- -q, --quiet 只显示容器的 ID
- --format 格式化输出结果
- -f, --filter 过滤显示结果，使用 key=value 的方式过滤，如` -f name=myNginx `，可选的过值对如下：
    - id 根据容器的 id 过滤
    - name 根据容器名称过滤
    - ancestor 根据镜像信息过滤，如 ` -f ancestor <image-name>[:<tag>] | <image id> `
    - status 根据状态进行过滤，可选值有 created, restarting, running, removing, paused, exited, or dead
    - publish or expose 根据端口过滤，如 ` -f  publish <port>[/<proto>] ` 或者 `-f  expose <startport-endport>/[<proto>] `
    - before or since  根据创建时间或者容器名的前后过滤
    - label 根据标签过滤

```bash

docker ps --filter "label=color"

docker ps --filter "name=xxxxx"

docker ps --filter status=running

docker ps --filter ancestor=nginx:latest

# 显示创建比容器 9c3527ed70ce 早的所有容器 
docker ps -f before=9c3527ed70ce

```

### 2. docker stop 停止正在运行的容易

docker stop 语法格式 `docker stop [--time , -t] container [container...]` 停止一个或者多个正在运行的容器，可选参数 -t 指定运行命令多少秒后停止容器

```bash

# 根据容器 id 停止容器
docker stop 9c3527ed70ce

# 根据条件过滤停止容器
docker stop  $(docker ps -q -f name=myNginx)

```

### 3. docker start 启动容器

```bash

# 根据容器名称或者id启动容器
docker start [container-name|container-id]

```

### 4. docker restart 者重启容器

可选参数 `--time, -t ` 指定多少秒后停止并重启容器

```bash

# 根据容器 id 重启容器
docker restart 9c3527ed70ce
docker restart  $(docker ps -q -f name=myNginx)

```

### 5. docker rm 删除容器

可以删除一个或者多个容器，可参数 -f 强制删除

```bash
# 删除容器，
docker rm [container-name|container-id]

```

### 6. docker logs 查看容器运行日志

```bash
#查看实时的日志：
sudo docker logs -f --tail 100  container_id 或者 container_name

#查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2018-02-08" --tail=100 container_id

#查看最近30分钟的日志:
docker logs --since 30m container_id

#查看某时间段日志：
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" container_id

```

### 7. docker exec 命令

在宿主机运行容器中的命令，使用例子如下：

```bash

# 进入容器的终端命令行
docker exec -it [container_id | container_name] [/bin/sh | /bin/bash] 

# 查看容器的文件内容
docker exec [container_id | container_name]  cat /logs/ils.2020-08-19.log 

# 运行容器的命令
docker exec -it [container_id | container_name] nginx -s reload 

```

### 8. docker cp 命令

将容器中的文件复制到宿主机中或者将宿主机的文件复制容器中

```bash

# 将容器文件复制到宿主机中
docker cp [container_id | container_name]:/xxx/xxx  /home/xxxxxx

# 将宿主机文件复制到容器中
docker cp /home/xxxxxx  [container_id | container_name]:/xxx/xxx

```


## 归档镜像导入导出

### 1. docker save 命令将 Docker 镜像保存成 tar 包

docker save 命令语法 ` docker save [ -o | --output | > ]  filename.tar [ reposity:tag | imageId ] `

```bash

#第一种方式
docker save -o flannel.tar rancher/mirrored-flannelcni-flannel:v0.20.0
#第二种方式
docker save  > flannel-cni-plugin.tar fcecffc7ad4a

```
### 2. docker load 命令将镜像归档文件导入镜像仓库

docker load 命令语法 `docker load [ --input | < ] filename.tar` 

```bash

# 第一种方式
docker load --input filename.tar

# 第二种方式
docker load < filename.tar

# 如果导入的镜像没有仓库和标签，使用 docker tag 命令修改
docker tag ancher/mirrored-flannelcni-flannel:1.0.0  container_id

```





## 参考连接
1. https://docs.docker.com/engine/reference/commandline/images/