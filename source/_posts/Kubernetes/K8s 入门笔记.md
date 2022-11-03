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
kubectl 是与 kubernetes 集群交互的一个命令行工具，kubectl 通过与 kubernets api server 提供 Rest API 来操作控制 K8s 集群，类似于 docker 中的 docker 命令。kubectl 在执行命令的时候可以加上一个 -v=9 参数（注意参数值越大日志越详细）查看执行的具体步骤：
- 读取 kubectl 的配置文件，判断需要与哪个 api server 交互
- 生成 request body
- 调用 API
- 输出结果
#### 2.3.1 kubectl 的语法
kubectl 的命令行语法格式 ` kubectl [command] [resource-type] [name] [flags]`
- command 指定对资源执行的操作，如：create、apply、get、describe、delete 等
- resource-type 指定要操作的资源类型，使用资源的单数或者复数、简写都可以，如 pods 或 pod 或 po, nodes 或 node 或 no, 等等
- name 根据名称指定要操作的具体资源
- flags 

## 3. K8s resouce(资源)的理解
K8s 组件是支持 K8s 平台运行的软件，是系统运行的进程，资源是通过组件去创建和管理的。跟 Linux 中一切皆文件, 在 K8S 中也有一切皆资源的概念。Resource 是 K8s 的一个基础概念，k8s 用 Resource 来表示集群中的各个资源。比如 Pod、Service、Deployment、namespace 等等都属于 K8s 的资源，尽管这个资源看起来差别很大，但它们都有许多共同的属性，如 name(名称)、kind(类型)、apiVersion(api版本)、metadata(元信息等)。

### 3.1 查看 resource(资源)的命令
```bash
kubectl api-resource 
```
### 3.2 resource(资源)文件定义
k8s 资源支持定义的属性很多，定义资源文件时，未定义的属性 k8s 会设置有默认值，k8s 支持通过 YAML 和 JSON 格式文件定义资源对象，使用命令 `kebectl explain [ resource | resource.attribe ]` 资源能定义哪些属性，使用 YAML 定义资源的格式如下：
```yaml
# 指定资源的版本，命令 kubectl explain resource 可以查看资源使用的版本，如查看 pod 资源的版本，运行命令： kubectl explain pod
apiVersion: v1  
# 指定资源的类型，pod、namespace, service 等等
kind: pod
# metadata 可以定义的资源的名称等，可以使用命令 kubectl explain resource.metadata 查看支持哪些属性的定义  
metadata:   
  # 指定资源的名称
  name: test-nginx
# 定义资源的特定属性
spec: 
  # 使用命令 kubectl explain resource.
  containers:
    # 指定容器的名称
    - name: nginx-container 
      # 指定容器运行的镜像
      image: nginx
      ports:
        # hostport(主机端口) 和 containerPort(容器端口) 相当于 docker run -p 指定的端口
        - containerPort: 80
          hostport: 80  
```
快速生成资源文件
```bash
# 不存在的资源，需要指定 --dry-run 表示不是正在的创建资源，只是试运行，为了导出资源文件 
kubectl create deployment web --image=nginx-o yaml --dry-run
# 根据存在的资源导出资源文件
kubectl get pods [pod-name] -o yaml
```

### 3.3 资源的命名空间
命名空间为集群中的资源名称赋予作用域，主要作用是资源名称隔离，在命名空间中资源名称必须是唯一的，但是并不是所有的资源都使用命名空间隔离的名称，具体可以使用命令 `kubectl api-resource` 查看。
创建命名空间使用以下命令：
```bash
# 第一种方式创建命名空间，查看创建日志，在命令后面加参数 -v = 9 其中 9 表示日志的级别，一般数字越大信息越详细
kubectl create [ns | namespaces]  空间名称  
# 第二种方式，第一中方式其实时第二种方式的简化处理
kubectl create -f /xx/xx/xx.yaml

# 查看命名空间信息，可选参数 -o 指定查看信息的详细程度和输出格式
kubectl get [ns | namespaces] 空间名称 -o [wide | ymal | json ] 
# 删除命名空间
kubectl delete [ns | namespaces]  空间名称
```
### 3.4 操作 resouce(资源)的常用命令
1. kubectl create 命令，用于创建 k8s 资源，常用可选参数：
- f 指定创建资源使用的资源文件或者流
```bash
kubectl create -f resource-file.yaml 或 resource-file.josn 或 http://xxx/xx/xx.yaml
```
2. kubectl apply 命令，用于更新资源，不存在则创建新的资源，常用的可选参数
- -f 指定创建资源使用的资源文件或者文件流
- -k 指定创建资源使用某个目录下的资源文件
```bash
# 根据指定的资源文件更新或者创建资源
kubectl apply -f  resource-file.yml 或 resouce-file.json 或 https://xxx/xxx/resource-file.yml
# 根据资源文件的标准输入流创建或者更新资源
cat resource-file.yml |  kubectl apply -f - 
# 指定读取目录下的资源文件
kubectl apply -k  /home/resource-dir
```
3. kubectl get 命令查看资源信息，常用参数
- -A 指定查看全部命名空间的资源
- -n 指定查看某个命名空间的资源，如 `-n kube-system `
- -o, --output 指定结果的输出格式，常用的可选值有 wide、yaml、json、name(只显示名称) 等等
注意查看资源时，如果时需要命名空间隔离的资源，需要 `-n ` 参数指定命名空间或者 ` -A ` 指定查看全部命名空间的资源，如果指定命名空间，默认查看的是 default 命名空间的资源，可以使用命令 ` kubectl api-resources ` 查看资源是否需要命名空间隔离。
```bash
# 查看 k8s 的所有 node 节点
kubectl get nodes 
# 根据名字查看某个 node 节点
kubectl get node [node-name] 
# 查看 pod 资源
kubectl get pods # 不指定命名空间，查看的 default 命名空间的所有 pod 
# 查看所有命名空间下的所有 pod
kubectl get pods -A
# 查看某个命名空间的所有 pod
kubectl get pods kube-system
# 根据名称查看某个 pod 
kubectl get pod [pod-name] -o wide | yaml | josn 
```


## 4. Pod
Pod 是 k8s 的最小调度单位，Pod 包含一个或多个 Container(容器) ，K8s 创建 Pod 时，先默认创建运行一个 init 容器，然后在运行资源文件中定义的应用容器，应用容器使用的网络模式是 Docker 的 Container 模式，共享 init 容器的网络命名空间，所以 Pod 中运行的容器是共享网络的，所以使用 localhost 加端口就可以相互访问。注意 init 容器也是可以在资源文件中自定义，具体可以查看官方文档。
kubernetes 本身的组件也可以通过容器化的方式运行在集群中，并且都存在于 kube-system 命名空间下。

### 4.1 Pod 定义 
使用` kubectl explain pod ` 查看 pod 资源可以定义的属性和支持的版本
```yaml
apiVersion: v1
kind: Pod # 注意要大写
metadata:
  name: test-nginx
spec:
    # 指定容器的名称
  - name: nginx-container 
    # 指定容器运行的镜像
    image: nginx
    # 指定镜像拉取的策略
    imagePullPolicy: IfNotPresent
    ports:
        # 容器 expose 的端口
      - containerPort: 80
        # 宿主机的端口，相当于 docker run -p 8000:80
        hostPort: 8000 
```
### 4.2 操作 Pod 的常用命令
1. 创建和更新 pod
```bash
# 创建 pod，注意如果 yaml 文件或者不使用 -n 指定命名空间，则 pod 会创建在默认 default 命名空间下，是使用命令 kubectl explain pod.metadata 可以查看
kubectl create pod -f xxx.yaml
# 创建或者更新 pod 
kubectl apply pod -f xxx.yaml
```
2. 删除 pod
```bash
# 根据名称删除 pod，如果不指定命名空间，则删除 default 命名空间下的 pod 
kubectl delete pod [pod-name] [-n 命名空间]
```
3. 查看 pod
第一种方式：使用 kubectl get pod [pod-name] [-o wide | yaml | json ] [-n 命名空间 | -A ]
```bash
# 不指定命名空间，默认查看 default 命名空间下的 pods
kubectl get pod -o wide 
# 查看所有命名空间的 pods
kubectl get pod -A 
# 根据名字查看指定的 pod 不指定命名空间，默认查看 default 命名空间的 pod 
kubectl get pod [pod-name]
```
第二种方式：kubectl describe pod [po-name] [-n 命名空间 | -A]，使用第二种方式，可以查看 pod 的 events 信息
```bash
# 查看 default 命名空间的下的所有 pod
kubectl describe pod 
# 根据名称查看 pod 信息，注意如果不指定命名空间，则默认查看 default 命名空间下的 pod 
kubectl describe pod [pod-name]
```







## 参考连接
1. https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/