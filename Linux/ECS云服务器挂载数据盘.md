# ECS 云服务器挂载数据盘


## 查看服务是否有未挂载的硬盘

```bash
# 第一种查看方式，该命令的结果以树形显示，如果硬盘的没有 part 类型(TYPE)说明硬盘未分区，挂载点(MOUNTPOINT)是空的，说明硬盘未挂载到系统
lsblk -p
# 第二种查看方式
fdisk -l
```

## 挂载前先分区硬盘

```bash
# 执行以下命令，为硬盘 /dev/sdb 分区
fdisk /dev/sdb
```

输入命令后需要依次输入以下参数
1. 输入 n 表示新增分区
2. 输入 p 表示建立主分区
3. 输入 1 表示只需要建立一个分区
4. First sector 起始扇区，因为只是建立一个分区，直接回车即可
5. last sector 结束扇区，因为只是建立一个分区，直接回车即可
4. 输入 w 表示将分区信息写到硬盘并结束分区。
![Linux分区说明图](../images/Linux分区说明图.png)



## 格式化硬盘

硬盘分区后，还需要将分区格式化为 linux 支持的文件系统格式才行

```bash
# 将 /dev/sdb 硬盘的 /dev/sdb1 分区格式化为 ext4 文件系统格式
mkfs -t ext4 /dev/sdb1
```
## 将分区挂载到指定目录

```bash
# 将 /dev/sdb1 分区挂载到 /data 目录，如果 /data 目录不存在则需要先 mkdir /data
mount /dev/sdb1 /data
```

## 开机自动挂载分区

`mount` 命令挂载的分区只是临时挂载，重启机器后就会失效，故需要将挂载信息写入文件 /etc/fstab 中，机器重启开机时就会自动挂载分区，操作方式是在 /etc/fstab 文件末尾添加一条记录如下，可以执行 ` mount -a `命令检查添加的记录是否有错,该命令表示自动挂载 `/etc/fstab` 的内容。
```bash
# 设备标识                                 挂载点     文件系统    挂载选项         
UUID=6d935c4a-a61f-4c5f-b546-e1df4cae729c  /         ext4       defaults        0    2
```

`/etc/fstab` 文件配置说明
- 第一列：分区标识，可以分区名称或者分区UUID 或者 NFS, 分区 UUID 可以通过命令 `blkid` 查看，如 `blkid  /dev/sdb1`
- 第二列：挂载点，即分区需要挂载到的目录
- 第三列：指定分区的文件系统
- 第四列：挂载选项，默认填写 defaults 即可
- 第五列：表示是否转储dump，未配置则默认为 0
- 第六列：表示开机 fsck 文件系统检查，0：表示不检查，1：表示第一位检查，一般用于根挂载点，2：其他磁盘配置参数

