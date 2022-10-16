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
| docker | 负责镜像管理以及Pod和提供容器的真正运行接口(CRI) , CRI 是一个接口，用来操作容器的接口。k8s通过CRI对容器进行操作，创建、启停容器等 |

### 2.3 kubectl 
kubectl 就是 Kubernetes API 的封装的一个客户端命令行，用于控制和操作整个 K8s 集群，类似于 docker 中的 docker 命令。

## 3. K8s 资源的理解
K8s 组件是支持 K8s 平台运行的软件，是系统运行的进程，资源是通过组件去创建和管理的。跟 Linux 中一切皆文件, 在 K8S 中也有一切皆资源的概念。Resource 是 K8s 的一个基础概念，k8s 用 Resource 来表示集群中的各个资源。比如 Pod、Service、Deployment 等等都属于 K8s 的资源，尽管这个资源看起来差别很大，但它们都有许多共同的属性，如 name(名称)、kind(类型)、apiVersion(api版本)、metadata(元信息等)。

查看 K8s 资源有哪些资源使用如下的命令:
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
Namespace 的作用主要是用于名称隔离，可比如在不同的命名空间下 Pod 的名称是一样的，只需要保证资源名称在一个命名空间内保持唯一即可。并不是所有的资源都使用命名空间隔离的，具体可以使用命令 `kubectl api-resource` 查看。

## 4. Pod
Pod 是 k8s 的最小调度单位，Pod 包含一个或多个 Container(容器) ，Pod 中运行的容器是共享网络的，使用的网络模式是 Docker 的 Container 模式，K8s 创建 Pod 时候会先创建一个空的基础容器，然后在使用 Container 模式创建 Pod 中定义的容器，让它们共享基础容器的网络命名空间，所以容器在 Pod 中是共享网络，可以使用 localhost 就相互访问。
### 4.1 Pod 定义 
