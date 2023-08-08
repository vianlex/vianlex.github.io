---
title: SSH 端口转发用法说明
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
SSH 端口转发的命令格式如下：

```
ssh -N -L [localport]:[targethost]:[targetport] [username]@[ssh-tunnel-ip] -p ssh-tunnel-port

-L 表示本地运行的 ssh 命令监听到，访问本地端口 localport 的信息时，通过中间跳板机 ssh-tunnel 将信息转发到目标主机 targetprot 的目标端口 targetport 上  
-N 表示 SSH 连接只进行端口转发，不登录远程 Shell
-f 表示 SSH 连接成功后，该命令就后台运行
-p 用于指定执行转发操作的 SSH 通道服务器(中间服务器)端口

```
例子1，如下图, 本地电脑能直接访问服务器 A 不直接访问服务器 B 和 C，服务器 A 能直接访问服务器 B，而不能直接访问服务器 C

![SSH端口转发例图-01](/images/SSH端口转发例图-01.png)

本地电脑想要远程登录服务器 C，则需要使用服务器 B 作为中间服务器器，将服务器 A 本端口转发服务器 C，然后本地电脑访问服务器 A 设置的转发端口，就能访问服务器 C，则要执行如下命令：

```
# 在服务器 B 中执行命令
ssh -N -L 6011:192.168.1.4:22 username@192.168.1.3 -p 22
# 本地电脑执行名
ssh username@192.168.1.2 -p 6011

```
例子2、如下图, 本地电脑只能访问到服务器 A 如果想访问 MySql、服务器 B、HTTP 服务器、Email 服务器，则必须通过服务器 A 作为中间转发通道可以实现访问。

![SSH端口转发例子-02](/images/SSH端口转发例子-02.png)

想要访问服务器 B，要执行的命令如下：

```
# 本地电脑运行如下命令，通过中间服务器 A(192.168.1.1) 作为转发通道，将本地电脑的端口 3111 转发到服务器 B(192.168.1.3) 的22端口
ssh -N -L 3111:192.168.1.3:22 username@192.168.1.1 -p 22
# 本地电脑再运行命令，即可访问到服务器 B，username 服务器 B 的 ssh 用户名
ssh username@127.0.0.1 -p 3111   

```
想要访问 Myql 服务器，本地电脑也可以使用服务器 A 作为中间跳板机，端口转发，然后访问到 MySql

```
# ssh 转发命令运行后，监听本地端口 3336 如果收 3306 端连接的信息，则通过服务器 A 转发到 mysql 服务器的 3306 端口
ssh -N -L 3336:192.168.1.2:3306 username@192.168.1.1 -p 22
# 运行连接命令
mysql -u username -h 127.0.0.1 -p 3336

```
想要访问 Http 服务器和访问 Email 服务也可以通过本地端口转发来实现

```
# 本地电脑先运行转发命令
ssh -N L 880:192.168.1.2:80 -L 225:192.168.1.2:25 username@192.168.1.1 -p 22
# 浏览器输入如下连接即可访问 Http 和 Email 服务
http://127.0.0.1:880
http://127.0.0.1:225

```

## 远程端口转发
远程转发指的是在远程 SSH 服务器建立的转发规则，访问远程服务器会将指定端口的流量转发到本地服务，远程转发的命令如下：

```
# 将访问 remotehost 的 remote-port 端口的流量转发到 target-host 的 target-port 端口
ssh -R remote-port:target-host:target-port -N remotehost

```
![远程端口转发例图](/images/远程端口转发例图.png)

如上图，内网本地电脑无法连接外网，但是服务器 A 可以访问到本地电脑，外部网络想要访问到本地电脑，则可以通过远程端口转发的形式访问，运行如下命令：

```
# 运行该命令后，只要访问 192.168.1.1 的 8080 端口就会将流量转发到 127.0.0.1 的 8080 端口
ssh -N -R 8080:127.0.0.1:8081 192.168.1.1 

```

## 动态端口转发
动态转发指的是通过 SSH 监听本地的某个端口，将监听到流量数据转发通过 SSH 通道(即中间服务器)转发出去，动态端口转发的命令格式如下：

```
ssh -D local-port tunnel-host -N 

```


## 参考链接：
1、https://vpsmore.com/ssh-scp-over-jump-server.html
2、https://www.icode9.com/content-4-940210.html
3、https://www.cainiaojc.com/ssh/ssh-port-forwarding.html
4、https://solitum.net/posts/an-illustrated-guide-to-ssh-tunnels/#remote-tunnels