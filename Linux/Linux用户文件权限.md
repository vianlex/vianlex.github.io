# Linux 用户文件权限

## 用户和用户组

### 用户管理

在 Linux 中有两种用户类型分别是超级用户（root）和普通用户。

- 超级用户（root）: root 用户拥有最大权限，不受限制可以做任何操作；
- 普通用户：普通用户受权限控制，只有授予指定权限后才能执行有权限的操作。

#### 用户配置文件

通过命令 `cat /etc/passwd` 查看用户配置文件，能查看到所有 Linux 用户

用户配置文件 `cat /etc/passwd` 文件内容每行的格式为：`username:password:UID:GID:GECOS:homedir:shell`，GECOS 可以认为是注释字段，用户存储用户的个人信息，可以是名字电话等信息（可以用逗号隔开）

```bash

root:x:0:0:root:/root:/bin/bash
alice:x:1001:1001:Alice Smith,123456789:/home/alice:/bin/bash

```

#### 创建和管理用户

```bash

# 创建用户
sudo useradd -m -s /bin/bash username     # -m 创建家目录，-s 指定 shell，注意 -m 和 -s 都是可选的
sudo adduser username                     # 交互式创建（Debian/Ubuntu）

# 设置/修改密码
sudo passwd username                     # 修改指定用户密码
passwd                                   # 修改当前用户密码

# 修改用户属性
sudo usermod -s /bin/zsh username        # 修改shell
sudo usermod -d /new/home username       # 修改家目录
sudo usermod -aG sudo username           # 添加用户到sudo组
sudo usermod -L username                 # 锁定用户
sudo usermod -U username                 # 解锁用户

# 删除用户
sudo userdel username                    # 删除用户，保留家目录
sudo userdel -r username                 # 删除用户及家目录

```

#### 查看登录用户

可以通过 `whoami` 指令查看当前登录用户是超级用户还是普通用户，输出 root 则表示为超级用户，输出其他用户名则表示为普通用户

#### 用户切换

通过 su 命令可以切换登录用户

```bash

# 从当前用户切换到 sysadmin 用户 
su sysadmin 

```

### 用户组管理

在 Linux 中每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理。

#### 用户组配置文件

通过 `cat /etc/group 或者 getent group` 命令查看用户组配置文件，可以查看用户是属于那个用户组的。

用户组配置文件 `/etc/group` 每一行的内容格式为：` groupname:password:GID:userlist`，内容示例如下：

```bash 

docker:x:991:sysadmin
developers:x:1002:alice,charlie

```

Linux 系统部分默认组说明：

- root 组：表示用户是系统管理员
- wheel 组：表示用户有 sudo 权限，执行 sudo 命令时，通过 `/etc/sudoers` 的配置判断是否不需要输入密码，注意默认情况 wheel 组的 sudo 权限可能未打开，编辑 `/etc/sudoers` 文件打开即可
- users 组：表示普通用户组
- nobody 组：表示无特权用户组

#### 创建和管理用户组

```bash 

# 创建组
sudo groupadd groupname

# 查看指定用户组的信息
getent group groupname

# 添加用户到组
sudo usermod -aG groupname username
sudo gpasswd -a username groupname

# 从组中移除用户
sudo gpasswd -d username groupname

# 设置组管理员
sudo gpasswd -A username groupname       # 设置组管理员

# 删除组
sudo groupdel groupname

```

### 文件权限

#### 查看文件权限

通过 `ls -l ` 命令查看文件，命令输出结果格式为：-rwxr-xr-- 1 user group size date filename

```text 

类型和权限   链接数 所有者 所属组  大小  修改时间        文件名
drwxrwxrwx    2    alice  dev    4096  Jan 01 10:00  project
-rw-r--r--    1    bob    sales  1024  Jan 01 09:00  report.txt

project 文件权限说明：
文件类型为 d 表示该文件是一个目录
文件权限为 rwxrwxrwx 表示文件的所有者、所有组以及其他用户，都对该文件有读(r)写(w)执行(x)的权限

文件 report.txt 权限说明：
文件类型为 - 表示该文件是一个普通文件
文件权限为 rw-r--r-- 表示文件所有者对文件只有读写权限没有执行权限，所属组对文件只有读权限，其他用户文件只有读权限

```

#### 文件权限修改

1. 通过八进制模式修改文件权限

```bash

# 常用权限组合
chmod 755 file    # rwxr-xr-x (所有者可读写执行，其他人可读执行)
chmod 644 file    # rw-r--r-- (所有者可读写，其他人只读)
chmod 600 file    # rw------- (仅所有者可读写)
chmod 777 file    # rwxrwxrwx (所有人可读写执行)
chmod 750 file    # rwxr-x--- (所有者全部，组可读执行，其他人无)
chmod 2770 dir    # 设置SGID，用户和组有全部权限

# 递归修改目录及内容
chmod -R 755 directory/

```

2. 通过符号模式修改文件权限

文件用户简写符号：

- u (user/owner): 文件所有者
- g (group): 所属组
- o (others): 其他用户
- a (all): 所有用户

权限类型：

- r (read): 读取权限
- w (write): 写入权限
- x (execute): 执行权限
- -：无权限

```bash 

# 添加权限
chmod u+x script.sh      # 给所有者添加执行权限
chmod g+w file.txt       # 给组添加写权限
chmod a+r document.pdf   # 给所有人添加读权限
chmod +x script.sh       # 给所有用户添加执行权限

# 设置权限
chmod u=rwx,g=rx,o=r file    # 所有者rwx，组rx，其他人r
chmod u=rw,g=r,o= file       # 其他人无权限

# 添加和取消权限
chmod u+x, g-w, o=r file

# 移除权限
chmod u-w file.txt       # 移除所有者的写权限
chmod o-rx directory/    # 移除其他人的读和执行权限

```

#### 修改所有者和所属组 (chown, chgrp)

```bash 

# 修改所有者和所属组
sudo chown username:groupname file
sudo chown username:groupname directory/ -R  # 递归修改

# 只修改所有者
sudo chown username file
sudo chown username directory/ -R

# 只修改所属组
sudo chgrp groupname file
sudo chown :groupname file  # 另一种写法
sudo chgrp groupname directory/ -R

# 复制权限从参考文件
chmod --reference=source_file target_file
chown --reference=source_file target_file

```

#### 文件特殊权限

1. SUID(Set User ID)权限

- 文件执行时，以文件所有者的权限运行
- 显示为 s 代替 x：rwsr-xr-x

```bash 

chmod u+s file      # 设置SUID
chmod 4755 file     # 数字方式

# Linux 系统 passwd 文件拥有 SUID 权限，可通过以下命令查看
ls -l /usr/bin/passwd  # 查看 passwd 命令的 SUID

```

2. SGID(Set Group ID)权限
   
- 对于文件：执行时以文件所属组的权限运行
- 对于目录：在该目录中创建的新文件继承目录的所属组
- 显示为 s 代替 x：rwxr-sr-x

```bash

# 设置目录的SGID
chmod g+s directory/
chmod 2775 directory/
mkdir -m 2775 shared_dir  # 创建时设置

# 查看
ls -ld shared_dir/  # 显示 drwxr-sr-x

```

#### sudo 权限

当命令前缀带上 sudo，即可为这一条命令临时赋予 root 授权，如当前用户，没有另一个用户和用户组的权限，又想去访问另一个用户的数据或者执行命令时，可以通 sudo 命令临时赋予 root 授权或者将当前用户添加到有权限的用户组

注意：不是所有的用户，都有权利使用 sudo 权限，我们需要为普通用户添加 sudo 权限
