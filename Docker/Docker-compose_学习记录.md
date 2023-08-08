# Docker-compose 学习记录


## docker-compose 介绍

docker-compose 是用来定义和管理多容器运行的 docker 应用程序，通过 YAML 配置文件和简单的 Docker-Compose 命令就可以管理和控制 docker 容器的生命周期和日志，其可以看作是 docker 容器自动化管理工具。

docker-compose 是以 project 和 service 的概念去管理 docker 容器的，每一个容器是一个 service，每一个或者多个 docker-compose 配置文件所在的目录是一个 project, project 的名称默认是配置文件的上层目录名。

注意 docker-compose 管理的是 service, 所以 dokcer-compose 命令是以 service 维度去操作管理 docker 容器的。

## docker-compose 常用命令的简单介绍

docker-compose 命令运行时，会默认读取当前目录的 docker-compose 配置文件，如果 docker-compose 命令运行所在目录不存在 docker-compose 文件，则需要要使用 -f 指定 docker-compose 配置文件或者指定使用 -p 指定 project 的名称。

docker-compse 命令详细用法使用 ` --help ` 方式查看，如果查看 docker-compose ps 的详细用法，运行 ` docker-compose ps --help ` 命令即可。以下是 docker-compose 部分命令解释说明。

#### docker-compose pull 

根据 docker-compose 配置文件拉取镜像

```bash
# 默认拉取 docker-compose 配置文件所有 service 的镜像
docker-compose pull 

# 根据服务名称拉取单个或者多个服务的镜像
docker-compose pull [service-name]

docker-compose pull [service-name1] [serivce-name2]
```

#### docker-compose create 

创建或者重建运行中容器，命令执行成功后，容器的状态都是 Created 状态，表示只是创建了容器，并未运行容器，所以要注意重建运行中的容器后，容器的状态将由 Running 状态变成 Created 服务无法在使用，必须要 `docker-compose start ` 启动容器后才可以。

```bash
# 根据 docker-compose 配置文件中的镜像创建容器或者重建运行中的容器，容器的状态会变成 Created 状态
docker-compose create 

# 根据一个或者多个服务名称创建或者重建运行中的容器
docker-compose create [service-name]

docker-compose create [service-name1] [service-name2]
```

#### docker-compose start 

启动 Created 状态的容器，命令执行成功后，容器状态变成 Running 状态

```bash
# 根据 docker-compose 配置文件启动所有 Created 状态的容器
docker-compose start 

# 根据一个或者多个服务名启动 Created 状态的容器
docker-compose start [service-name1] 

docker-compose start [service-name1] [service-name2]
```

#### docker-compose run 

创建容器并启动容器，等价于运行 `docker-compose create` 和 `docker-compose start` 命令

```bash
# 以后台守护进程的方式创建和运行 docker-compose 配置文件中的镜像容器
docker-compose up -d 
```

#### dockerc-compose down

停止和删除 docker-compose 配置文件中的所有容器和网络

```bash
docker-compose down 
```

#### docker-compose logs 

查看容器日志

```bash
# 根据服务名查看容器的日志
docker-compose logs <service-name> -f --tail=200
```

#### docker-compose exec 

运行容器中的命令或者进入容器内容

```bash
# 根据服务名进入容器内部
docker-compose exec -it <service-name> </bin/sh | /bash>

# 根据服务名运行容器中的命令
docker-compose exec -it <service-name> ls -il

docker-compose exec -it <service-name> env # 查看环境变量

docker-compose exec -it <service-name> nginx -t # 运行 nginx 配置文件验证命令
```

#### docker-compose cp 

将容器的文件或者文件夹复制到宿主机中

```bash
#  将宿主机中的 /home/sysadmin/data/hello.txt 文件复制到容器的 /data 目录下
docker-compose cp /home/sysadmin/data/hello.txt <serivce-name>:/data

# 将容器中的 /data/hello.txt 文件复制到宿主机的 /home/sysadmin/data 目录下
docker-compose cp <serivce-name>:/data/hello.txt /home/sysadmin/data 
```




