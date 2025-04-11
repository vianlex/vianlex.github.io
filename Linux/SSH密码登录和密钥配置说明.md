# SSH 密码登录和密钥配置说明


## 账号密码登录

```bash

ssh username@host -p 22

```

## 密钥登录

```bash

# -i 参数指定登录的 rsa 密钥（注意：服务器必须先配置有公钥才能使用密钥登录）
# -vvv 参数表示使用 debug 模式登录，会输出登录信息
ssh username@host -p 22 -i ~/.ssh/vianlex_rsa -vvv

```

## 密钥登录配置

1、运行如下命令生成密钥，如果已有密钥则不必重新生成，默认会在用户目录，生成 .ssh 目录和 id_rsa 私钥 以及 id_rsa.pub 公钥

```bash

ssh-keygen -t rsa -C "vianlex@email.com" -f ~/.ssh/vianlex_rsa

-t 指定密钥类型，一般类型有 dsa和 rsa 默认是 rsa 类型，可以省略。
-C 设置注释，一般填写的都是自己的邮箱
-f 指定密钥文件存储文件名
-P 密钥的密码，一般都是不指定密码的，-P '' 指定密码为空字符时，运行命令可以少敲两个回车 
-b 指定密钥的位数，可以设置值如 1024 或 4069，未研究位数的作用

```

2、将公钥复制到远程服务器，复制的方式有两种

第一种方式，使用命令 ssh-copy-id

```bash

# 将公钥复制到程服务器 ~/.ssh/authorizeys 文件中
# -i 参数指定公钥文件，不指定的话，ssh-copy-id 命令默认读取用户目录下 .ssh 文件夹中的 id_rsp.pub 文件
ssh-copy-id -i ~/.ssh/id_rsp.pub  username@host -p 22

```

第二种方式，将公钥 id_rsa.pub 文件内容追加到远程服务器 ~/.ssh/authorized_keys 文件中，如果 .ssh 目录和 authorized_keys 文件不存在则需要创建后，再把 id_rsa.pub 内容复制到 authorized_keys 中。 

```bash

# 上传 id_rsa.pub 文件到远程服务，可以使用 sftp 工具上传，也可以使用 scp 上传，或者其他方式上传，例子使用 scp 命令上传

scp ~/.ssh/id_rsa.pub username@host:/home/username   

# 密码登录远程服务将刚才上传的公钥内容追加到 authorizeys 文件中，注意要使用 >> 追加，使用 > 会覆盖 authorized_keys 文件中的其他公钥

cat ~/id_rsa.pub >> ~/.ssh/authorized_keys

# 注意服务器 authorized_keys 必须要有执行权限，如果没有的话无法登录，权限修改命令如下

chmod -R 700 ~/.ssh

```

3、按上面的说明配置好后，即可直接使用命令 ` ssh username@host -i ~/.ssh/id_rsa ` 免密登录远程服务

4、禁止密码登录，限制只能使用密钥登录，需要修改 ` sudo vi /etc/ssh/sshd_config `文件如下

```bash
# 修改登录端口 port 默认是 22, 修改端口前查看是否被占用 netstat -ano | grep 2222
port 2222
# 允许 root 用户通过ssh登录
PermitRootLogin yes
# 禁用密码登录（阿里云服务器重置 Root 密码时，会自动修改 yes，开启密码登录）
PasswordAuthentication no
# 允许使用ssh权限登录
RSAAuthentication yes
PubkeyAuthentication yes

```

注意修改配置文件要重启 sshd 才能生效，重启命令,如下

```bash

# 第一种方式
sudo service sshd restart

# 第二种方式
sudo systemctl restart sshd.service

```

5、ssh 快捷登录说明，使用 ~/.ssh/config 文件实现

```bash

Host server01
HostName xx.xx.xx.xx
Port 22
User username
IdentityFile  /path/id_rsa # 指定登录的密钥，不指定默认使用的是 ~/.ssh/id_rsa 注意 ~ 符号表示的用户目录

Host server02
HostName xx.xx.xx.xx
Port 22
User username

```

以上文件配置后，直接使用 ssh server01 和 ssh server02 就能登录到服务器，注意如果没配置密钥连接方式的话，运行 ssh server01 还是会提示输入密码

## 服务器查看 ssh 的登录日志

```bash

tail -f -n 200  /var/log/secure

```



