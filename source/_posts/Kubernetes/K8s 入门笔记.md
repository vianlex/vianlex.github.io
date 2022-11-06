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
K8s 组件是支持 K8s 平台运行的软件，是系统运行的进程，资源是通过组件去创建和管理的。跟 Linux 中一切皆文件, 在 K8S 中也有一切皆资源的概念。Resource 是 K8s 的一个基础概念，k8s 用 Resource 来表示集群中的各个资源。d都可以在资源文件中配置。比如 Pod、Service、Deployment、namespace 等等都属于 K8s 的资源，尽管这个资源看起来差别很大，但它们都有许多共同的属性，如 name(名称)、kind(类型)、apiVersion(api版本)、metadata(元信息等)。

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
  # 指定资源的命名空间，指定的话，默认是 default
  namespace: default
  # 指定资源的版本
  resourceVersion: "11012"
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
# 或者
kubectl run Pod test-nginx --image=nginx --dry-run=client -o yaml
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
需要命名空间隔离的资源，使用 kubectl 操作时，都是需要指定命名空间的，如果不指定命名空间，则默认操作的是 default 命名空间下的资源。使用命令 `kubectl api-resources ` 查看哪些资源是需要命名空间隔离的。

1. kubectl create 命令，用于创建 k8s 资源，常用可选参数：
- f 指定创建资源使用的资源文件或者流
```bash
# 注意需要命名空间隔离的资源，如果没有指定命名空间的会默认将资源创建在 default 命令空间中
kubectl create -f resource-file.yaml 或 resource-file.josn 或 http://xxx/xx/xx.yaml
```
2. kubectl apply 命令，用于更新资源，不存在则创建新的资源，常用的可选参数
- -f 指定创建资源使用的资源文件或者文件流
- -k 指定使用某个目录下的 kustomization.yaml 文件创建或者更新资源
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
- --show-lables 查看资源的 label 
- -w, --watch=<true|false> 实时显示资源信息，相当于 tail 中的 -f 
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
4. kubectl describe 命令展示资源详情信息
```bash
# 查看 Pod 的详情信息和 Pod 容器事件 Events
kubectl describe pod [pod-name] [-n 命名空间]
# 查看 node 节点的详细信息和节点的内存和 cpu 信息，以及 Pod 限制 cpu 和内存信息
kubectl describe node [node-name]
```
5. kubectl label 命令，用于新增、更新、删除资源标签，资源的每个标签都是 `key=value` 的形式，注意在 yaml 中定义资源的标签时，使用的是` key : value ` 形式
命令的语法格式 `kubectl lable <resource-type : resource-file > <resouece-name>... <key=label>... [--resource-version=version]` 其中 ... 表示可以指定多个然后用空格隔开 
```bash
# 查看 node 节点标签
kubectl get nodes --show-labels
# 给名为 k8s-node01 的 node 节点打上 hello=world 标签
kubectl label node k8s-node  hello=world
# --overwrite 表示打标签时，如果存在标签 key 相同的标签，则覆盖更新标签
kubectl label node k8s-node --overwrite hello=node-world
# 给所有node 节点都打上 test-k8s-node = true 标签
kubectl label node --all  test-k8s-node = true 
# 给名为 k8s-node01 和 k8s-node02 的节点都打上 hello=world 和 whoiam=node 标签
kubectl label node k8s-node01 k8s-node02 hello=world whoiam=node
# 删除资源的标签，使用 key和减号，如删除所有节点中标签 key 等于 hello 的标签
kubectl lable node --all hello- 

# 查看 pods 的标签
kubectl get pods --show--label
# 为名 test-web 的 pod 打上 nginx-pod=true 标签
kubectl lable pods test-web nginx-pod=true
# 根据文件资源文件中定义的 kind、metadata.namespace、metadata.name 找到对应的 pod 打上 hello=world 标签
kubectl label pod -f xxx.yaml hello=world
```
6. kubectl exec 命令，在宿主机执行 Pod 中容器的命令，跟 docker exec 类似
命令语法格式：` kubectl exec <pod-name> [-n 命名空间] [ -c Pod 中容器名称 ] [ -it ]  -- <container-command>`
- -n 指定命名空间，不指定的话，默认是 default 命名空间下的 Pod
- pod-name 指定 Pod 的名称
- -c 指定运行 Pod 中的哪个容器的命令，如果不指定默认随机运行一个
- container-command 指定容器要运行的命令，如 bash、ls、cat、date 等等想要运行的命令

```bash
# 进入 default 命名空间下 test-web 内的 web01 容器中
kubectl exec  test-web -c web01 -it -- /bin/sh 
```

## 4. Pod
Pod 是 k8s 的最小调度单位，Pod 包含一个或多个 Container(容器) ，K8s 创建 Pod 时，Pod 内容器默认使用的 Container 网络模式，所以在运行 Pod 中定义的容器之前，会先默认创建运行一个 init 容器，其他容器使用 init 容器的网络命名空间, 从而实现 Pod 内容器共享网络。即 Pod 内容器使用 localhost 就可以相互访问。注意 init 容器也是可以在资源文件中自定义，具体可以查看官方文档。
kubernetes 本身的组件也可以通过容器化的方式运行在集群中，并且都存在于 kube-system 命名空间下。注意 k8s 支持通过资源文件或者运行 kubectl run 创建 Pod。 

### 4.1 定义 Pod 资源文件
使用` kubectl explain pod ` 查看 pod 资源可以定义的属性和支持的版本
```yaml
apiVersion: v1
# 注意要大写
kind: Pod 
metadata:
  name: test-web
spec:
  # 定义 Pod 中的容器，可以多个
  containers:
      # 指定容器的名称, 注意 docker ps 查看容器的时候，看到的名字实际为： k8s_<pod容器名>_<pod名>_<命名空间>_<uid>_<Priority>
    - name: web01
      # 指定容器运行的镜像
      image: nginx
      # 指定镜像拉取的策略，可选值分别为，Always(总是从远程仓库拉取)、IfNotPresent(本地没有镜像时，才会从远程仓库拉取)、Never(只使用本地镜像，没有就报错) 默认是 IfNotPresent
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
# 根据资源文件中的 kind、metadata.namespace、metadata.name 确定要删除的资源
kubectl delete -f xxx.yaml
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
第二种方式：kubectl describe pod [po-name] [-n 命名空间 | -A]，使用第二种方式，可以查看 pod 详细信息和 events 事件
```bash
# 查看 default 命名空间的下的所有 pod
kubectl describe pod 
# 根据名称查看 pod 信息，注意如果不指定命名空间，则默认查看 default 命名空间下的 pod 
kubectl describe pod [pod-name]
```
4. pod 端口映射到宿主机端口
命令格式 ` kubectl port-forward [pod-name] [hostPort:containerPort] `, 注意要 pod 所在的 node 节点运行才行
```bash
# 将 pod 端口映射到宿主机的端口，注意要在 pod 在的 node 节点运行才行
kubectl port-forward test-web 3000:80
```
5. 进入 Pod 内的容器
```bash
kubectl exec test-web -n default -c web01 -it -- bash
# 查看环境变量
kubectl exec test-web -n default -c web01 env 
```
6. 查看 Pod 内的容器日志
```
kubectl logs test-web -n default -c web01 -f --tail=200
```
7. kubectl run 命令创建 Pod
命令语法格式: `kubectl run <pod-name> --image=image [--dry-run=server|client] ` 具体查看帮助文档
```bash
# 通过 --dry-run 参数，指定显示 Pod 的创建资源文件信息，不实际运行 Pod
kubectl run test-web  --image=nginx --dry-run=client -o yaml  
```
### 4.3 Pod 容器数据持久化，即容器挂载目录到宿主机，防止数据丢失
第一种方式：通过 volumes 和 volumeMounts 定义挂载的目录，如下
```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: test-web
spec:
  # 指定将 Pod 资源创建在标签为 hello=world 的 node 主机节点中，注意如果多个 node 节点都存在 hello=world 标签, k8s 会调度选定一个节点，
  # 如果固定选择某个 node 节点，需要标签唯一
  nodeSelector:
    #注意在，在 yaml 中定义或者使用 label 时用 : 代替 = 
    hello: world 
  volumes:
    - name: nginx-data
      hostPath: # 定义挂载的宿主机目录
        # 注意该目录是 node 节点主机的目录，Pod 创建时调度到不确定的节点，最好使用 nfs、ceph、glusterfs 文件共享存储目录,
        # 或者通过 nodeSelector 指定 pod 部署的节点，才能保证，Pod 删除后重新创建时，挂载目录还是固定的主机目录 
        path: /opt/data 
        # 指向一个目录，不存在时自动创建
        type: DirectoryOrCreate     
  containers:
    - name: test-nginx # 容器名字
      image: nginx # 镜像
      volumeMounts:
        - name: nginx-data # 注意 name 要跟上面 hostPath 的名字一样，因为是根据名字取匹配的
          mountPath: /usr/share/nginx/html  # 指定容器的挂载目录
```

### 4.4 Pod 重启策略
Pod 重启策略(RestartPolicy)，定义容器的重启规则，当 Pod 内某个容器异常退出或者探针健康检测失败时，kubelet 会根据重启策略(RestartPolicy)来进行相应的操作。Pod 的重启策略有三种分别是 Always、OnFailure、Never，默认值是 Always。
- Always 当容器进程退出时，kubelet 总是会自动重启容器
- OnFailure 当容器终止运行且退出码不为0时，kubelet 会自动重启容器
- Never 当前容器运行状态如何，kubelet 都不会自动启动容器 

```bash
apiVersion: v1
kind: Pod 
metadata:
  name: test-web
  namespace: default
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      # 命令正常执行成功返回的 0，设置退出码为 1 测试 OnFailure 重启策略
      args: 
        - /bin/sh
        - c 
        - sleep 10 && exit 1

```

### 4.5 容器 Probe(探针)
probe(探针) 是由 kubelet 对容器定期执行的健康诊断。 诊断的方式是 kubelet 在容器内执行代码，或者发送一个网络请求。如果没有在资源文件中定义 prode 则 kubelet 会默认所有的诊断结果都是健康的。
#### 4.5.1 探针检测的方式
- exec 在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
- gRPC 如果容器中实现 gRPC健康检查，可以使用 gRPC 执行一个远程过程调用。如果响应的状态是 SERVING，则认为诊断成功。
- httpGet 对容器服务发起 HTTP GET 请求。如果响应的状态码大于等于 200 且小于 400，则任务诊断成功。
- tcpSocket 对容器的指定端口执行 TCP 检查。如果端口打开，则诊断被认为成功

#### 4.5.2 探测结果
每次探测都将获得以下三种结果之一：
- Success（成功） 容器通过了诊断。
- Failure（失败）容器未通过诊断。
- Unknown（未知） 诊断失败，不会采取任何行动。

#### 4.5.3 探针的类型
k8s 提供了三种容器探针，分别如下：
- startupProbe 启动探针，用于诊断容器中的服务是否已经启动成功，如果探针检测返回失败结果，则 kubelet 会杀死容器，并且根据容器重启策略(RestartPolicy)决定是否重启。注意该探针如果诊断成功之前不会运行 livenessProbe 和 readinessProbe 探针，未资源文件中配置 startupProbe 探针，则 kubelet 默认是诊断成功的。
- livenessProbe 存活探针，用于诊断容器是否已经挂掉了，如果探针检测返回失败结果，则 kubelet 会杀死容器，并且根据容器重启策略(RestartPolicy)决定是否重启。
- readinessProbe 就绪(READY)探针，用于诊断容器中的服务是否能正常接收请求，如果探针检测返回失败结果，Endpoint 控制器将 Pod 从 Endpoint 对应的服务中移除，不会将任何请求发送该 Pod 上，直到探针检测返回成功结果为止。
1. livenessProbe 存活探针例子，使用 `kubectl describe pod <pod-name>`命令查看检测是否成功，如果失败会有事件提示。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-web
spec:
  containers:
    - name: web01
      image: nginx
      ports:
        - containerPort: 80
      livenessProbe:
        # kubectl 每3秒请求一次 /index 如果请求返回的状态码范围是 200 <= statuCode < 400 说检测成功
        httpGet:
          path: /index
          port: 80
        initialDelaySeconds: 5 # 表示容器运行5秒后，kubelet 开始检测
        periodSeconds: 3 # 表示每隔 3 秒 kubelet 检测一次
        successThreshold: 3 # 表示3次检测成功才算是检测成功，主要防止出现误差，默认值是 1
        failureThreshold: 3 # 表示3次检测失败才算是检测失败，主要防止出现误差，默认值是 1
```

### 4.6 Pod 容器资源限制
Pod 内容器运行依赖的硬件资源，最重要的指标就是 cpu 和内存，k8s 提供 requests 和 limits 两种参数类型来分配和限制容器使用的资源。它们的区别如下：
- requests 限制的资源只是作为 k8s 创建 Pod 时调度到指定节点的判断依据，不限制容器运行资源，只有主机节点的可分配资源大于等于 requests 限制的资源时，才允许 Pod 调度到此节点。
- limits 限制 Pod 内容器能使用的最大资源，如果容器使用的内存超出设置值，则会报 OOM。如果内存和 cpu 的值都设置为 0 则 表示资源不作限制。
```yaml
...
sepc:
  containers:
    - name: web01
      image: nginx
      resource:
        # 使用命令 kubectl describe node [node-name] 能查看主机节点的内存和 cpu 
        requests:
          memory: 200Mi
          cpu: 50m
        limits:
          memory: 500Mi
          cpu: 100m 
```
### 4.7 Pod 状态和生命周期
#### 4.7.1 生命周期
```mermaid
gantt
  title Pod 生命周期
  dateFormat ss
  axisFormat %Ss
  pod start : milestone, s1, 00, 0s
  section Init Container
  # 内容，颜色、标签、起始点，
  init Container-01 : i1, 00, 3s
  init Container-02 : i1, 01, 3s
  init Container-03 : i1, 02, 3s
  section Main Container
  post start hook:  m1, 05, 2s
  main container: m1, 05, 6s
  livenessProbe: m1, 07, 2s
  readinessProbe: m1, 07, 2s
  pre stop hook: m1, 09, 2s
  section  
  pod stop : milestone, crit, s1, 11, 0s
```
Pod 资源对象从创建到结束的时间段称为 Pod 的生命周期，Pod 周期过程和钩子函数说明如下：
1. Pod 创建过程
2. 运行初始化容器(init container)过程
3. 运行主容器(main container)过程
    - 容器启动后执行的钩子函数（post start），容器终止前执行的钩子函数（ pre stop）
    - 容器的存活性探测(liveness probe)、就绪性探测(readiness probe)
4. Pod 终止过程

```yaml



```

#### 4.7.2 Pod 状态和容器状态
1. Pod 生命周期中各种状态说明
- Pending（等待中）	Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。
- Running（运行中）	Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。
- Succeeded（成功）	Pod 中的所有容器都已成功终止，并且不会再重启。
- Failed（失败）	Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。
- Unknown（未知）	因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。

2. Pod 内容器生命周期状态
- Waiting （等待） 容器运行前的等待状态，如拉取容器镜像，或者向容器应用 Secret 数据等等。
- Running（运行中） 表明容器正在执行状态并且没有问题发生。
- Terminated（已终止） 容器已经开始执行并且或者正常结束或者因为某些原因失败。

### 4.8 静态 Pod 
静态 Pod 指的是 kubelet 自动创建的 Pod，不需要我们使用` kubectl <create|apply> `手动创建。静态 Pod 的 yaml 是存放在 ` /etc/kubernetes/manifests/ ` 中的，kubectl 会自动扫描该目录的 yaml 文件并自动创建，创建的 Pod 即为静态 Pod。如果我们想创建静态 Pod 直接将 yaml 放到该目录即可，kubectl 会自动创建。


## 5. ConfigMap 和 Secret
k8s 提供 ConfigMap 和 Secret 两种不同类型的资源，来实现业务配置信息的统一管理。在静态 Pod 中无法使用配置资源。
1. ConfigMap 用于管理不敏感的配置项。ConfigMap 资源的定义如下：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-web-config
  # 不指定命名空间的话，资源默认在 default 命名空间下
  namespace: default
# 定义应用中使用的配置项 
data:
  # 配置项的值要加引号，不然创建资源时会报错
  redis_host: "127.0.0.1"
  redis_port: "6739"

```
2. Secret 用于管理中的配置信息，如账号密码，等等敏感配置信息。Secret 资源的定义如下：
```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: test-web-secret
  # 不指定命名空间的话，资源默认在 default 命名空间下
  namespace: default 
# type 指定 data 能定义哪些配置项，Opaque 表示用户可以定义的任意数据，当类型为 kubernetes.io/service-account-token 时 data 中只能配置 k8s 服务账号令牌
# 具体查看： https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#secret-types
type: Opaque
# 定义应用使用的配置项
data:
  # 与configMap的区别是。配置项的值需要 base64 编码
  db_username: "YWRtaW4K"
  db_password: "MTIzNDU2Cg=="
```
3. 创建和查看 configMap 和 Secret 跟 Pod 一样操作即可
```bash
# 创建资源
kubectl create -f test-web-config.yaml
# 查看资源, 不指定命名空间，默认查看的是 default 命名空间的
kubectl get <configmap | cm> [资源名]
或者
kubectl describe <configmap | cm> [资源名]
```
4. 通过内容为 `key=value ` 格式的文件创建 configMap 资源
```bash
# 通过 key=value 文件创建 configMap 资源
kubectl create configmap test-web-config -n default --from-env-file=configs.txt
# 通过 key=value 文件创建 secret 注意文件中的 value 是需要 base64 的
kubectl create configmap ggeneric test-web-config -n default --from-env-file=configs.txt
# configs.txt 的文件内容格式如下
hello="world"
lang="en"
```
5. Pod 内容器服务使用 ConfigMap 和 Secret 资源中的配置项，方式如下：
- 作为环境变量使用
使用命令  `kubectl exec test-web -c web01  -- env ` 可以查看容器环境变量
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-web
spec:
  containers:
    - name: web01
      image: nginx
      env:
        # 定义环境变量 db_url
        - name: db_url
          value: jdbc://mysql/xxx//xx/xx
          # 定义环境变量 db_password，变量的值从 Secret 资源中获取
        - name: db_password
          valueFrom:
              secretKeyRef:
                # Secret 的资源名
                name: test-web-secret
                # 获取 Secret 资源中 db_password 的值
                key: db_password
        - name: redis_port
          valueFrom:
            configMapKeyRef:
              name: test-web-configmap
              key: redis_port
```
- 将配置资源挂载为容器文件
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-web-02
  namespace: default
spec:
  containers:
  - name: test-nginx
    image: nginx
    volumeMounts:
    - name: secret-config
      # test-web-secret 中的 data 的每一个配置项都会在容器目录 /etc/config 中生成文件，key 作为文件名，value 是文件的内容  
      mountPath: "/etc/config"
      readOnly: true
  volumes:
  - name: secret-config
    secret:
      secretName: test-web-secret
```































## 参考连接
1. https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/