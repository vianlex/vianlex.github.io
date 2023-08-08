# dnf 软件包管理工具

## 1. dnf 简介
dnf 致力于改善 yum 的瓶颈，即性能、内存占用、依赖解决、速度和许多其他方面。DNF使用RPM、libsolv和hawkey库进行包管理。
 
## 2. 安装 dnf 

```bash
# 先 epel-release 仓库，因为默认的软件库中没有 dnf 
yum install epel-release  -y
# 安装 dnf 
yum install dnf 
```
## 3. dnf 查看软件库

```bash
# 查看可用的软件库
dnf repolist

# 查看所有软件库
dnf repolist all 
```
## 4. dnf 查看软件包

```bash
# 查看所有可用和已经安装的软件包
dnf list 

#  查看已经安装的软件包
dnf list installed 

# 查看所有可用的软件包
dnf list available

```
## 5. dnf 搜索软件包

搜索软件包的命令 `dnf search 软件名` 查找子软件包命令 `dnf provides 软件名`，如：

```bash
# 搜索软件包, 如查找 docker 软件包
dnf search docker-ce 
# 子软件包
dnf 
```
## 6. dnf 查看软件包详情

查看软件包详情使用命令`dnf info 软件包名`，如：

```bash 
# 如查看 docker 软件包详情
dnf info docker-ce

```
## 7. dnf 添加软件仓库

```bash
# 如添加 docker 的阿里云软件源
dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
## 8. dnf 安装软件

常用可选参数 
 - -y 安装时，遇到确认默认全部选择 yes  
 - -q 不显示安装过程
 - -v, --verbose 详细显示安装过程
 - --nobest 不尝试将软件升级到最高的版本，不指定时默认是 --best 会尝试安装最高版本
 - --nodocs 不要安装文档
 - -e, --errorlevel 安装日志显示到 error 级别
 - -d, --debuglevel 安装日志显示到 debug 级别
 - --enablerepo 指定从哪个库中安装软件

```bash
# 如安装 docker 软件
dnf install docker-ce -y --nobest

# 重新安装软件包
dnf reinstall docker-ce 

```
## 9. 升级软件

```bash
# 升级所有软件到最新的版本
dnf update  或者  dnf upgrade 
# 升级所有软件到最新的稳定版本
dnf distro-sync
# 升级指定的软件包，如升级 docker
dnf update update -y docker-ce

```
## 10. 回滚降级软件

```bash
# 如回滚降级 docker
dnf downgrade docker-ce

```
## 11. 卸载、删除软件包

```bash
# 卸载指定的软件包，如卸载 docker
dnf remove -y docker-ce 或者 dnf erase -y docker-ce

# 删除无用的依赖包，有些依赖包安装的时候用到，运行的时候用不到的
dnf autoremove

# 清理缓存中的无用软件包
dnf clean all
```
## 12. 查看 dnf 的历史运行命令

```bash
# 查看运行过的历史命令
dnf history 
```

## 13. dnf module 命令

```
# 查看所有可用的软件包
dnf module list 

# 查看指定可用的软件包版本
dnf module list 软件名

# 安装指定版本的软件包, 注意使用冒号在软件名后面指定软件版本
dnf module install docker-ce:19

# 降级到指定版本使用以下命令
dnf module reset docker-ce
dnf module install docker-ce

```