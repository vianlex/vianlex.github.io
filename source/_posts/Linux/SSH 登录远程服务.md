---
title: SSH 各种用法记录
---
## 跳板机
![跳板机说明例图-01](/images/SSH跳板机服务器-01.png)
如上图本地电脑，能直接访问服务器 A，无法访问服务器 B，本地电脑要想登录远程和操作服务器 B，则需要使用服务器A作为中间的跳板。
1、通过中间跳板机登录远程服务器
```
ssh -J username@192.168.1.2:22 username@192.168.1.3 -p 22
```
如果 ssh 的端口时默认的 22 端口可以省略掉简写成：
```
ssh -J username@192.168.1.2 username@192.168.1.3 
```
2、通过中间跳板机上传文件
```
scp -P 22 -o 'ProxyJump username@192.168.1.2 -p 22' uploadFile.txt username@192.168.1.2:/home/username

```
## 本地端口转发

```

ssh -N [-L|-R]  <local port>:<remote host>:<remote port> <SSH server host>
-L 表示本地端口转发
-R 表示远程端口转发
-N 表示不执行命令,只进行端口转发
-f 表示执行命令前转入后台运行
-p 指定中间服务器的端口

将连接到本地电脑 61001 端口的消息通过中间服务器 A 转发到服务器 B 的22端口
ssh -p 22 -N -L 61001:39.98.110.32:22  sysadmin@10.1.20.107
ssh username@127.0.0.1 -p 61001


```



## 参考链接：
https://vpsmore.com/ssh-scp-over-jump-server.html
https://www.icode9.com/content-4-940210.html