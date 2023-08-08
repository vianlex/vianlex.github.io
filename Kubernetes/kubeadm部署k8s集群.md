# kubeadm 快速部署 kubernetes 集群测试环境

## 1. 安装要求

部署 Kubernetes 集群机器需要满足以下几个条件：

- 至少2台机器
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止swap分区

## 2. 硬件环境准备

准备至少两台机器，一台 master 节点，两台 node 节点。

| 序号 |  机器 IP     | 操作系统 | 节点类型 |
| --- | --           |  --      |    --   |
| 1   | 192.168.204.3 | CentOS8  | master  |
| 2   | 192.168.204.4 | CentOS8  |  node   |
| 2   | 192.168.204.5 | CentOS8  |  node   |

### 2.1 云服务的安全组设置

如果搭建的集群使用外网，需要在安全组中开放端口

|  节点              |  协议类型| 访问方向     |  端口  |   描述 |  
|   --               | --      | --      | --    |  --      | 
|  Master 服务器     | TCP     |	入方向	| 6443	| Kubernetes API server	的端口 |
|   Master 服务器    | TCP	    | 入方向	| 2379-2380	| etcd server client API	kube-apiserver, etcd 的端口 |
|   Master 服务器    | TCP	    | 入方向	| 10250	| Kubelet API	的端口 |
|   Master 服务器    | TCP	    | 入方向	| 10259	| kube-scheduler	的端口 |
| node 服务器        | TCP      | 入方向	| 10250 | Kubelet API	的端口 |
| node 服务器        | TCP      | 入方向	| 30000-32767 |  NodePort Services 的端口 |


## 3. CenteOs 系统环境配置（ master 和 node 机器都要改）

### 3.1 关闭防火墙 和 SELINUX(类型 Window UAC)

```bash
# 关闭防火墙, 关闭 firewalld 只使用 iptables 防火墙
systemctl stop firewalld && systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时关闭，机器重启后重新启动
```
### 3.2 关闭 Swap

当系统使用 swap 会降低性能。所以 kubelet 要求禁用 Swap

```bash
# 临时关闭
swapoff -a
# 永久关闭，执行了命令， kubeadm init 提示没有的话，重启系统试试 systemctl reboot
sed -ri 's/.*swap.*/#&/' /etc/fstab
# 或者
vim /etc/fstab 用 # 注释掉 UUID swap 分区
```
### 3.3 修改机器的主机名

在一个局域网中，每台机器都有一个主机名，用于主机与主机的区分，默认的主机名，不容易记住，需要修改成方便我们识别的

```bash
# 运行以下命令将 master 主机名改成 k8s-master
hostnamectl set-hostname k8s-master
# 修改对应 node 节点的主机名
hostnamectl set-hostname k8s-node01
hostnamectl set-hostname k8s-node02

# 在 hosts 中添加 ip 地址和主机名的映射关系，主机名相当于域名，hosts 是机器的本地 DNS 服务，通过主机访问网络时，会在 hosts 解析到对应的 ip
cat >> /etc/hosts << EOF
192.168.204.3 k8s-master
192.168.204.4 k8s-node01
192.168.204.5 k8s-node02
EOF
```
### 3.4 将桥接的 IPv4 流量传递到 iptables 的链

```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 立即生效
sysctl --system 
```
### 3.5 时间同步(可以不修改)

在集群中，时间是一个很重要的概念，要保证集群机器中的时间一致，不然可能会导致集群出现问题。

```
# 安装时间同步软件
yum install ntpdate -y
# 使用阿里云的时间服务器
ntpdate time1.aliyun.com 
# 通过定时任务，定时同步时间
*/1 * * * * /usr/sbin/ntpdate time2.aliyun.com > /dev/null 2>&1

```


## 4 安装 kubeadm、kubelet 和 kubectl

kubeadm 创建 Kubernetes 集群的最快捷的一种命令行工具，其常用的命令如下：
 - kubeadm init 初始化 Kubernetes 主节点
 - kubeadm join 用于 node 节点加入到集群中
 - kubeadm upgrade 更新 Kubernetes 集群到新版本
 - kubeadm token 管理 k8s token 令牌
 - kubeadm reset 还原 kubeadm init 或者 kubeadm join 所作的操作
 - kubeadm version 打印出 kubeadm 版本
kubectl 是操作管理 k8s 集群的命令行工具。
kubelet 是容器运行时，k8s 组件 api service 接收到 kubectl 的操作请求，通过控制 kubelet 去创建和管理容器的。 

### 4.1 添加阿里云源

```bash
# 添加 k8s 软件源
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 设置 aliyun 镜像加速库，要重启 docker
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"],
  "exec-opts":["native.cgroupdriver=systemd"]
}
EOF
```

### 4.2 master 节点安装 kubeadm、kubelet 和 kubectl

kubelet 组件用于处理 api-server 发到本节点的任务，管理 Pod 及 Pod 中的容器。每个 kubelet 进程都会在 api-server 上注册本节点自身的信息，定期向 Master 汇报节点资源的使用情况，并通过 cAdvisor 监控 容器和节点资源。

```bash
# 安装的时候需要指定版本，kubeadm init 初始化节点设置的版本相对应
yum install -y  kubeadm-1.20.15 kubectl-1.20.15 kubelet-1.20.15
# 启动 kubelet 并设置开机启动
systemctl start kubelet && systemctl enable kubelet
```

### 4.3 初始化 master 节点

```bash
kubeadm init \
  --apiserver-advertise-address=192.168.204.3 \ 
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.20.15 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16

# --apiserver-advertise-address 指定 master 节点的地址，如果是阿里云服务不能指定为外网地址会报 bind: cannot assign requested address 错误
# --kubernetes-version 指定 k8s 的版本，要和 kubectl 的版本相同，使用命令 kubectl --version 可以查看 kubectl 的版本
# --service-cidr 指定 service 资源的网络地址段
# --pod-network-cidr 指定 Pod 内部网络地址段

```
### 4.4 k8s 客户端 kubectl 工具配置

kubectl 是与 kubernetes 集群交互的一个命令行工具, kubectl 通过调用 api server 组件 Rest Api 来交互来操作集群的。Api Server 接口的认证信息和访问地址默认是存放在 /etc/kubernetes/admin.conf 的， kubectl 请求 Api 
Server 接口获取认证信息和访问地址，默认是从用户目录下的 .kube/config 文件读取或者从环境变量 KUBECONFIG 指定的文件中读取，所以要作以下配置：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 或者
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```
### 4.5 node 节点安装 kubeadm、kubelet 

```bash
yum install -y  kubeadm-1.20.15  kubelet-1.20.15
# 启动 kubelet 并设置开机启动
systemctl start kubelet && systemctl enable kubelet
```
### 4.6 Node 加入 master 所在集群

需要在 master 节点中先安装网络插件，才能访问到集群

```
# master 节点获取 token, 默认token有效期为24小时 
kubeadm token create --print-join-command
# 在 node 指定 kubeadm join 命令即可加入集群 
kubeadm join 192.168.1.1:6443 --token esce21.q6hetwm8si29qxwn \
    --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5

```

## 5. 注意 coredns 组件

如果未安装网络插件的话，coredns 组件的 pod 会一直在 pending 中，直到检测有网络插件完成成功为止。

## 6. 部署网络插件

kubernetes 需要使用第三方的网络插件来实现 kubernetes 的网络功能，第三方网络插件有多种，常用的有 flanneld、calico 和 cannel（flanneld+calico），不同的网络组件，都提供基本的网络功能，为各个 Node 节点提供 IP 网络等。
k8s 的组件可以通过容器化的方式，运行在 k8s 集群中，将 calico 网络插件创建在集群中，操作如下：

```
# 下载网络插件的资源文件，要注意下载的版本是否支持安装的 k8s 
wget https://docs.projectcalico.org/v3.20/manifests/calico.yaml
# 部署网络插件
kubectl apply -f calico.yaml
# 查看部署的网络插件，使用如下命令       
kubectl get pods -n kube-system
# 显示的结果如下，说明部署成功
NAME                   READY   STATUS    RESTARTS   AGE
calico-kube-xx-xx-xx   1/1    Running   0          72s

```

## 7. 测试kubernetes集群

在 Kubernetes 集群中创建一个 pod，验证是否正常运行：

```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```

访问地址：http://NodeIP:Port  




