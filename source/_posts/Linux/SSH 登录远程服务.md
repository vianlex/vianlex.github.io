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

## 参考链接：https://vpsmore.com/ssh-scp-over-jump-server.html
