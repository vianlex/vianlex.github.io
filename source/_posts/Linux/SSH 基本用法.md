---
title: SSH 基本用法
date: 2022-05-12
---

## 账号密码登录

```
ssh username@host -p 22
```

## 公钥登录

1、运行如下命令生成密钥，如果已有密钥则不必重新生成，默认会在用户目录，生成 .ssh 目录和 id_rsa 私钥 以及 id_rsa.pub 公钥

```
ssh-keygen -t rsa -C "xxxx@email.com"

-t 指定密钥类型，一般类型有 dsa和 rsa 默认是 rsa 类型，可以省略。
-C 设置注释，一般填写的都是自己的邮箱
-f 指定密钥文件存储文件名
-P 密钥的密码，一般都是不指定密码的，-P '' 指定密码为空字符时，运行命令可以少敲两个回车 
-b 指定密钥的位数，可以设置值如 1024 或 4069，未研究位数的作用

```
2、将公钥复制到远程服务器，复制的方式有两种

第一种方式，使用命令 ssh-copy-id

```
ssh-copy-id -i ~/.ssh/id_rsp.pub  username@host

-i 参数指定公钥文件，不指定的话，ssh-copy-id 命令默认读取用户目录下 .ssh 文件夹中的 id_rsp.pub 文件

```
第二种方式，将公钥 id_rsa.pub 文件内容追加到远程服务器 ~/.ssh/authorizeys 文件中，如果 .ssh 目录和 authorizeys 文件不存在则需要创建后再把 id_rsa.pub 内容复制到其中。 

```
上传 id_rsa.pub 文件到远程服务，可以使用 sftp 工具上传，也可以使用 scp 上传，或者其他方式上传，例子使用 scp 命令上传

scp ~/.ssh/id_rsa.pub username@host:/home/username   

密码登录远程服务将刚才上传的公钥内容追加到 authorizeys 文件中

cat ~/id_rsa.pub >> ~/.ssh/authorizeys

注意远程服务的 .ssh 目录权限不是 700 要修改一下

chmod -R 700 ~ /.ssh


```


3、按上面的说明配置好后，即可直接使用命令 ssh username@host 免密登录远程服务