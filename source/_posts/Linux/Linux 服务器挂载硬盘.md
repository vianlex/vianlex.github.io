--- 
title: Linux 服务器挂载硬盘
---

## 服务器硬盘
1、使用命令 fdisk -l 查看服务有哪些硬盘
2、使用命令 df -h 查看已挂着的命令

## 在硬盘上创建文件系统
```
命令格式：mkfs.文件系统 硬盘，具体例子如下：

mkfs.ext4 /dev/vdb

```
## 使用 blkid 命令查看硬盘 UUID


## 参考链接
1、https://www.cnblogs.com/hiit/p/12106099.html
