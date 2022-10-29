---
title: kubeadm安装 k8s 问题记录
---
1. kubeadm init 提示 [WARNING FileExisting-tc]: tc not found in system path 的解决办法
```bash
# 安装 iprote-tc
dnf install iproute-tc
```
2. 提示 [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver 处理办法
```bash
# 在 /etc/docker/daemon.json 文件中加入以下内容，并重启 docker , 如果 daemon.json 文件不存在，则创建一个
{
 "exec-opts":["native.cgroupdriver=systemd"]
}
```
3. 组件 coredns 一直在 pending 中的，使用命令 `journalctl -f -u kubelet.service` 查看日志，如果有提示 Unable to update cni config: no networks found in /etc/cni/net.d 则说明是网络组件（Flannel 或者 calico 等）没有安装

4. 安装 flannel 插件是如果一直无法成功,使用 docker images 查看` flannel `和`flannel-cni-plugin`镜像是否已经下载成功
