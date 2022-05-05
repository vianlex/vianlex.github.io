---
title: SSH 登录和上传文件到远程服务器用法
---
## SSH 通过中间服务跳板登录远程服务
![跳板机说明例图-01](/images/SSH跳板机服务器-01.png)
```
ssh -J username@192.168.1.2:22 username@192.168.1.3 -p 22
```


## SCP 通过中间服务器跳板上传文件到远程服务器
```
scp -P 22 -o 'ProxyJump username@192.168.1.2 -p 22' uploadFile.txt username@192.168.1.2:/home/username

```

## 参考链接：https://vpsmore.com/ssh-scp-over-jump-server.html
