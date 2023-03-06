---
title: Docker Swarm 笔记
---
## Swarm 介绍
Swarm 是多个以 Swarm Mode 运行的 Docker 主机组成的集群，Swarm 集群中的 Docker 主机可以充当管理节点（Manager）、工作节点（Worker）或者两者都可以。
- 管理节点：主要负责整个集群的管理工作包括集群配置、服务管理等所有跟集群有关的工作。
- 工作节点：主要负责执行运行服务的任务。
![Swarm架构图](/images/docker-swarm.png)



## Swarm 基本概念
Swarm 集群管理涉及的抽象对象有三个分别是 Node、Task、Service。

- 节点(node)：Swarm 的管理节点(manager)和工作节点(worker)的统称。
- 任务(Task)： Task 是 Swarm 中的最小的调度单位，一个运行的容器就是一个 Task。
- 服务(Service)：Service 是由一组 Task 组成构成。可以把 Task 看作是 k8s 的 Pod，Service 相当于 K8s 的 Deployment 等工作负载。Service 又分为以下两种：
  1. replicated services 按照一定规则在各个工作节点上运行指定个数的任务。
  2. global services 每个工作节点上都会运行 Service 中的任务。
- 负载均衡(Load balancing)：Swarm 通过 ingress 负载均衡暴露 Service 外部访问。如果你没有定义端口，则 Swarm 会默认暴露 30000-32767 范围内的一个端口。当有请求访问服务时，Swarm 通过内部的负载均衡将请求分发到相应的 Task 中。 


注意：当你通过 Docker Swarm 创建一个 service 时，你定义了它的理想状态（副本数、网络、存储资源、对外暴露的端口等）。Docker会维持它的状态，例如，如果一个 worker node 不可用了，Docker Swarm 会调度不可用 node 上的 task 到其他 nodes 上。运行在容器中的一个 task，是 swarm service 的一部分，且通过 swarm manager 进行管理和调度，和独立的容器是截然不同的。Swarm service 相比单容器的一个最大优势就是，你能够修改一个服务的配置：包括网络、数据卷，不需要手工重启服务。Docker 将会更新配置，把过期配置的 task 停掉，重新创建一个新配置的容器。

## Swarm 节点管理
1. 查看节点信息
```bash
docker node ls 
```
2. 获取添加工作节点或者管理节点的 token
```bash
# 获取加入工作节点的 token
 docker swarm join-token worker
# 管理节点
 docker swarm join-token manager
```
3. 节点脱离 swarm 集群
```bash
# -- force 表示强制离开集群
docker swarm leave [--force]
```
4. 删除集群某个节点
```bash
docker node remove <节点名称|节点ID>
```


## 部署服务
通过 ` docker service create `命令可以快速部署服务，该命令语法格式为：`docker service create <--name 服务名字> <--replicas task副本数> <image> [容器运行后要执行的命令]`，常用可选参数如下：
- --name 指定服务的名字
- --replicas 指定 Task 的副本数量
-  -w, --workdir 指定容器的工作目录
- env 指定环境变量，如` -env JAVA_OPS=""`
- --mount 容器目录挂载到主机目录，如：`--mount host/xxx/xxx:container/xxx/xxx` 或者 `--mount type=bind,src=<VOLUME-NAME>,dst=<CONTAINER-PATH> `，推荐使用第二种新语法，比较明了
- -p, --publish port 指定端口，如：`-p host-port:container-port` 或者 `--publish published=<host-port>,target=<container-port>`，推荐使用第二种新语法，比较明了
- --update-delay 设置滚动更新延迟时间，即为多个副本任务的更新间隔时间
- --update-parallelism 每次并发更新的任务数量
- --mode 指定服务的运行模式可选值有：replicated(按一定规则调度 task 到合适的 node), global(每个工作节点都运行一个 task), replicated-job, global-job

1. 创建服务
```bash
# 创一个名 test-swarm 的服务，运行2个副本任务容器
docker service create --name test-swarm  -p 2000:80  --replicas 2 nginx:latest

 endpoint_mode=dnsrr
```
2. 查看正在运行的 services，使用如下命令
```bash
docker service ls 
```
3. 查看 service 信息，使用命令 `docker service inspect --pretty <服务名>`
```bash
# 查看名为 test-swarm 的 service 详细信息
docker service inspect --pretty test-swarm
```
4. 使用命令`docker service ps <服务名>`查看服务 Task 的运行信息
```
# 查看 test-swarm 服务中 task 的运行情况   
docker service ps test-swarm
```
5. 使用命令`docker service update <选项参数> <服务名>`，具体支持哪些配置的更新改动可以使用帮助命令查看 `docker serivce update --help `
```bash
# 新增 3000:80 端口映射
docker service update --publish-add 3000:80
# 删除端口映射
docker service update --publish-rm 3000:80

```
6. 扩容增加任务副本数量
```bash
# 将 test-swarm 服务的任务副本数量修改为 3
docker service scale test-swarm=3
# 等同以上命令
docker service update --replicas 3 test-swarm
```

7. 删除服务，使用命令 `docker service rm <服务名>`
```bash
# 删除 test-swarm 服务
docker service rm test-swarm
```

8. 查看服务的日志，使用命令 ` docker service logs [OPTIONS] <SERVICE|TASK> `
```bash
# 查看服务 test-swarm 的日志
docker service logs -f --tail=200 test-swarm
```

## 部署多个服务
批量部署多个服务需要使用 docker stack 命令和 docker-compose.yaml 配置文件，docker stack 是 docker serice 在基础上封装了一层，一个 stack 管理着多个 serivces，docker stack 常用命令如下：
- docker stack deploy 部署或者更新 stack 
- docker stack ls 查看已部署的 stack 列表
- docker stack ps 查看 stack 部署的服务下的任务列表  
- docker stack rm 删除一个 stack
- docker stack services 查看 stack 部署的所有服务 


1. 定义 stack 使用的 docker-compose 文件，[配置说明](https://docs.docker.com/compose/compose-file)
```yaml
version: '3.7'

services:
  my-web:
    image: nginx:latest
    ports:
      - target: 80 # 容器端口 
        published: 8080 # 主机端口
        protocol: tcp # 可选值 tcp、udp
        mode: ingress # 可选值 host、ingress 默认 ingress
    # 注意如果没有配置 deploy 属性的话，ports 属性中 mode 配置的 ingress 模式是不起作用的，会模式为 host 模式
    deploy:
      mode: replicated # 可选值 replicated 和 global 默认值 replicated
      replicas: 2 # 配置服务副本数量
      endpoint_mode: vip # 配置服务发现模式，可选值 vip 和 dnsrr, 默认值 vip
      # 配置重启策略
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s

```

2. 使用 ` docker stack deploy ` 命令部署服务，命令语法 `docker stack deploy [OPTIONS] STACK(表示 stack 名)`，常用选项如下：
- --compose-file , -c	 指定 docker-compose 配置文件
```bash
# 通过 docker-compose 配置文件部署一个名为 mystack 的 stack，部署后生成的服务名格式为：stack 名 + docker-compose 文件配置的服务名，即 mystack_my-web
docker stack -c docker-compose.yml mystack 
```

## 服务发现和负载均衡
Service 相当于它的所有 Task 的一个反向代理，Service 的服务发现和负载均衡是利用 Linux 内核的 iptables 和 IPVS 的功能来实现。Docker Swarm 提供了两种不同机制的服务发现，分别是：
- VIP(Virtual IP)：Swarm 创建应用服务时，会创建一个应用服务的虚拟 IP，Docker 内嵌的 DNS 服务会维护该虚拟 IP 和应用服务名记录，同时在每个应用服务容器内创建一个 ingress_sbox 网络命令空间，然后利用 Linux 内核的 iptables 和 IPVS 在 ingress_sbox 网络命名空间通过 VIP 转发请求到服务容器的 IP，从而实现负载均衡。 
- DRR(DNS round-robin)：通过 Docker 引擎提供的内嵌 DNS 服务，维护服务名对应 task 容器的 IP 地址列表，当前请求访问服务时，Docker DNS 解析服务名，获取服务所有任务容器的 IP 地址列表，然后将请求转发到其中任意一个(一般默认是第一)，从而实现负载均衡。

Docker Swarm 通过  ingress overlay 网络实现了网格路由(routing mesh)功能。在 Swarm 集群中所有的工作节点(worker)和管理节点(manager)都会参与到网格路由中。当部署的服务通过 `--publish` 暴露端口时，由于所有的节点都在网格路由(routing mesh)中，所以全部的节点都会是监听` --publish `暴露的端口，因此访问集群中的任何节点都可以访问到服务。 如果不想每个节点都监听服务暴露的端口，则需要将暴露的模式指定为 host 模式，来禁用路由网格，如 `--publish published=8080,target=80,mode=host `。
![网格路由](/images/docker-swarm网络路由.png)

路由网格模式只支持 VIP 的服务发现模式，如果使用 DRR 模式是不起作用的。可以通过 `--endpoint_mode: vip | drr` 指定服务选择那种服务发现，默认是 vip


## 管理节点和工作节点的心跳
```bash
# 修改检测心跳1分钟
docker swarm update --dispatcher-heartbeat 60s
```

## 参考链接
1. https://docs.docker.com/engine/swarm/swarm-tutorial/
2. https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/
3. https://docs.docker.com/engine/swarm/networking/