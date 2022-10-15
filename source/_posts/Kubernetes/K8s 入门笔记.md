---
title: K8s 入门笔记
---
## 1. 介绍
Kubernetes 的简称 K8s，Kubernetes 是 Google 开源的容器编排工具。kubernetes 的本质是一组服务器集群，集群的每个节点上运行特定的程序组件，通过组件去实现资源管理的自动化。

## 2. K8s 组件
一个 K8s 集群主要是由控制节点（Master node）、工作节点（Work node）构成，每个节点上都会安装不同的组件, 如下图所所示(图片来源于网络)：
![图一](/images/k8s-架构图.png)
### 2.1 Master 节点 
Mater 节点负载整个集群的管理和和控制、以及负责集群的决策。
| 组件 | 作用 |
|--    | --  |
| apiserver |提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制 |
| controller-manager | 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等 |
| scheduler | 负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上 |
| etcd | 保存了整个集群的状态（集群中各种资源对象的信息） |
### 2.2 Node 节点
Node 真正运行工作负载的节点，通过 kube-proxy 管理应用服务的访问入口，包括集群内 pod 到 service 的访问，以及集群外访问 service。
|组件|作用|
| -- | -- |
| Kubelet | 负责维护容器的生命周期，即通过控制 docker、来创建、更新、销毁容器 |
| Kube-Proxy | 负责提供 Node 节点应用集群的服务发现和负载均衡，Internet(或者用户)访问应用前必须经过 Kube-Proxy |
| docker | 负责镜像管理以及Pod和提供容器的真正运行接口, 容器运行时接口（Container Runtime Interface；CRI），是 kubelet 和容器运行时之间通信的主要协议，让 kubelet 能够使用各种容器运行时，无需重新编译集群组件 |
### 2.3 kubectl 
kubectl 就是 Kubernetes API 的封装的一个客户端命令行，用于控制和操作整个 K8s 集群，类似于 docker 中的 docker 命令。

## 3. K8s 资源的理解
K8s 组件是支持 K8s 平台运行的软件，是系统运行的进程，只有组件工作才能创建出资源，查看 K8s 资源使用如下的命令:
```
kubectl api-resource  
```
命令显示的结果如下：
|  结果标题 | 结果说明 |
| --   | --   |
| NAME | 资源名称 |
| SHORTNAMES | 资源名称简写 |
| APIVERSION  | 资源版本 |
| NAMESPACED | 是否可使用命名空间隔离，true 是， false 否 |
| KIND | 资源类型 |

### 3.1 资源的命名空间


### 3.2 定义资源文件
定义某个一类型的资源是可以用使用 `kubectl explain` 命令查看需要定义的资源属性，如以下例子：  
``` 
kubectl explain pod

在进一步看里面的属性定义，如 metadata 使用如下命令：
kubectl explain pod.metadata
``` 
